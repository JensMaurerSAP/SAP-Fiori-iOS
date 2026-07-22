# Buttons — Component Guidelines

> SAP Fiori for iOS | Category: Fiori SDK
> Figma: `https://www.figma.com/community/file/1450853598524410675/sap-fiori-for-ios-ui-kit` (Buttons page)
> Development reference: UIKit `FUIButton`, SwiftUI `FioriButton`

## What is it?

Buttons allow users to perform actions, make decisions, or begin a process. The label communicates the action that will be initiated.

---

## When to Use

**Do:**
- Use buttons only for actions
- Use simple, specific labels: "Create", "Edit", "Save", "Approve", "Reject", "Submit", "Cancel"
- Use short, meaningful labels in command form ("Save", not "Saving")
- Use toggle buttons to activate/deactivate or switch states (Follow/Unfollow, Select/Selected)
- Use secondary tint style for positive actions
- Use secondary negative style for destructive actions

**Don't:**
- Show too many buttons — use a segmented control when selecting from a small group
- Use green for positive actions — use tint style
- Truncate button labels

---

## Anatomy

A button consists of a label, an image, or both — on a filled or unfilled background.

Three content configurations:
- Image only
- Label only
- Image + label (image position: `.leading`, `.trailing`, `.top`, or `.bottom`)

---

## Button Types

### A. Primary Button
The most important action in the view. **Only one primary button per screen.**

```swift
FioriButton { _ in Text("Sign In") }
    .fioriButtonStyle(FioriPrimaryButtonStyle())
```

### B. Secondary Button
Optional or lower-priority actions. Paired alongside primary or used standalone.

```swift
FioriButton { _ in Text("Dismiss") }
    .fioriButtonStyle(FioriSecondaryButtonStyle())
```

### C. Tertiary Button
Lowest-priority actions, or actions inside the navigation bar.

```swift
FioriButton { _ in Text("Learn More") }
    .fioriButtonStyle(FioriTertiaryButtonStyle())
```

### D. Navigation Button
For navigation bar items. No background; tint-colored label.

```swift
FioriButton { _ in Text("Edit") }
    .fioriButtonStyle(FioriNavigationButtonStyle())
```

### E. Toggle Button
Switches between two persistent states without navigation. Use `isSelectionPersistent: true` — the button toggles between `.normal` and `.selected` states on each tap.

Common pairs: Follow/Unfollow, Select/Selected, Bookmark/Bookmarked

```swift
FioriButton(isSelectionPersistent: true,
            action: { state in
                isFollowing = (state == .selected)
            },
            label: { state in
                Text(state == .selected ? "Following" : "Follow")
            },
            image: { state in
                Image(systemName: state == .selected ? "person.fill.checkmark" : "person.badge.plus")
            })
    .fioriButtonStyle(FioriSecondaryButtonStyle())
```

---

## SDK Architecture

### `FioriButton` vs standard `Button`

| Use case | Type | Style protocol |
|----------|------|---------------|
| Fiori-styled buttons | `FioriButton` | `FioriButtonStyle` applied via `.fioriButtonStyle(_:)` |
| Standard SwiftUI `Button` | `Button` | `PrimitiveButtonStyle` applied via `.buttonStyle(_:)` |

The two style systems are separate. Never apply `FioriPrimaryButtonStyle` to a plain `Button` — use `PrimaryButtonStyle` instead (see below).

### Default style

When no `.fioriButtonStyle(_:)` is applied, `FioriButton` defaults to `FioriPrimaryButtonStyle`.

---

## `FioriButton` Initializers

```swift
// Most common: per-state label ViewBuilder
FioriButton(
    isSelectionPersistent: Bool = false,
    action: ((UIControl.State) -> Void)? = nil,
    label: @escaping (UIControl.State) -> any View
)

// With image
FioriButton(
    isSelectionPersistent: Bool = false,
    action: ((UIControl.State) -> Void)? = nil,
    label: @escaping (UIControl.State) -> any View = { _ in EmptyView() },
    image: @escaping (UIControl.State) -> any View = { _ in EmptyView() },
    imagePosition: FioriButtonImagePosition = .leading,
    imageTitleSpacing: CGFloat = 8.0
)

// Attributed string (all states same label)
FioriButton(isSelectionPersistent: Bool = false,
            title: AttributedString,
            action: ((UIControl.State) -> Void)? = nil)

// Per-state attributed string
FioriButton(isSelectionPersistent: Bool = false,
            title: @escaping (UIControl.State) -> AttributedString,
            action: ((UIControl.State) -> Void)? = nil)
```

### States

For **non-persistent** buttons (`isSelectionPersistent: false`): use `.normal`, `.highlighted`, `.disabled`.
For **persistent** (toggle) buttons (`isSelectionPersistent: true`): use `.normal`, `.selected`, `.disabled`.

