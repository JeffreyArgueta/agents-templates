## 9. Auth & Authorization — Cookie-Based

> Read this when touching auth, cookies, JWT, or authorization guards.

**Stack:** `cookie-parser` + `jsonwebtoken` (or `jose`). No localStorage. All auth state lives in **httpOnly cookies**.

### Environment
Add to `requiredEnvVars` in `src/config/environment.js`:
`JWT_SECRET`, `JWT_REFRESH_SECRET`, `COOKIE_SECRET` (if signing), `COOKIE_DOMAIN` (optional).

### Setting Cookies (login / refresh)
```js
import jwt from 'jsonwebtoken';
const accessToken  = jwt.sign({ id:user.idUser, role:user.role }, JWT_SECRET,  { expiresIn:'15m' });
const refreshToken = jwt.sign({ id:user.idUser }, JWT_REFRESH_SECRET, { expiresIn:'7d' });

res.cookie('accessToken', accessToken, {
  httpOnly: true,
  secure: process.env.NODE_ENV === 'production', // true in prod (HTTPS)
  sameSite: 'strict', // or 'lax' if you need top-level navigation; never 'none' without secure:true
  maxAge: 15*60*1000,
  path: '/',
  // domain: COOKIE_DOMAIN, // only if cross-subdomain
});
res.cookie('refreshToken', refreshToken, {
  httpOnly: true,
  secure: process.env.NODE_ENV === 'production',
  sameSite: 'strict',
  maxAge: 7*24*60*60*1000,
  path: '/api/v1/auth/refresh', // narrow scope
});
```

### Middlewares

**`src/middlewares/auth.middleware.js`:**
```js
import jwt from 'jsonwebtoken';
import { UnauthorizedError, ForbiddenError } from '../utils/errors.js';
export const authenticate = (req, _res, next) => {
  const token = req.cookies?.accessToken;
  if (!token) throw new UnauthorizedError('Authentication required');
  try { req.user = jwt.verify(token, process.env.JWT_SECRET); next(); }
  catch { throw new UnauthorizedError('Invalid or expired token'); }
};
export const authorize = (...roles) => (req, _res, next) => {
  if (!roles.includes(req.user.role)) throw new ForbiddenError('Insufficient permissions');
  next();
};
```
- Use `cookie-parser`: `app.use(cookieParser(COOKIE_SECRET))` **before** `auth`.
- Never read `Authorization: Bearer` header when using cookies — document that the frontend uses `fetch(..., { credentials:'include' })`.

### CSRF & CORS Considerations
- `sameSite: 'strict'` mitigates CSRF for same-site cookies. If you need cross-site (frontend on different domain), use `sameSite: 'none'` + `secure:true` **and** add `csurf`/`csrf-csrf` double-submit token: set `csrfToken` non-httpOnly cookie, require `x-csrf-token` header on mutating requests.
- **CORS** must have `credentials:true`, `origin` whitelist (`ALLOWED_ORIGINS`), `allowedHeaders` must include `X-CSRF-Token` if used, and frontend must send `credentials:'include'`.
- Rate-limit auth routes with `sensitiveLimiter` (`1min / 10 req`) and add brute-force protection.

### Routes Example
```js
// routes/v1/auth.routes.js
import { Router } from 'express';
import { authenticate } from '../../middlewares/auth.middleware.js';
import * as authController from '../../controllers/auth/index.js';
const router = Router();
router.post('/login', authController.login);                 // sets cookies
router.post('/refresh', authController.refresh);             // rotates refreshToken
router.post('/logout', authenticate, authController.logout); // clears cookies
export default router;

// routes/v1/user.routes.js
router.get('/', authenticate, authorize('admin'), userController.getUsers);
```

### Rules
- Never store tokens in JS-accessible storage; never log cookies.
- `logout` clears cookies: `res.clearCookie('accessToken', { path:'/' }); res.clearCookie('refreshToken', { path:'/api/v1/auth/refresh' });`.
- Rotate refresh tokens on use; keep refresh token family in DB/Redis if you need revocation.

---
