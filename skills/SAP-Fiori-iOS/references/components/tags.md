# Tags — Component Guidelines

> SAP Fiori for iOS | Category: Fiori SDK
> Figma: `https://www.figma.com/community/file/1450853598524410675/sap-fiori-for-ios-ui-kit`
> Development reference: UIKit `FUIButton` (tag style), SwiftUI `Tag`, `TagStyle`

## What is it?

Tags display quick, useful bits of information — keywords, labels, categories, or statuses. They are **not interactive** and serve as independent bits of information with a visually distinct appearance from plain text.

---

## When to Use

**Do:**
- Keep tag labels concise — max **2 words** recommended
- Use to display complementary information about an object

**Don't:**
- Write full sentences in a tag
- Place icons or images inside a tag
- Overload with excessive tags or very long text values

---

## Anatomy

```
A. Container (filled or outlined, RoundedRectangle cornerRadius: 8)
B. Label
```

**A. Container** — `RoundedRectangle(cornerRadius: 8)` filled or stroked.
**B. Label** — keyword or short information. Max 2 words.

---

## SDK Architecture

`Tag` is a first-class Fiori component styled via the `TagStyle` protocol and `.tagStyle(_:)` modifier.

### Built-in styles

| Style accessor | Appearance |
|---------------|-----------|
| Default (`TagFioriStyle`) | `.accentLabel10` text / `.accentBackground10` fill |
| `.tagStyle(.light)` | `.secondaryLabel` text, outlined with `.quaternaryLabel` border, no fill |
| `.tagStyle(.dark)` | `.primaryLabel` (dark constant) text, `.tertiaryLabel` fill |
| `.tagStyle(.outlined)` | `.secondaryLabel` text, `.tertiaryLabel` outlined border, no fill |

### Accent color mapping

| Hue | Label token | Background token |
|-----|------------|-----------------|
| Mango | `.accentLabel1` | `.accentBackground1` |
| Red | `.accentLabel2` | `.accentBackground2` |
| Raspberry | `.accentLabel3` | `.accentBackground3` |
| Pink | `.accentLabel4` | `.accentBackground4` |
| Indigo | `.accentLabel5` | `.accentBackground5` |
| Blue | `.accentLabel6` | `.accentBackground6` |
| Teal | `.accentLabel7` | `.accentBackground7` |
| Green | `.accentLabel8` | `.accentBackground8` |
| Grey | `.accentLabel9` | `.accentBackground9` |

Apply `.foregroundStyle()` and `.background()` directly on `Tag` to set accent colors:

```swift
Tag("Started")
    .foregroundStyle(Color.preferredColor(.accentLabel7))
    .background(RoundedRectangle(cornerRadius: 8)
        .fill(Color.preferredColor(.accentBackground7)))
```

Or use the `TagStyle` closure form:

```swift
Tag("PM01")
    .tagStyle { config in
        config.tag
            .font(.fiori(forTextStyle: .footnote))
            .foregroundStyle(Color.preferredColor(.accentLabel8))
            .background(RoundedRectangle(cornerRadius: 8)
                .fill(Color.preferredColor(.accentBackground8)))
    }
```

---

## Layout

Tags are **not interactive**. Multiple tags line up horizontally. Use `MHStack` for wrapping rows or `HStack` for non-wrapping.

---

## SwiftUI Code Examples

### Basic tag (default style)
```swift
import FioriSwiftUICore

Tag("Finance")  // default: accentLabel10 / accentBackground10
```

### Accent color tags (teal / green / mango)
```swift
// Teal
Tag("Started")
    .foregroundStyle(Color.preferredColor(.accentLabel7))
    .background(RoundedRectangle(cornerRadius: 8)
        .fill(Color.preferredColor(.accentBackground7)))

// Green
Tag("PM01")
    .foregroundStyle(Color.preferredColor(.accentLabel8))
    .background(RoundedRectangle(cornerRadius: 8)
        .fill(Color.preferredColor(.accentBackground8)))

// Mango
Tag("103-Repair")
    .foregroundStyle(Color.preferredColor(.accentLabel1))
    .background(RoundedRectangle(cornerRadius: 8)
        .fill(Color.preferredColor(.accentBackground1)))
```

