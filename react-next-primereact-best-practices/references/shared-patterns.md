# Shared patterns for the team structure

Use this file when the same UI, error, loading, or API pattern appears in multiple pages or groups.

## Placement guidance

Shared patterns can live in one of two places:

- in a common shared area when they are app-wide
- in `src/resources/<group>/...` when they are domain-specific

Default rule:

- app-wide provider or message infrastructure: keep close to top-level app composition
- domain-specific shared UI: keep inside `src/resources/<group>/components`
- shared API clients and helpers: keep inside `src/resources/<group>/lib` or a common shared lib module

## 1. Shared message or toast strategy

Problem:

- each page creates its own `useRef<Messages>` or `useRef<Toast>`
- success and error handling are repeated

Preferred pattern:

- one shared provider or helper for application-wide feedback
- page-level code calls the shared API instead of re-creating message refs

For Next.js:

- wire the provider near the top-level app layout or a route-group layout
- keep only local, truly route-specific message state inside individual pages

## 2. API error extraction

Problem:

- pages repeatedly parse Axios errors inline
- fallback messages differ arbitrarily across pages

Preferred pattern:

- one shared error extractor
- one reusable `ApiErrorMessage` component where visual output repeats

Put this in:

- a common shared module when many groups use the same error shape
- or `src/resources/<group>/lib` and `components` when the shape is domain-specific

## 3. `FormField`

Problem:

- label + input + error layout is repeated across forms

Preferred pattern:

- a shared `FormField` wrapper that handles label, required marker, child input, and validation message layout

Use it when:

- more than one form repeats the same visual structure
- PrimeReact inputs share the same label and error layout

Do not turn it into a generic mega-form abstraction.

## 4. Page loading

Problem:

- pages repeat the same loading spinner and wrapper markup

Preferred pattern:

- a shared `PageLoading` component
- optional message support for fetch or transition states

## 5. Centralized API client

Problem:

- page files call `axios` directly with repeated base URLs and headers

Preferred pattern:

- one centralized client
- environment-driven base URL
- standardized timeout and headers
- shared interceptors only if the project really needs them

Keep the client in:

- `src/resources/<group>/lib` when it is domain-scoped
- or a common shared lib module when it is app-wide

## 6. Server action result handling

Next-specific rule:

- when using server actions, keep action functions in `services` if they represent domain operations
- keep client-side forms focused on collecting input and rendering results
- do not duplicate the same error normalization separately in every client page that calls a server action

## 7. Server and client split

Problem:

- a whole page becomes `"use client"` because one widget needs client behavior
- server-loaded data gets fetched again immediately on the client

Preferred pattern:

- keep the route entrypoint as a server component
- pass prepared data into a focused client component for interactive PrimeReact UI
- isolate client-only state and browser APIs inside the smallest practical subtree

## 8. What not to normalize

- page-local one-off state that is unlikely to repeat
- thin wrappers that only rename PrimeReact components
- generic abstractions that hide the route or domain structure
- tutorial-only patterns from learning chapters
