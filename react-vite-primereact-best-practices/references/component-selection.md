# PrimeReact component selection

Use this file when choosing PrimeReact components for the project's tokenized UI flows.

## ggaureg-ui

Purpose: accept a tokenized entry flow for Google Authenticator registration and guide the user into OTP enrollment.

Recommended component groups:

- `Card` or `Panel` for a compact entry surface
- `Message` for flow status or validation summary
- `Button` for primary continue action
- `ProgressSpinner` for token validation or loading state
- `Divider` for visual separation when the screen has instructions and actions

Use plain routing and API code for token validation. Do not let the component layer decide the token lifecycle.

## certreg-ui

Purpose: handle certificate registration after the `url token` entry flow is resolved.

Recommended component groups:

- `Stepper` or `Steps` when the screen is explicitly multi-step
- `FileUpload` only as a UI helper when backend upload contract is validated explicitly
- `Password` for `.pfx/.p12` password input
- `InputText` for plain string fields
- `Message`, `InlineMessage`, or `Toast` for validation and status
- `Dialog` only for confirmation or destructive retry flows

Verify multipart behavior, auth header attachment, and error handling against backend contract before relying on built-in upload behavior.

## signdoc-ui

Purpose: handle tokenized document signing flows and optional `signingSessionId` reuse.

Recommended component groups:

- `Stepper` or `Steps` for sign flow progression when the UI is task-driven
- `FileUpload` or file picker UI only if upload/request behavior is checked against the API contract
- `Password`, `InputText`, and OTP inputs for sign inputs
- `DataTable` for document or sign result lists only when needed
- `Tag`, `Badge`, or `Message` for signing status and certificate state
- `Toast` for transient success/failure feedback

Keep `signingSessionId` logic in app/service code, not in component abstractions.

## General selection rules

- Prefer PrimeReact components for presentation and interaction consistency.
- Avoid pulling a second component library into the same screen without strong need.
- Treat network-capable components as optional helpers, not as the source of truth for request behavior.
- Keep validation, auth, token flow, and API mapping in app code aligned with backend contract.