### `FioriButtonImagePosition`

```swift
public enum FioriButtonImagePosition {
    case top      // image above label
    case leading  // image left of label (default)
    case bottom   // image below label
    case trailing // image right of label
}
```

---

## Button Styles (`FioriButtonStyle`)

Apply via `.fioriButtonStyle(_:)`. All styles accept `maxWidth: CGFloat?` and `minHeight: CGFloat?` parameters. All buttons have a minimum height of **38 pt** and minimum width of **44 pt**.

**Shape:** `Capsule` on iOS 26+, `RoundedRectangle(cornerRadius: 8)` on earlier versions.

### `FioriPrimaryButtonStyle`

```swift
FioriPrimaryButtonStyle(
    _ maxWidth: CGFloat? = nil,
    minHeight: CGFloat? = nil,
    loadingState: FioriButtonLoadingState = .unspecified,
    isLoading: Bool = false
)
```

| State | Foreground | Background |
|-------|-----------|-----------|
| `.normal` | `.base2` | `.tintColor` |
| `.highlighted` / `.selected` | `.base2` | `.tintColorTapState` |
| `.disabled` | `.separator` | `.tertiaryFill` |

### `FioriSecondaryButtonStyle`

```swift
FioriSecondaryButtonStyle(
    colorStyle: FioriButtonColorStyle = .tint,
    maxWidth: CGFloat? = nil,
    minHeight: CGFloat? = nil,
    loadingState: FioriButtonLoadingState = .unspecified,
    isLoading: Bool = false
)
```

| `colorStyle` | State | Foreground | Background |
|-------------|-------|-----------|-----------|
| `.tint` | `.normal` | `.tintColor2` | `.informationBackground` |
| `.tint` | `.highlighted/.selected` | `.tintColorTapState` | `.informationBackground` |
| `.normal` | `.normal` | `.secondaryLabel` | `.secondaryFill` |
| `.normal` | `.highlighted/.selected` | `.secondaryLabel` | `.secondaryFill` |
| `.negative` | `.normal` | `.negativeLabel` | `.negativeBackground` |
| `.negative` | `.highlighted/.selected` | `.negativeLabelTapState` | `.negativeBackgroundTapState` |
| any | `.disabled` | `.separator` | `.tertiaryFill` |

### `FioriTertiaryButtonStyle`

```swift
FioriTertiaryButtonStyle(
    colorStyle: FioriButtonColorStyle = .tint,
    maxWidth: CGFloat? = nil,
    minHeight: CGFloat? = nil,
    loadingState: FioriButtonLoadingState = .unspecified,
    isLoading: Bool = false
)
```

| `colorStyle` | State | Foreground | Background |
|-------------|-------|-----------|-----------|
| `.tint` | `.normal` | `.tintColor2` | `.clear` |
| `.tint` | `.highlighted/.selected` | `.tintColorTapState` | `.secondaryFill` |
| `.normal` | `.normal` | `.primaryLabel` | `.clear` |
| `.normal` | `.highlighted/.selected` | `.secondaryLabel` | `.secondaryFill` |
| `.negative` | `.normal` | `.negativeLabel` | `.clear` |
| `.negative` | `.highlighted/.selected` | `.negativeLabel` | `.secondaryFill` |
| any | `.disabled` | `.separator` | `.clear` |

### `FioriNavigationButtonStyle`

For navigation bar actions. No `maxWidth`/`minHeight` parameters.

```swift
FioriNavigationButtonStyle(colorStyle: FioriButtonColorStyle = .tint)
```

| State | Foreground | Background |
|-------|-----------|-----------|
| `.normal` | `.tintColor` | `.clear` |
| `.highlighted` / `.selected` | `.tintColorTapState` | `.clear` |
| `.disabled` | `.quaternaryLabel` | `.clear` |

### `FioriPlainButtonStyle`

No fill, no border. Tint-colored text only.

| State | Foreground | Background |
|-------|-----------|-----------|
| `.normal` | `.tintColor` | `.primaryBackground` |
| `.highlighted` / `.selected` | `.tintColorTapState` | `.primaryBackground` |
| `.disabled` | `.separator` | `.primaryBackground` |

### `FioriGlassButtonStyle` (iOS 26+)

```swift
@available(iOS 26.0, macOS 26.0, tvOS 26.0, watchOS 26.0, *)
@available(visionOS, unavailable)
FioriGlassButtonStyle(
    glassEffect: FioriButtonGlassEffectStyle = .plain,
    maxWidth: CGFloat? = nil,
    minHeight: CGFloat? = nil
)
```

```swift
public enum FioriButtonGlassEffectStyle {
    case plain         // untinted glass; primaryLabel foreground
    case tint          // tinted glass; white foreground
    case systemManaged // no glass applied; system provides it (use in toolbars)
}
```

