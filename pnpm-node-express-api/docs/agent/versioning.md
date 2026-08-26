## 15. API Versioning Strategy

> Read this when planning API versioning (v1/v2) or deprecations.

### Structure
```
src/routes/
  v1/
    user.routes.js
    auth.routes.js
    index.js        # mounts all v1 routes
  v2/               # when breaking changes are needed
  index.js          # { v1Router } or mounts /v1 and /v2
```
- `src/app.js` mounts `app.use('/api/v1', v1Router)`. Never mount versionless `/api`.
- **Swagger** is versioned: `src/config/swagger.js` generates `swaggerSpecV1`; served at `/api/v1/docs`. `v2` gets `/api/v2/docs`.

### Rules
- **Additive changes** (new field optional, new endpoint) → stay in `v1`.
- **Breaking changes** (rename/remove field, change auth, change pagination shape e.g. offset→cursor) → create `v2` folder, copy+modify affected routes/controllers/services, keep `v1` for deprecation period.
- **Deprecation:** add `Deprecation: true` + `Sunset: <date>` headers in `v1` response when `v2` is stable; log usage; remove after 2-3 releases.
- Models/schemas are shared until divergence forces `schemas/v2/` or `models/v2/` split — avoid premature duplication.
- Version bump is documented in `package.json` + `swagger.info.version`.

---