### Light style (outlined)
```swift
Tag("Draft")
    .tagStyle(.light)
```

### Outlined style
```swift
Tag("Q4 2026")
    .tagStyle(.outlined)
```

### Horizontal tag row (wrapping) — `MHStack`
```swift
MHStack(spacing: 8, lineSpacing: 6) {
    Tag("Started")
        .foregroundStyle(Color.preferredColor(.accentLabel7))
        .background(RoundedRectangle(cornerRadius: 8)
            .fill(Color.preferredColor(.accentBackground7)))
    Tag("PM01")
        .foregroundStyle(Color.preferredColor(.accentLabel8))
        .background(RoundedRectangle(cornerRadius: 8)
            .fill(Color.preferredColor(.accentBackground8)))
    Tag("103-Repair")
        .foregroundStyle(Color.preferredColor(.accentLabel1))
        .background(RoundedRectangle(cornerRadius: 8)
            .fill(Color.preferredColor(.accentBackground1)))
}
.environment(\.tagLimit, 3)
.environment(\.tagsLineLimit, 1)
```

### Non-wrapping horizontal scroll
```swift
ScrollView(.horizontal, showsIndicators: false) {
    HStack(spacing: 6) {
        Tag("Finance")
        Tag("Q4 2026")
        Tag("EMEA")
    }
}
```

### Inside ObjectItem
```swift
ObjectItem {
    Text("Invoice #1234")
} tags: {
    Tag("Finance")
        .foregroundStyle(Color.preferredColor(.accentLabel7))
        .background(RoundedRectangle(cornerRadius: 8)
            .fill(Color.preferredColor(.accentBackground7)))
    Tag("Overdue")
        .foregroundStyle(Color.preferredColor(.accentLabel2))
        .background(RoundedRectangle(cornerRadius: 8)
            .fill(Color.preferredColor(.accentBackground2)))
}
```

---

## Figma Variants → SwiftUI

| Figma Property | Figma Value | SwiftUI |
|---------------|-------------|---------|
| Style | Filled | `Tag("...").background(RoundedRectangle(cornerRadius: 8).fill(...))` |
| Style | Light / Outlined | `.tagStyle(.light)` or `.tagStyle(.outlined)` |
| Color | Teal | `.accentLabel7` / `.accentBackground7` |
| Color | Green | `.accentLabel8` / `.accentBackground8` |
| Color | Mango | `.accentLabel1` / `.accentBackground1` |
| Color | Default | No modifier — uses `.accentLabel10` / `.accentBackground10` |
| Layout | Horizontal row | `HStack` or `ScrollView(.horizontal)` |
| Layout | Wrapping | `MHStack(spacing:lineSpacing:)` |
| Tag limit | Shown | `.environment(\.tagLimit, n)` |
| Line limit | Shown | `.environment(\.tagsLineLimit, n)` |
| Interactive | Never | No `.onTapGesture`, no `Button` |

---

## Do's ✓ / Don'ts ✗

**Do:**
- Use `Tag` — not manual `Text + Capsule` — so styling is consistent and theme-aware
- Apply accent color pairs together (label + background) — never mix pairs
- Use `.tagStyle(.light)` or `.tagStyle(.outlined)` for low-emphasis tags
- Allow tag rows to wrap using `MHStack` when space is constrained

**Don't:**
- Build tags manually with `Text` + `.clipShape(Capsule())` — use `Tag` and `TagStyle`
- Use `CustomTagStyle` — it is deprecated
- Make tags tappable/interactive
- Hardcode hex colors — always use `Color.preferredColor(...)` tokens

---

## Related Components

- [object-cell.md](object-cell.md) — uses tags in main content area
- [cards.md](cards.md) — tags in card header
- [filter-feedback-bar.md](filter-feedback-bar.md) — filter chips (similar appearance, but interactive)
