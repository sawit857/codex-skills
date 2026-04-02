# PrimeReact component selection

Use this file to look up available PrimeReact components before writing custom UI. Always check the inventory below first.

## PrimeReact component inventory

Search this table before implementing any UI element. If a PrimeReact component exists, use it.

### Form inputs

| Component | Use for |
|---|---|
| `InputText` | Single-line text input |
| `InputTextarea` | Multi-line text input |
| `InputNumber` | Numeric input with formatting |
| `InputMask` | Masked input (phone, ID card, etc.) |
| `InputOtp` | OTP / verification code input |
| `Password` | Password input with toggle mask |
| `Dropdown` | Single-select dropdown |
| `MultiSelect` | Multi-select dropdown |
| `AutoComplete` | Text input with suggestions |
| `Calendar` | Date/time picker |
| `Checkbox` | Single or grouped checkboxes |
| `RadioButton` | Radio button group |
| `InputSwitch` | Toggle switch |
| `SelectButton` | Button-style single/multi select |
| `ToggleButton` | Two-state toggle button |
| `Rating` | Star rating input |
| `Slider` | Range slider |
| `ColorPicker` | Color selection |
| `Listbox` | Scrollable selection list |
| `TreeSelect` | Tree-structured dropdown |
| `CascadeSelect` | Cascading dropdown |
| `Chips` | Tag/token input |
| `Knob` | Circular value input |
| `KeyFilter` | Input filtering by pattern (used as directive on InputText) |

### Buttons & actions

| Component | Use for |
|---|---|
| `Button` | Primary action button |
| `SplitButton` | Button with dropdown menu |
| `SpeedDial` | Floating action button with radial menu |

### Data display

| Component | Use for |
|---|---|
| `DataTable` | Tabular data with sort, filter, pagination, selection |
| `TreeTable` | Hierarchical tabular data |
| `DataView` | Card/list view of data collections |
| `OrderList` | Reorderable list |
| `PickList` | Dual-list transfer |
| `Timeline` | Chronological event display |
| `Tree` | Hierarchical tree view |
| `VirtualScroller` | Virtualized large list rendering |
| `DataScroller` | Lazy-loading scrollable list |

### Overlays & dialogs

| Component | Use for |
|---|---|
| `Dialog` | Modal dialog |
| `ConfirmDialog` | Confirmation prompt (use with `confirmDialog()` API) |
| `ConfirmPopup` | Inline confirmation popup |
| `Sidebar` | Slide-in panel |
| `OverlayPanel` | Popup panel anchored to an element |
| `Tooltip` | Hover/focus tooltip |

### Feedback & status

| Component | Use for |
|---|---|
| `Toast` | Transient notification messages |
| `Message` | Inline static message (info/success/warn/error) |
| `InlineMessage` | Compact inline message next to form fields |
| `ProgressBar` | Determinate/indeterminate progress bar |
| `ProgressSpinner` | Loading spinner |
| `Skeleton` | Content placeholder while loading |
| `BlockUI` | Block interaction on a region during loading |
| `Badge` | Numeric badge on icons/buttons |
| `Tag` | Labeled status tag |

### Navigation & menu

| Component | Use for |
|---|---|
| `Menubar` | Horizontal top navigation bar |
| `Menu` | Popup menu |
| `MegaMenu` | Multi-column dropdown menu |
| `TabMenu` | Tab-style navigation |
| `Steps` | Step indicator (read-only step bar) |
| `Stepper` | Interactive multi-step wizard |
| `Breadcrumb` | Breadcrumb navigation |
| `PanelMenu` | Accordion-style vertical menu |
| `TieredMenu` | Nested tiered popup menu |
| `ContextMenu` | Right-click context menu |
| `Dock` | macOS-style dock menu |
| `TabView` | Tabbed content panels (also layout) |

### Layout & containers

| Component | Use for |
|---|---|
| `Card` | Content card with header/body/footer |
| `Panel` | Collapsible content panel |
| `Accordion` | Accordion panels |
| `TabView` | Tabbed content panels |
| `Fieldset` | Grouped content with legend |
| `Divider` | Visual separator line |
| `Splitter` | Resizable split panels |
| `ScrollPanel` | Custom scrollbar panel |
| `Toolbar` | Grouped action bar |
| `ScrollTop` | Scroll-to-top button |

### Media & misc

| Component | Use for |
|---|---|
| `Image` | Image with preview |
| `Carousel` | Content carousel/slider |
| `Galleria` | Image gallery |
| `Avatar` | User avatar circle/square |
| `AvatarGroup` | Grouped avatars |
| `Chip` | Removable tag/chip |
| `FileUpload` | File upload (verify backend contract before using built-in request behavior) |
| `Terminal` | Terminal-style text UI |

## Decision rules

1. **Search the inventory above first.** If a component exists for the need, use it directly.
2. **Customize via props/CSS, not by reimplementing.** Use `className`, `style`, `pt` (passthrough), or design token overrides.
3. **Custom only when nothing fits.** Follow PrimeReact design tokens for colors, spacing, and typography.
4. **Never wrap PrimeReact with no added logic.** Do not create `<AppButton>` that only re-exports `<Button>`.

---

## Per-screen component suggestions

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
