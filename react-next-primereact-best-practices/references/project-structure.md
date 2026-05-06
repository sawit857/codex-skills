# Next.js + PrimeReact Project Structure

Use this file first when the task is about folder layout, module placement, refactoring structure, or deciding where new code should live.

## Recommended default: Enterprise Structure

This is the team-preferred structure for serious applications.

Use it when:

- the project will grow beyond a few pages
- multiple teams or domains will work in the same repo
- the codebase needs clear domain boundaries
- feature logic, services, and shared modules should be grouped under domain/resource roots

```text
project root
├─ public/
└─ src/
   ├─ app/
   │  ├─ layout.tsx
   │  ├─ page.tsx
   │  ├─ not-found.tsx
   │  ├─ api/
   │  └─ <route segments>/
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

Use this for smaller or medium projects where a layer-based shared structure is enough.

```text
project root
├─ public/
└─ src/
   ├─ app/
   │  ├─ layout.tsx
   │  ├─ page.tsx
   │  └─ api/
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
- domain separation would add more structure than value
- the team wants one shared layer-based module tree

Do not keep a project in this structure once domain sprawl becomes obvious. Move to `Enterprise Structure` instead of allowing unrelated features to pile into shared folders.

## How `src/app` and `src/resources/<group>` work together

Use `src/app` for routing, segment composition, and Next-specific entrypoints.

Use `src/resources/<group>` for reusable or domain logic that should survive beyond a single route file.

Typical split:

- `src/app/.../page.tsx`
  - compose the page
  - read route params and search params
  - call server-side loaders or import the top-level page component
- `src/resources/<group>/components`
  - shared UI for that group
  - domain-aware forms, tables, cards, message blocks
- `src/resources/<group>/services`
  - domain operations and server action adapters
- `src/resources/<group>/lib`
  - lower-level helpers, API clients, DTO mappers, wrappers
- `src/resources/<group>/models`
  - interfaces, types, domain models

Page files should orchestrate. They should not become the permanent home for duplicated API calls, repeated error extraction, or large form infrastructure.

## Placement rules

### `src/app`

Put these here:

- route entrypoints
- segment layouts
- route groups
- loading and error boundaries
- route handlers
- top-level composition for page trees

Do not place reusable business logic here unless it is truly route-local and unlikely to be reused.

### `src/resources/<group>/services`

Put these here:

- domain operations
- server actions tied to that group
- higher-level API workflows
- request/response orchestration for that domain

Examples:

- employee CRUD operations
- submit flows for chapter-specific exercises
- domain-specific validation or save/update/delete entrypoints

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

Use top-level app providers in `src/app` composition when they affect the whole application.

### `src/resources/<group>/hooks`

Put these here:

- reusable hooks for the group
- hooks that encapsulate repeated view logic

Do not hide domain actions in hooks if plain services or server actions make the intent clearer.

### `src/resources/<group>/layouts`

Put these here:

- shared layout components for that group
- layout wrappers that are not route entrypoint layouts

### `src/resources/<group>/assets`

Put these here:

- static images
- local CSS owned by the group
- group-scoped visual assets

### `src/resources/<group>/tests`

Put these here:

- tests for group-specific models, services, helpers, and components

## Next App Router file roles

Use these file names intentionally:

| File | Purpose |
| --- | --- |
| `page.tsx` | UI entrypoint for a route |
| `layout.tsx` | Shared layout around route children |
| `template.tsx` | Layout-like wrapper that re-renders on navigation |
| `loading.tsx` | Segment-level loading UI |
| `error.tsx` | Segment-level error UI |
| `not-found.tsx` | Not found UI |
| `default.tsx` | Default slot for parallel routes |
| `route.ts` | Request handler for App Router endpoints |

## Migration guidance

If a project starts with `Global Standard Structure` and later grows:

1. keep `src/app` in place
2. create `src/resources/<group>`
3. move related `components`, `models`, `services`, and `lib` modules into the group
4. leave only truly global shared modules at the top level if needed
5. avoid partial migration where half of one feature stays in global folders and half moves into resources

## Review questions

Ask these during reviews:

- Is the route layer too heavy?
- Is reusable or domain logic trapped in page files?
- Is a growing project still pretending to be small?
- Are `services` and `lib` split by responsibility rather than arbitrarily?
- Is the chosen structure still appropriate for the current size and team shape?
