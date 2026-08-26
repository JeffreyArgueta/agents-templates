## 10. Security & Middlewares

> Read this when changing middlewares, CORS, helmet, rate limiting, or raw SQL handling.

Order in `src/app.js` (ESM) — **order matters**:

```js
import express from 'express';
import cookieParser from 'cookie-parser';
import compression from 'compression';
import helmet from 'helmet';

app.use(express.json({ limit: '10kb' }));           // 1) body limit — prevents large payload DoS
app.use(express.urlencoded({ extended:true, limit:'10kb' }));
app.use(cookieParser(process.env.COOKIE_SECRET));    // 2)
app.use(compression());                              // 3) gzip/deflate before cors
app.use(corsMiddleware);                             // 4)
app.use(helmetMiddleware);                           // 5)
app.use(xssSanitize());                              // 6)
app.use(morgan('combined', { stream: { write: m=>logger.info(m.trim()) } }));
app.use(limiter);                                    // 7) before routes
app.use('/api/v1', v1Router);
app.use(notFound);
app.use(errorHandler);                               // last
```

- **body limit:** `10kb` for JSON (raise only for upload routes with explicit `multer`/`busboy` limits).
- **helmet:** strict CSP (`defaultSrc 'self'`, `scriptSrc 'self'`, `styleSrc 'self' 'unsafe-inline'`, `imgSrc 'self' data: https:`, `frameAncestors 'none'`), `referrerPolicy`, `hsts` (1 year, includeSubDomains, preload).
- **cors:** whitelist from `ALLOWED_ORIGINS` (comma-separated). Allow `!origin` (same-origin / curl), otherwise check `ALLOWED_ORIGINS.split(',').includes(origin)`. `credentials:true`, `exposedHeaders:['X-Total-Count']`, `allowedHeaders:['Content-Type','X-CSRF-Token','X-Requested-With']`, `maxAge:86400`. Required for cookie auth.
- **compression:** `compression` middleware (gzip) — after body parser, before routes. Exclude already-compressed assets.
- **rate-limiter:** general `15min / 100 req` + `sensitiveLimiter` `1min / 10 req` for auth / sensitive routes. Use `express-rate-limit` with `standardHeaders:true`.
- **xss-sanitize:** sanitize body/query/params.
- **validation:** Joi `.strict()` on all external input; strip unknown fields. Never trust `req.query`/`req.body` without schema.
- **raw SQL:** never interpolate — always `replacements`/`bind` (see §4).
- **cookies:** always `httpOnly`, `secure` in production, `sameSite` as §9; use `cookie-parser` with `COOKIE_SECRET` if signing.

---
