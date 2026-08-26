## 6. Repositories, Services & Controllers — Conventions

> Read this when touching repositories, services, or controllers — or deciding where logic belongs.

### Repositories
- Pure data access: wrap Sequelize calls (`findAll`, `findOne`, `create`, `bulkCreate`, `update`, `destroy`, `count`). No business rules, no Joi, no `error.status` mapping beyond re-throwing.
- Accept plain params (`where`, `limit`, `offset`, `cursor`, `transaction`) and return raw models/rows. ESM:
  ```js
  import { User } from '../../models/index.js';
  export const getUsers = ({ limit, offset, transaction }) => User.findAll({ limit, offset, transaction, order:[['idUser','DESC']] });
  ```
- One repository function per operation; kept thin so services stay testable (mock repositories in unit tests).

### Services
- Receive already-normalized params (`limit`, `offset`, `cursor`, validated `value`).
- Normalize pagination: `Math.max(1, parseInt(limit,10) || 10)` / `Math.max(0, parseInt(offset,10) || 0)` for offset; for cursor validate `cursor` is positive integer or `null`.
- Delegate DB calls to repositories; use `buildPagination` (offset) or `buildCursorPagination` (cursor) — see §8.
- Throw typed errors (`NotFoundError`, `ConflictError`) — see §7 — so the centralized handler can map them.
- Log with `logger.info` / `logger.error`.
- Validate before writing; wrap multi-step writes in a **transaction** (pass `transaction` to repositories); if auditing, create snapshot within the same transaction.

### Controllers — Single Error Policy (no duplication)

- **Do NOT write `try/catch` + `res.status(error.status || 500)` in every controller.** The centralized `errorHandler` (§7) is the single place that maps errors to HTTP.
- **Pattern — throw, don't handle:**
  ```js
  import * as userService from '../../services/user/index.js';
  import * as schema from '../../schemas/user.schema.js';
  import { ValidationError } from '../../utils/errors.js';
  // Express 5 auto-forwards async throws to errorHandler — no try/catch needed.
  export const getUsers = async (req, res) => {
    const { error, value } = schema.searchSchema.validate(req.query);
    if (error) throw new ValidationError(error.details[0].message);
    const result = await userService.getUsers(value);
    res.status(200).json({ status: true, message: 'Users retrieved successfully', data: result.data, pagination: result.pagination });
  };
  // Only use try/catch when you need to TRANSLATE a low-level error:
  // try { await repo.create(data); } catch (e) { if (e.name==='SequelizeUniqueConstraintError') throw new ConflictError('Already exists'); throw e; }
  ```
- Never import models or repositories or access DB directly.

### Composite PK (Bridge / Junction Tables)

For N:M bridge tables (e.g. `UserRole` with PK `(idUser, idRole)`):

- **Schemas** require both ids in `getByIdSchema`, `deleteSchema`, `createSchema`, `updateSchema` (update uses `.min(3)`).
- **Repositories/Services** `getById` / `delete` use `Model.findOne({ where: { idUser, idRole } })` (Sequelize `findByPk` does not support composite keys). The repository does the `findOne`; service interprets `null` → `throw new NotFoundError()`.
- **Services** `update` signature is `(idUser, idRole, updateData)`.
- **Controllers** validate `{ idUser, idRole }` and pass `value.idUser, value.idRole` to the service.

**Exception — auto-increment bridge:** if a bridge needs multiple rows per pair (e.g. multiple titles per level), use a surrogate PK like `idRecord` (auto-increment) with `UNIQUE(idUser, idType, name)`. Then treat it as a single-PK model (validate `idRecord` only).

### Extensibility Patterns

- **Search endpoint:** `POST /<resources>/search` with optional AND filters, paginated. Validate with `searchSchema`.
- **Aggregated resource:** `GET /<resources>/:id/complete` and `PUT /<resources>/:id/complete` returning/updating grouped sections (single canonical shape defined in `utils/<resource>Complete.js` via `toCompleteResource`).
- **Alternate identifier:** `PUT /<resources>/by-<field>/:<field>/complete` (e.g. by email, by national ID) — identity comes from the route param, strip id from body schema via `completeSchema.fork(['id'], s => s.optional())`.
- **Dual mode (single or bulk) for bridge tables:** `POST /<bridge>` accepts object or array; schema uses `bulkCreateSchema`; controller branches on `Array.isArray`; service uses `bulkCreate` in one transaction and creates **one audit snapshot per distinct parent id**; duplicates `(idA, idB)` → `throw new ConflictError()` or dedupe.

---
