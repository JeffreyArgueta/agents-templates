## 13. Testing — Unit, Integration & Visual

> Read this when writing unit, integration, or e2e tests, factories, or mocks.

### Structure

```
src/
  components/ui/button.test.jsx
  features/example/hooks/use-example.test.jsx
  services/api.test.js
  test/
    setup.js
    factories/example.factory.js
    mocks/handlers.js
    mocks/server.js
e2e/
  example.spec.js
```

### Stack

- Choose a test stack per project and record it in §0. Recommended default: `vitest` + `jsdom` + `@testing-library/react` + `@testing-library/user-event` + `msw` — but any equivalent is acceptable if used consistently.
- **Setup** (`src/test/setup.js` — adapt to the chosen runner):
  ```js
  import '@testing-library/jest-dom/vitest';
  import { afterAll, afterEach, beforeAll } from 'vitest';
  import { server } from './mocks/server.js';
  beforeAll(() => server.listen({ onUnhandledRequest: 'warn' }));
  afterEach(() => server.resetHandlers());
  afterAll(() => server.close());
  ```
- **Factories** (avoid brittle fixtures):
  ```js
  // src/test/factories/example.factory.js
  import { faker } from '@faker-js/faker';
  export const buildExample = (o = {}) => ({
    name: faker.person.fullName(),
    email: faker.internet.email().toLowerCase(),
    ...o,
  });
  ```

### Conventions

- **Unit:** test primitives and hooks in isolation — assert `aria-invalid`, `aria-describedby`, keyboard behavior for listboxes, and variant classes via `cn`.
- **Integration:** render a route or feature with mocked network — assert loading, success, error, conditional fields, and progress flows.
- **Accessibility:** automated checks via `vitest-axe` or `jest-axe`; enforce a coverage threshold (e.g. `80%` lines — adjust per project) and fail CI if below.
- **E2E:** use Playwright or equivalent for critical paths — happy path, validation errors, and `429`/`500` states.
- **Scripts:** `pnpm test`, `pnpm test:watch`, `pnpm test:coverage`, `pnpm test:e2e` (names are conventional — adapt to the chosen runner).

### Example (unit)

```jsx
import { render, screen } from '@testing-library/react';
import { TextField } from '@/components/ui/text-field.jsx';

test('shows error and aria-invalid', () => {
  render(<TextField id="email" label="Email" value="bad" error="Invalid email" onChange={() => {}} />);
  expect(screen.getByLabelText(/email/i)).toHaveAttribute('aria-invalid', 'true');
  expect(screen.getByText(/invalid email/i)).toBeInTheDocument();
});
```

---