### `FioriButtonColorStyle`

```swift
public enum FioriButtonColorStyle {
    case normal    // neutral grey tones
    case tint      // app tint color (default)
    case negative  // destructive red tones
}
```

---

## Loading State

Built-in loading state support. Use `FioriButtonLoadingState` with Primary, Secondary, or Tertiary styles:

```swift
public enum FioriButtonLoadingState {
    case unspecified  // default; no loading UI
    case processing   // shows a circular ProgressView in place of image
    case success      // shows a checkmark icon
}
```

```swift
@State private var loadingState: FioriButtonLoadingState = .unspecified

FioriButton { _ in Text("Submit") }
    .fioriButtonStyle(FioriPrimaryButtonStyle(
        maxWidth: .infinity,
        loadingState: loadingState
    ))
    .onTapGesture {
        Task {
            loadingState = .processing
            try await submitForm()
            loadingState = .success
            try await Task.sleep(for: .seconds(3))
            loadingState = .unspecified
        }
    }
```

When `loadingState` is `.processing` or `.success`, the `containerView(_:)` on `FioriButtonStyleConfiguration` substitutes the image view with a `ProgressView` or success icon respectively. **Only trigger the loading animation when processing time exceeds 1000ms.**

The `isLoading: Bool` parameter (separate from `loadingState`) dims the entire button to 25% opacity and applies a greyed style — use for skeleton loading states.

---

## Custom `FioriButtonStyle`

```swift
struct CustomFioriButtonStyle: FioriButtonStyle {
    func makeBody(configuration: FioriButtonStyleConfiguration) -> some View {
        let color: Color
        switch configuration.state {
        case .normal:
            color = .preferredColor(.tintColor)
        case .highlighted, .selected:
            color = .preferredColor(.tintColorTapState)
        default:
            color = .preferredColor(.tertiaryFill)
        }

        return configuration.containerView(.unspecified)
            .foregroundColor(.white)
            .padding(.horizontal, 16)
            .padding(.vertical, 8)
            .background(Capsule().fill(color))
    }
}
```

`FioriButtonStyleConfiguration` provides:
- `configuration.state: UIControl.State` — current button state
- `configuration.label` — label for current state
- `configuration.image` — image for current state
- `configuration.label(for:)` / `configuration.image(for:)` — label/image for any state
- `configuration.imagePosition: FioriButtonImagePosition`
- `configuration.imageTitleSpacing: CGFloat`
- `configuration.containerView(_: FioriButtonLoadingState)` — arranges image + label per `imagePosition`, substituting loading/success UI when needed

Apply to a hierarchy:
```swift
someView.fioriButtonStyle(CustomFioriButtonStyle())
```

---

## Standard `Button` Styles (not `FioriButton`)

These `PrimitiveButtonStyle` types apply to SwiftUI's native `Button`, not `FioriButton`:

| Style | Usage |
|-------|-------|
| `PrimaryButtonStyle` | `.buttonStyle(PrimaryButtonStyle())` on a `Button` |
| `SecondaryButtonStyle(colorStyle:)` | `.buttonStyle(SecondaryButtonStyle())` |
| `TertiaryButtonStyle(colorStyle:)` | `.buttonStyle(TertiaryButtonStyle())` |
| `StatefulButtonStyle(color:depressedColor:disabledColor:)` | Fully custom colors on a `Button` |

```swift
// On a standard Button
Button("Approve") { approve() }
    .buttonStyle(PrimaryButtonStyle())
```

---

## Destructive Role via `configureStandardButton`

To use `Button(role: .destructive)` semantics with `FioriButton`:

```swift
FioriButton(action: { _ in deleteItem() }) { _ in Text("Delete") }
    .fioriButtonStyle(FioriSecondaryButtonStyle(colorStyle: .negative))
    .configureStandardButton { fioriAction in
        Button(role: .destructive, action: fioriAction) { EmptyView() }
    }
```

This ensures the system respects the destructive role (e.g. in `.confirmationDialog`) while Fiori handles visual styling.

---

## Button Sizes

### Auto-width (default)
Content-sized width. Minimum height **38 pt**. Use inside components (cards, list rows, object cells).

### Full-width
Pass `maxWidth: .infinity` to the style initializer — do not use `.frame(maxWidth: .infinity)` on the button itself:

```swift
FioriButton { _ in Text("Continue") }
    .fioriButtonStyle(FioriPrimaryButtonStyle(.infinity))
    .padding(.horizontal, 16)
```

### Custom height
```swift
FioriButton { _ in Text("Sign In") }
    .fioriButtonStyle(FioriPrimaryButtonStyle(nil, minHeight: 44))
```

> Minimum touch area for any button: **44 × 44 pt**

