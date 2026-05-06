# React + Vite + PrimeReact Project Structure

Use this file first when the task is about folder layout, module placement, refactoring structure, or deciding where new code should live.

This structure is intentionally aligned with the `react-next-primereact-best-practices` skill, but adapted for a lightweight Vite SPA. The main difference is that Vite does not use App Router files such as `page.tsx` or `layout.tsx`. Instead, `src/app` owns app composition, routing, providers, and layouts.

## Recommended default: Resource-group structure

This is the preferred structure for serious applications, even on Vite.

Use it when:

- the project will grow beyond a few screens
- multiple domains or teams will work in the same repo
- the codebase needs clear boundaries between routing, UI composition, and domain logic
- feature logic, services, and shared modules should stay grouped under domain/resource roots

```text
project root
├─ public/
└─ src/
   ├─ app/
   │  ├─ AppShell.tsx
   │  ├─ router.tsx
   │  ├─ providers.tsx
   │  ├─ layouts/
   │  └─ routes/
   └─ resources/
      ├─ feature/
      │  ├─ assets/
      │  ├─ contexts/
      │  ├─ components/
      │  ├─ hooks/
      │  ├─ layouts/
      │  ├─ lib/
      │  ├─ models/
      │  ├─ services/
      │  └─ tests/
      └─ standard/
         ├─ assets/
         ├─ contexts/
         ├─ components/
         ├─ hooks/
         ├─ layouts/
         ├─ lib/
         ├─ models/
         ├─ services/
         └─ tests/
```

Treat `standard` as only one concrete resource group. Other projects may create `customer`, `sales`, `admin`, `hr`, `inventory`, or other groups with the same internal layout.

## Secondary option: Global Standard Structure

Use this only for smaller projects where domain separation would add more ceremony than value.

```text
project root
├─ public/
└─ src/
   ├─ app/
   │  ├─ AppShell.tsx
   │  ├─ router.tsx
   │  └─ providers.tsx
   ├─ components/
   ├─ hooks/
   ├─ lib/
   ├─ services/
   ├─ models/
   ├─ contexts/
   ├─ layouts/
   ├─ styles/
   └─ tests/
```

Use this when:

- the app is still small
- one shared layer-based module tree is enough
- domain separation would slow the team down more than it helps

Once domain sprawl becomes obvious, move to the resource-group structure instead of letting unrelated features pile into global folders.

## How `src/app` and `src/resources/<group>` work together

Use `src/app` for application composition, router setup, provider wiring, and top-level layouts.

Use `src/resources/<group>` for reusable or domain logic that should survive beyond a single screen file.

Typical split:

- `src/app/router.tsx`
  - define client-side routes
  - compose route wrappers and app-level layouts
- `src/app/AppShell.tsx`
  - own the top-level application shell
  - mount providers, layout frame, and router outlet
- `src/resources/<group>/components`
  - shared UI for that group
  - domain-aware forms, tables, cards, and message blocks
- `src/resources/<group>/services`
  - domain operations and API workflows
- `src/resources/<group>/lib`
  - lower-level helpers, API clients, DTO mappers, wrappers
- `src/resources/<group>/models`
  - interfaces, types, and domain models

Screen files should orchestrate. They should not become the permanent home for duplicated API calls, repeated error extraction, or large form infrastructure.

## Placement rules

### `src/app`

Put these here:

- application shell
- route definitions
- app-level providers
- app-wide layouts
- route wrappers such as auth guards
- router composition

Do not place reusable business logic here unless it is truly app-wide and not domain-specific.

### `src/resources/<group>/services`

Put these here:

- domain operations
- API workflows
- save/update/delete entrypoints
- orchestration that belongs to one domain

Examples:

- employee CRUD operations
- customer search and save flows
- document or workflow submission flows

### `src/resources/<group>/lib`

Put these here:

- API clients
- wrapper functions
- DTO transforms
- utility helpers used by multiple services or components
- environment-aware low-level network configuration

Rule of thumb:

- if the module represents a domain action, use `services`
- if the module is lower-level shared plumbing, use `lib`

### `src/resources/<group>/models`

Put these here:

- TypeScript interfaces
- domain models
- DTO types
- helper types and enums

Keep transport shapes and UI shapes explicit when they differ.

### `src/resources/<group>/components`

Put these here:

- reusable UI for the group
- form fragments
- data views
- query panels
- shared field groups

Do not create thin wrappers that only rename a PrimeReact component with no added logic.

### `src/resources/<group>/contexts`

Put these here:

- provider logic scoped to the group
- context state that is not purely global

Use top-level app providers in `src/app` when they affect the whole application.

### `src/resources/<group>/hooks`

Put these here:

- reusable hooks for the group
- hooks that encapsulate repeated view logic

Do not hide domain actions in hooks if plain services make the intent clearer.

### `src/resources/<group>/layouts`

Put these here:

- shared layout components for that group
- reusable wrappers that are not the top-level app shell

### `src/resources/<group>/assets`

Put these here:

- static images
- local CSS owned by the group
- group-scoped visual assets

### `src/resources/<group>/tests`

Put these here:

- tests for group-specific models, services, helpers, and components

## Lightweight Vite guidance

This structure should stay lightweight.

- Do not copy Next-specific files such as `page.tsx`, `layout.tsx`, or `route.ts` into Vite projects.
- Do not create route wrappers or providers unless they remove real duplication.
- Keep the number of global folders small when the project is still compact.
- Add resource groups as the app grows, not preemptively for imaginary domains.

## Migration guidance

If a project starts with `Global Standard Structure` and later grows:

1. keep `src/app` in place
2. create `src/resources/<group>`
3. move related `components`, `models`, `services`, and `lib` modules into the group
4. leave only truly global shared modules at the top level if needed
5. avoid partial migration where half of one feature stays in global folders and half moves into resources

## Review questions

Ask these during reviews:

- Is app composition isolated under `src/app`?
- Is reusable or domain logic trapped in screen files?
- Is a growing project still pretending to be small?
- Are `services` and `lib` split by responsibility rather than arbitrarily?
- Is the chosen structure still appropriate for the current size and team shape?
