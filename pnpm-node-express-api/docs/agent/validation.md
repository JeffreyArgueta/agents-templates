## 5. Validation — Schemas

> Read this when adding or changing Joi/Zod schemas or validation rules.

- One file per model in `src/schemas/<model>.schema.js` (ESM: `import { Joi } from 'sequelize-joi'`).
- Export: `getByIdSchema`, `deleteSchema`, `createSchema`, `updateSchema`; optionally `searchSchema`, `bulkCreateSchema`, `completeSchema`.
- Rules:
  - All schemas use `.strict()` and English custom messages (e.g. `'Id must be a positive integer'`).
  - `updateSchema` requires at least one field beyond the id: `.min(2)` for single-PK models, `.min(3)` for composite-PK models, message `'You must provide at least one field to update (besides the ID)'`.
  - Define reusable field schemas (`const email = Joi.string().email()...`) with explicit messages.
  - For composite PKs, `getByIdSchema`/`deleteSchema` validate both keys (`idUser` + `idRole`).

---