---

## SwiftUI Code Examples

### Primary + secondary stack

```swift
VStack(spacing: 12) {
    FioriButton { _ in Text("Continue") }
        .fioriButtonStyle(FioriPrimaryButtonStyle(.infinity))

    FioriButton { _ in Text("Cancel") }
        .fioriButtonStyle(FioriSecondaryButtonStyle(maxWidth: .infinity))
}
.padding(.horizontal, 16)
```

### Approve / Reject (tint + negative)

```swift
HStack(spacing: 12) {
    FioriButton { _ in Text("Reject") }
        .fioriButtonStyle(FioriSecondaryButtonStyle(colorStyle: .negative))

    FioriButton { _ in Text("Approve") }
        .fioriButtonStyle(FioriSecondaryButtonStyle())
}
```

### Navigation bar button

```swift
.toolbar {
    ToolbarItem(placement: .primaryAction) {
        FioriButton(action: { _ in edit() }) { _ in
            Image(systemName: "square.and.pencil")
        }
        .fioriButtonStyle(FioriNavigationButtonStyle())
        .accessibilityLabel("Edit")
    }
}
```

### Button with image (leading)

```swift
FioriButton(
    action: { _ in scan() },
    label: { _ in Text("Scan") },
    image: { _ in Image(systemName: "camera.viewfinder") },
    imagePosition: .leading,
    imageTitleSpacing: 6
)
.fioriButtonStyle(FioriSecondaryButtonStyle())
```

### Disabled state

```swift
FioriButton { _ in Text("Submit") }
    .fioriButtonStyle(FioriPrimaryButtonStyle())
    .disabled(!formIsValid)
```

### Per-state label

```swift
FioriButton(action: { state in
    print("Tapped with state: \(state)")
}) { state in
    switch state {
    case .normal:    Text("Normal")
    case .highlighted: Text("Pressed")
    default:         Text("Disabled")
    }
}
.fioriButtonStyle(FioriSecondaryButtonStyle())
```

### Glass button (iOS 26+)

```swift
if #available(iOS 26.0, *) {
    FioriButton { _ in Text("Confirm") }
        .fioriButtonStyle(FioriGlassButtonStyle(glassEffect: .tint, maxWidth: .infinity))
}
```

---

## Figma Variants → SwiftUI

| Figma Property | Figma Value | SwiftUI |
|---------------|-------------|---------|
| Type | Primary | `FioriPrimaryButtonStyle()` |
| Type | Secondary | `FioriSecondaryButtonStyle()` |
| Type | Tertiary | `FioriTertiaryButtonStyle()` |
| Type | Navigation | `FioriNavigationButtonStyle()` |
| Type | Glass | `FioriGlassButtonStyle()` (iOS 26+) |
| Style | Tint | `colorStyle: .tint` (default) |
| Style | Normal | `colorStyle: .normal` |
| Style | Negative | `colorStyle: .negative` |
| State | Disabled | `.disabled(true)` |
| State | Loading / Processing | `loadingState: .processing` |
| State | Loading / Success | `loadingState: .success` |
| Width | Auto | Default — content-sized |
| Width | Full | `FioriPrimaryButtonStyle(.infinity)` |
| Toggle | Yes | `isSelectionPersistent: true` |
| Content | Label only | `label: { _ in Text("...") }` |
| Content | Image only | `image: { _ in Image(...) }` |
| Content | Label + image | Both `label:` and `image:` closures |
| Image position | Leading | `imagePosition: .leading` (default) |
| Image position | Trailing / Top / Bottom | `imagePosition: .trailing/.top/.bottom` |

---

## Do's ✓ / Don'ts ✗

**Do:**
- One Primary button per screen maximum
- Use `colorStyle: .negative` on Secondary/Tertiary for destructive actions — not a `.tint()` modifier
- Keep labels in command form and untruncated
- Meet 44 × 44 pt minimum touch target
- Apply loading state only when processing > 1000ms
- Show `.success` state for ~3 seconds before returning to `.unspecified`
- Use `FioriNavigationButtonStyle` for navigation bar items — not `FioriTertiaryButtonStyle`
- Apply `PrimaryButtonStyle` / `SecondaryButtonStyle` to standard `Button`, not to `FioriButton`

**Don't:**
- Use `.tint(Color.preferredColor(.negativeLabel))` to make a negative button — use `colorStyle: .negative`
- Apply `FioriPrimaryButtonStyle` to a plain SwiftUI `Button`
- Use `FioriGlassButtonStyle` on visionOS — it is unavailable there
- Stack more than 3 buttons vertically without reconsidering the information architecture

---

## Related Components

- [navigation-bar.md](navigation-bar.md) — `FioriNavigationButtonStyle` usage in toolbars
- [forms.md](forms.md) — form submission button patterns
