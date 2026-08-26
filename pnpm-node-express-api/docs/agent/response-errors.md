## 7. API Response & Centralized Error Handling

> Read this when changing API response shape or error handling.

**Success:**
```json
{ "status": true, "message": "Resources retrieved successfully", "data": {} }
```
**List (offset — adds pagination):**
```json
{
  "status": true,
  "message": "Resources retrieved successfully",
  "data": [],
  "pagination": { "total": 100, "limit": 10, "offset": 0, "currentPage": 1, "totalPages": 10, "hasNextPage": true, "hasPrevPage": false }
}
```
**List (cursor — see §8):**
```json
{
  "status": true,
  "message": "Resources retrieved successfully",
  "data": [],
  "pagination": { "limit": 10, "nextCursor": 42, "hasNextPage": true }
}
```

**Error classes** (`src/utils/errors.js`, ESM):
```js
export class AppError extends Error { constructor(message, status, code){ super(message); this.status=status; this.code=code; } }
export class ValidationError extends AppError { constructor(msg){ super(msg, 422, 'VALIDATION_ERROR'); } }
export class NotFoundError extends AppError { constructor(msg='Resource not found'){ super(msg, 404, 'NOT_FOUND'); } }
export class ConflictError extends AppError { constructor(msg='Conflict'){ super(msg, 409, 'CONFLICT'); } }
export class UnauthorizedError extends AppError { constructor(msg='Unauthorized'){ super(msg, 401, 'UNAUTHORIZED'); } }
export class ForbiddenError extends AppError { constructor(msg='Forbidden'){ super(msg, 403, 'FORBIDDEN'); } }
```

**Centralized middleware** (`src/middlewares/error.middleware.js` — must be **last** in `app.js`, after routes and 404 handler):
```js
import { logger } from '../utils/logger.js';
export const errorHandler = (err, req, res, _next) => {
  const status = err.status || 500;
  const message = status === 500 ? 'Internal server error' : err.message;
  if (status === 500) logger.error(err.stack);
  res.status(status).json({ status:false, message, ...(err.code && { code: err.code }) });
};
export const notFound = (req, res) => res.status(404).json({ status:false, message:'Route not found', path: req.path });
```
- Controllers/services **throw** typed errors; Express 5 forwards async throws to `errorHandler` — **no per-controller `try/catch`**.
- Joi validation failure at controller → `throw new ValidationError(error.details[0].message)` instead of manual `res.status(422)`.

---
