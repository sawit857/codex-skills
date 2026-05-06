# Next.js + PrimeReact Security Reference

Use this file when the task touches authentication, uploads, request boundaries, sensitive values, or dependency review.

## Token and Session Boundaries

Keep these roles separate:

- authentication token
  - used for protected API calls
  - usually sent as `Authorization: Bearer <token>` when required by backend contract
- route parameter or search parameter
  - used to resolve page state or navigation context
  - do not treat it as a replacement for authentication
- cookie or server session
  - used when the application relies on server-managed auth or request state
  - keep it distinct from client-managed UI state
- temporary session identifier
  - used only for short-lived flow or recovery state when explicitly needed
  - keep it separate from authentication state

## Server and Client Boundary

Next.js makes boundary mistakes easy to hide. Treat them as security and correctness issues.

- do not import server-only modules into client components
- do not expose secrets through `NEXT_PUBLIC_*` unless the value is meant for the browser
- keep secret-bearing logic on the server whenever possible
- avoid moving protected request logic into client components just for convenience

## PrimeReact Boundary

PrimeReact is a presentation and interaction layer. It must not silently redefine:

- auth storage strategy
- token passing rules
- upload contract
- error mapping contract
- request headers required by the backend

## Network-Capable Components

Treat components with built-in request behavior carefully.

Examples:

- upload widgets
- lazy-loading tables with server callbacks
- components that assume a default request format

For these components, verify:

- request path and method match backend contract
- authorization headers are attached correctly
- multipart or JSON payload shape matches backend expectation
- error responses are mapped without leaking internal details

## Dependency Security Checks

Before merge:

- pin exact versions in `package.json`
- commit lock file
- run `npm audit`
- review direct and transitive dependencies
- avoid stacking multiple large UI libraries together without explicit need

## Sensitive Data Rules

- do not log tokens, passwords, OTPs, or secret values in browser or server logs unless explicitly required and scrubbed
- do not persist sensitive values in browser storage unless the backend contract explicitly requires it
- prefer server-managed or short-lived in-memory state when possible
- keep route-bound parameters or temporary session identifiers only for the flows that explicitly require them

## Offline-Only Asset Loading

The client browser may not have outbound internet access. All assets must be self-contained.

- do not load any script, stylesheet, font, or icon from an external CDN or remote URL
- do not use Google Fonts, unpkg, cdnjs, jsdelivr, or similar services
- PrimeIcons and all icon sets must be imported from locally installed npm packages
- font files must be hosted inside the project or bundled through the application
- after build, verify the output has no references to external domains unless the project explicitly requires them
