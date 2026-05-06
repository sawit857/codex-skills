# Shared Patterns for the Team Structure

Use this file when the same UI, error, loading, or API pattern appears in multiple screens or groups.

## Placement guidance

Shared patterns can live in one of two places:

- in `src/app` when they are app-wide
- in `src/resources/<group>/...` when they are domain-specific

Default rule:

- app-wide provider or toast/message infrastructure: keep close to app composition
- domain-specific shared UI: keep inside `src/resources/<group>/components`
- shared API clients and helpers: keep inside `src/resources/<group>/lib` or a common app-wide lib module

## 1. Shared message or toast strategy

Problem:

- each screen creates its own `useRef<Toast>` or `useRef<Messages>`
- success and error handling are repeated

Preferred pattern:

- one shared provider or helper for app-wide feedback
- screen-level code calls the shared API instead of re-creating toast refs

For Vite:

- wire the provider near the top-level app shell
- keep only truly local message state inside individual screens

## 2. API error extraction

Problem:

- screens repeatedly parse Axios errors inline
- fallback messages differ arbitrarily across the app

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

- screens repeat the same loading spinner and wrapper markup

Preferred pattern:

- a shared `PageLoading` component
- optional message support for fetch or transition states

## 5. Centralized API client

Problem:

- screen files call `axios` directly with repeated base URLs and headers

Preferred pattern:

- one centralized client
- environment-driven base URL
- standardized timeout and headers
- shared interceptors only if the project really needs them

Keep the client in:

- `src/resources/<group>/lib` when it is domain-scoped
- or a common shared lib module when it is app-wide

## 6. Service module result handling

Vite-specific rule:

- keep domain operations in `services`
- keep screen-level forms focused on collecting input and rendering results
- do not duplicate the same error normalization separately in every screen that calls the same operation

## 7. Route and screen split

Problem:

- route definitions start carrying page logic
- screen components fetch and transform the same data repeatedly

Preferred pattern:

- keep route composition inside `src/app`
- keep screen orchestration thin
- pass prepared data and handlers into focused UI components when reuse matters

## 8. What not to normalize

- screen-local one-off state that is unlikely to repeat
- thin wrappers that only rename PrimeReact components
- generic abstractions that hide the route or domain structure
- tutorial-only patterns from learning examples
