# PrimeReact Component Selection

Use this file to look up available PrimeReact components before writing custom UI. Always check the inventory below first.

## PrimeReact Component Inventory

Search this table before implementing any UI element. If a PrimeReact component exists, use it.

### Form Inputs

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

### Buttons and Actions

| Component | Use for |
|---|---|
| `Button` | Primary action button |
| `SplitButton` | Button with dropdown menu |
| `SpeedDial` | Floating action button with radial menu |

### Data Display

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

### Overlays and Dialogs

| Component | Use for |
|---|---|
| `Dialog` | Modal dialog |
| `ConfirmDialog` | Confirmation prompt (use with `confirmDialog()` API) |
| `ConfirmPopup` | Inline confirmation popup |
| `Sidebar` | Slide-in panel |
| `OverlayPanel` | Popup panel anchored to an element |
| `Tooltip` | Hover/focus tooltip |

### Feedback and Status

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

### Navigation and Menu

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

### Layout and Containers

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

### Media and Misc

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

## Decision Rules

1. **Search the inventory above first.** If a component exists for the need, use it directly.
2. **Customize via props/CSS, not by reimplementing.** Use `className`, `style`, `pt` (passthrough), or design token overrides.
3. **Custom only when nothing fits.** Follow PrimeReact design tokens for colors, spacing, and typography.
4. **Never wrap PrimeReact with no added logic.** Do not create `<AppButton>` that only re-exports `<Button>`.

---

## Per-Screen Component Suggestions

### Data Entry Screen

Purpose: collect user input, validate it, and submit it through a clear flow.

Recommended component groups:

- `Card` or `Panel` for a compact surface
- `InputText`, `InputNumber`, `Dropdown`, `Calendar`, or `Password` depending on field types
- `Message`, `InlineMessage`, or `Toast` for validation and status
- `Button` for primary and secondary actions
- `Divider` for separating sections when the screen combines instructions and actions

Keep validation and submission logic in forms, hooks, or services. Do not let the component layer redefine request behavior.

### Multi-Step or Wizard Screen

Purpose: guide the user through a sequence of steps with clear progress and checkpoints.

Recommended component groups:

- `Stepper` or `Steps` when the screen is explicitly multi-step
- `Dialog` only for confirmation or destructive retry flows
- `Message`, `InlineMessage`, or `Toast` for validation and status
- `ProgressSpinner` or `Skeleton` for async step transitions

Keep progression logic in app or service code rather than scattering it across many UI components.

### Upload or Processing Screen

Purpose: collect files or processing inputs and show status safely.

Recommended component groups:

- `FileUpload` only as a UI helper when backend upload contract is validated explicitly
- `InputText` or `Password` for supporting metadata
- `ProgressBar`, `ProgressSpinner`, `Message`, or `Toast` for status feedback
- `Tag` or `Badge` for lightweight state display

Verify multipart behavior, auth header attachment, and error handling against backend contract before relying on built-in upload behavior.

## General Selection Rules

- Prefer PrimeReact components for presentation and interaction consistency.
- Avoid pulling a second component library into the same screen without strong need.
- Treat network-capable components as optional helpers, not as the source of truth for request behavior.
- Keep validation, auth, token flow, and API mapping in app code aligned with backend contract.
