## 4. State, Data Fetching & Forms

> Read this when implementing forms, wizards, server-state fetching, or cascading selects.

### State Placement

- **Server state** (data fetched from an API) — use the server-state library chosen in §0. Do not duplicate it in local `useState`.
  ```js
  // Example with tanstack query — adapt to the chosen library
  // services/api.js
  export const listQuery = () => ({
    queryKey: ['items'],
    queryFn: getItems,
    staleTime: 1000 * 60 * 5,
  });
  // component
  const { data, isLoading, error } = useQuery(listQuery());
  ```
  If no server-state library is chosen, encapsulate caching, deduplication and abort handling inside `services/http.js` and custom hooks.

- **Multi-step / wizard state** — use `useReducer` or the chosen form library with `useForm` + `FormProvider`. Keep the reducer pure; derive visible steps with `useMemo`, not inline `filter` on every render.
  ```js
  const [state, dispatch] = useReducer(wizardReducer, initialState);
  const visibleSteps = useMemo(
    () => steps.filter((s) => !s.condition || s.condition === state.values.status),
    [state.values.status]
  );
  ```

- **Dependent / cascading selects** — model dependencies via query keys that include the parent value. Use `enabled` to prevent fetching until the parent is selected. This replaces manual `useEffect` chains.
  ```js
  const countriesQ = useQuery({ queryKey: ['countries'], queryFn: getCountries, staleTime: Infinity });
  const regionsQ   = useQuery({ queryKey: ['regions', country], queryFn: () => getRegions(country), enabled: !!country });
  const citiesQ    = useQuery({ queryKey: ['cities', country, region], queryFn: () => getCities(country, region), enabled: !!country && !!region });
  ```

- **Global client state** (auth session, theme, cart) — use the client-state library chosen in §0. Do not lift state through many props when a store or router loader is sufficient.

### Forms

- Use the form library chosen in §0. Recommended pattern when `react-hook-form` is selected:
  ```jsx
  import { useForm, FormProvider } from 'react-hook-form';
  import { zodResolver } from '@hookform/resolvers/zod';
  import { formSchema } from '@/features/example/schemas/example.schema.js';

  export function ExampleForm({ defaultValues }) {
    const methods = useForm({
      resolver: zodResolver(formSchema),
      defaultValues,
      mode: 'onBlur',
    });
    const { handleSubmit } = methods;
    return (
      <FormProvider {...methods}>
        <form onSubmit={handleSubmit(onSubmit)} noValidate>
          {/* fields */}
        </form>
      </FormProvider>
    );
  }
  ```
  If no form library is chosen, keep the same principles: validate on blur, clear errors on change after touch, and do not query `document` for error scrolling.

Rules:
- Each step or section validates its own slice; keep hidden step values with `shouldUnregister: false` (or equivalent).
- Conditional fields use schema-level refinement (`superRefine`, `discriminatedUnion`, or equivalent) — not ad-hoc `if` checks in the component.
- On submit error, scroll to the first invalid field via refs or a dedicated hook (e.g. `use-scroll-to-first-error`) that maps `errors` to `ref.current` — do not use `document.getElementById` or `document.querySelector` inside components.
- Format-on-type (e.g. phone, id) is handled via a controlled `Controller` or a `format` utility piped through `onChange`, with a `maxLength` guard.

### Performance Notes for State

- Avoid creating new `options` arrays inside render — memoize transformed catalogs.
- Apply `React.memo` only to expensive primitives (e.g. a listbox with hundreds of items) and only after profiling.
- Keep feature containers focused — extract `useWizard()`, `useSubmit()`, `useDependentData()` when a file grows beyond ~150 lines.

---
