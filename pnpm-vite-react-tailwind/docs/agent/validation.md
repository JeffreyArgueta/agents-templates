## 5. Validation — Schemas

> Read this when adding or changing validation schemas, field rules, or cross-field constraints.

- One schema per domain in `features/<domain>/schemas/<domain>.schema.js` (or `src/schemas/` if not using `features/`).
- Use the validation library chosen in §0. Example with `zod`:
  ```js
  import { z } from 'zod';

  export const emailSchema = z.string().email('Enter a valid email address');
  export const phoneSchema = z.string().regex(/^\d{4}-\d{4}$/, 'Phone must match 0000-0000');
  export const required   = (msg) => z.string().trim().min(1, msg);

  export const exampleSchema = z.object({
    name:  required('Name is required'),
    email: emailSchema,
    phone: phoneSchema,
    country: required('Select a country'),
  }).strict();
  ```
  If a different library is chosen, keep the same structure: reusable field schemas, a strict object schema per domain, and a factory when dynamic options are needed.

- Cross-field and conditional logic belongs in the schema (`superRefine`, `discriminatedUnion`, or equivalent) — not in the component.
- Normalize value types (string vs number) at the schema boundary — e.g. `z.coerce.number()` — and document the choice in the schema file.
- When catalog-driven options include a dynamic "other" entry, resolve its id once from the catalog and pass it to a schema factory: `createExampleSchema({ otherId })`.
- Validate on blur and clear errors on change after a field has been touched; do not validate only on submit.
- Legacy helpers in `utils/` may remain during migration, but new code should go through `schemas/`.

---
