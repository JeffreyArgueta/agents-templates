## 13. Testing — Unit, Integration & Factories

> Read this when writing unit, integration, or factory tests.

### Structure
```
tests/
  unit/
    repositories/user.repository.test.js   # mock Sequelize models
    services/user.service.test.js          # mock repositories
  integration/
    routes/user.routes.test.js             # supertest + real test DB
  factories/
    user.factory.js                        # fake data builders (see below)
  setup.js                                 # beforeAll: connect test DB, afterAll: close, beforeEach: truncate
```

### Conventions
- **Unit:** mock repositories (`vi.mock`/`jest.mock`), assert service throws `NotFoundError`/`ConflictError` on edge cases, verify pagination normalization (offset and cursor).
- **Integration:** `supertest(app)` with `credentials` cookies where needed; use separate test DB (`DB_NAME=test_*`); run migrations+seed before suite; clean between tests.
- **Coverage:** `pnpm test -- --coverage` with threshold `80%` lines/branches (fail CI if below).
- **Scripts:** `pnpm test`, `pnpm run test:unit`, `pnpm run test:integration`, `pnpm run test:watch`.

### Factories (avoid brittle fixtures)

Use `@faker-js/faker` + factory helpers instead of hard-coded objects. One factory per model, reusable in unit and integration tests:

```js
// tests/factories/user.factory.js
import { faker } from '@faker-js/faker';
export const buildUser = (overrides={}) => ({
  name: faker.person.fullName(),
  email: faker.internet.email().toLowerCase(),
  role: 'user',
  ...overrides,
});
export const createUser = async (overrides={}) => {
  const { User } = await import('../../src/models/index.js');
  return User.create(buildUser(overrides));
};
```
- Factories build valid payloads via the same Joi schemas when possible.
- In unit tests: `buildUser()` for in-memory objects. In integration: `await createUser()` for DB rows.

### Example (unit, ESM + vitest)
```js
import { describe, it, expect, vi } from 'vitest';
import * as repo from '../../../src/repositories/user/index.js';
import { getUsers } from '../../../src/services/user/getUsers.service.js';
import { buildUser } from '../../factories/user.factory.js';
vi.mock('../../../src/repositories/user/index.js');
```

---
