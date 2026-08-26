## 4. Data Layer — Models, Database & Migrations

> Read this when changing models, migrations, database config, or ORM mappings.

- **Source of truth:** versioned migrations in `database/migrations/` + `src/models/` for ORM mapping (camelCase ↔ snake_case). The initial migration may be generated from `database/01-create-database.sql` but after v1 never edit that file — create a new migration.
- **Tooling:** `sequelize-cli` (or `umzug`). Commands `pnpm run db:migrate` / `db:migrate:undo` / `db:seed`. Migrations are ESM (`export async function up(queryInterface, Sequelize)`).
- **ORM config (Sequelize 6 + MySQL):** `underscored: true`, `freezeTableName: true`, charset `utf8mb4`, collation `utf8mb4_bin`, pool `{ max: 10, min: 2 }`, timezone explicitly, `mysql2` driver.
- **Associations:** defined in `models/index.js` (1:N, N:M via bridge tables). Always add DB indexes for FKs and composite PKs.
- **IDs:** `id<Resource>` (e.g. `idUser`), auto-increment for main entities. Bridge tables use composite PK `(idA, idB)` — see §6.
- **Timestamps:** disable or enable consistently (`timestamps: false` or `true`). If disabled, manage audit dates manually (e.g. `lastUpdatedAt`).
- **Read-only columns** (e.g. `externalId`, `lastUpdatedAt` populated by system flows): strip on input via `Joi.any().strip()` so clients cannot overwrite them.
- **Performance:** add indexes for `where`/`orderBy` fields, cap pagination `limit` (e.g. `max 500`), prefer `findAndCountAll` or separate `count` + `findAll` via repository, avoid N+1 by eager loading only when needed. For large tables prefer **cursor pagination** — see §8.
- **Security — raw queries:** never interpolate values into `sequelize.query()` strings. Always use `replacements` / `bind`:
  ```js
  // BAD
  sequelize.query(`SELECT * FROM users WHERE email='${email}'`)
  // GOOD
  sequelize.query('SELECT * FROM users WHERE email = :email', { replacements: { email }, type: QueryTypes.SELECT })
  ```

---
