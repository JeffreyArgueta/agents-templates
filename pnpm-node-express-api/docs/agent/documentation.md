## 16. Documentation — Swagger (OpenAPI 3.0)

> Read this when updating Swagger/OpenAPI docs.

- Served at `/api/v1/docs` (and `/api/v2/docs` when v2 exists) via `swagger-ui-express` (`app.use('/api/v1/docs', swaggerUi.serve, swaggerUi.setup(swaggerSpecV1))`).
- Spec assembled in `src/config/swagger.js` from domain modules:
  - `swagger/helpers.js` (shared responses/parameters/helpers like `crud()`),
  - `swagger/paths.<domain>.js`,
  - `swagger/schemas.<domain>.js`.
- Conventions: describe success as `{ status: true, message, data }` (+ `pagination` for lists), errors as `{ status: false, message, code }`; fields in **camelCase**; note `credentials:'include'` + `ALLOWED_ORIGINS` requirement for cookie auth. Document `cookieAuth` security scheme if using `sameSite:none`.

---
