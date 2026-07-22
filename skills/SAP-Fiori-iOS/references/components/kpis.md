# KPIs — Component Guidelines

> SAP Fiori for iOS | Category: Fiori SDK
> Figma: `https://www.figma.com/community/file/1450853598524410675/sap-fiori-for-ios-ui-kit` (KPI page)
> Development reference: UIKit `FUIKPIView`, SwiftUI `_KPIItem`, `_KPIProgressItem`

## What is it?

Key performance indicators (KPIs) display measurable values to evaluate success or summarize an object. They can be used in headers, content areas, object cells, and modal windows.

---

## Placement Rules

| Location | Layout | Tappable |
|----------|--------|---------|
| Header / content area | Horizontal with fixed padding, center-aligned | Yes (optional) |
| Modal window | Vertically stacked, max 2 per row | No |
| Object cell / card | Inline | No |

---

## Anatomy

```
        [B. Unit]  A. KPI Value  [B. Unit]
                   C. KPI Label
              D. [████████░░] Progress
```

**A. KPI (metric)** — most prominent element, always above the label. **Mandatory.** May show a single value or a range (e.g. "1 of 2").

**B. KPI Unit Label** (optional) — clarifies the metric (e.g. $, h, M). Placed left or right of the metric. Maximum **2 unit labels**.

**C. KPI Label** — mandatory. Communicates what the KPI represents. Keep concise. For the progress view, can appear inside the indicator or below it.

**D. Progress View** (optional) — visual representation of completeness, 0–100%.

---

## Interaction

- **Header / content area KPIs** — can be read-only or tappable. When tappable, the KPI value appears in **tint color**. Tap navigates to a modal or a filtered list.
- **Cell KPIs** — never tappable.

---

## Adaptive Design

- Supported in compact and regular widths
- KPIs have defined minimum and maximum widths
- **Compact header**: 1–2 KPIs; overflow → multiple pages with page control
- **Regular header**: 4–5 KPIs depending on their width (see [kpi-header.md](kpi-header.md))

---

## Variations

### Standard KPI
Most common type — shows quantity, percentage, or value. Optional unit label (max 2).

The convenience initializers use `KPIItemData` to drive the formatted value:

```swift
import FioriSwiftUICore

// Using KPIItemData convenience initializer (recommended)
_KPIItem(data: .components([.metric("1.4"), .unit("M"), .unitInFront("$")]),
         subtitle: "Revenue")

// Plain value
_KPIItem(data: .components([.metric("342")]), subtitle: "Orders")

// Percent (0.0–1.0)
_KPIItem(data: .percent(0.64), subtitle: "Win Rate")

// ViewBuilder form — full control over kpi and subtitle slots
_KPIItem(kpi: {
    Text("$1.4M")
        .font(.fiori(forTextStyle: .largeKPI, weight: .light))
        .foregroundStyle(Color.preferredColor(.primaryLabel))
}, subtitle: {
    Text("Revenue")
        .font(.fiori(forTextStyle: .subheadline))
        .foregroundStyle(Color.preferredColor(.secondaryLabel))
})
```

`_KPIItem` renders with `maxWidth: 216` and center-aligns both `kpi` and `subtitle`. When an `action` is provided, the entire item is wrapped in a `Button` using `ButtonContainerStyle` — tapping changes the foreground color to `.tintColor` (normal) or `.tintColorTapState` (pressed); disabled uses `.primaryLabel`.

### Time KPI
Shows a breakdown of time. Always include a unit label (h for hours, m for minutes).

```swift
_KPIItem(kpi: {
    HStack(alignment: .lastTextBaseline, spacing: 2) {
        Text("4")
            .font(.fiori(forTextStyle: .largeKPI, weight: .light))
        Text("h")
            .font(.fiori(forTextStyle: .title2))
            .foregroundStyle(Color.preferredColor(.secondaryLabel))
        Text("30")
            .font(.fiori(forTextStyle: .largeKPI, weight: .light))
        Text("m")
            .font(.fiori(forTextStyle: .title2))
            .foregroundStyle(Color.preferredColor(.secondaryLabel))
    }
    .foregroundStyle(Color.preferredColor(.primaryLabel))
}, subtitle: {
    Text("Duration")
        .foregroundStyle(Color.preferredColor(.secondaryLabel))
})
```

### KPI with Icon
Icon sits to the **left** of the numeric value. Maximum **1 unit label** when using an icon.

```swift
_KPIItem(kpi: {
    HStack(alignment: .lastTextBaseline, spacing: 6) {
        Image(systemName: "cart")
            .font(.fiori(forTextStyle: .title2))
            .foregroundStyle(Color.preferredColor(.tintColor))
        Text("342")
            .font(.fiori(forTextStyle: .largeKPI, weight: .light))
    }
    .foregroundStyle(Color.preferredColor(.primaryLabel))
}, subtitle: {
    Text("Orders")
        .foregroundStyle(Color.preferredColor(.secondaryLabel))
})
```

### KPI Progress View (small + large)
Shows completeness visually (0–100%). Two sizes:
- **Small** — cards, content areas
- **Large** — headers, prominent content areas (see [kpi-header.md](kpi-header.md) for header rules)

Label can appear **inside** the progress indicator or **below** it (wraps to 2 lines if needed).

The progress item is `_KPIProgressItem`. The default style is `FioriCircularProgressViewStyle`, customizable via `.kpiProgressViewStyle(_:)`. The item renders at 130×130 pt (circular area) + footnote below, constrained to `maxWidth: 216`.

```swift
// Using KPIItemData convenience initializer
_KPIProgressItem(data: .percent(0.72), subtitle: "Quota Attainment", footnote: "$1.4M / $1.95M")

// ViewBuilder form — full control
_KPIProgressItem(kpi: {
    Text("72%")
        .font(.fiori(forTextStyle: .KPI, weight: .light))
}, fraction: 0.72, subtitle: {
    Text("Quota Attainment")
}, footnote: {
    Text("$1.4M / $1.95M")
        .foregroundStyle(Color.preferredColor(.secondaryLabel))
} progress: {
    ProgressView(value: 0.72)
        .tint(Color.preferredColor(.tintColor))
}

// Small — label below, for cards
KPIProgressItem {
    Text("64%")
        .font(.fiori(forTextStyle: .headline, weight: .light))
} subtitle: {
    Text("On-time Delivery")
} progress: {
    ProgressView(value: 0.64)
        .tint(Color.preferredColor(.positiveLabel))
}
.proposedViewSize: .small
```

---

## Tappable KPI (navigates to modal or filtered list)

Pass an `action` closure to `_KPIItem` — it wraps the item in a `Button` and renders the value in `.tintColor` automatically:

```swift
// Tappable via action parameter
_KPIItem(data: .components([.metric("1.4"), .unitInFront("$"), .unit("M")]),
         subtitle: "Revenue",
         action: { showRevenueDetail() })
// foreground color becomes .tintColor when enabled, .tintColorTapState when pressed
```

---

## KPIs in a Header (horizontal, center-aligned)
```swift
// Center-aligned horizontal strip
HStack(spacing: 24) {
    _KPIItem(data: .components([.metric("1.4"), .unitInFront("$"), .unit("M")]),
             subtitle: "Revenue")
    _KPIItem(data: .components([.metric("342")]), subtitle: "Orders")
    _KPIItem(data: .percent(0.64), subtitle: "Win Rate")
}
.frame(maxWidth: .infinity)
```

## KPIs in a Modal (vertically stacked, max 2 per row, not tappable)
```swift
LazyVGrid(columns: [GridItem(.flexible()), GridItem(.flexible())], spacing: 16) {
    ForEach(kpis) { kpi in
        _KPIItem(data: .components([.metric(kpi.value)]), subtitle: kpi.label)
        // No action — not tappable in modal context
    }
}
```

## Custom `KPIProgressViewStyle`

The circular progress ring is fully replaceable via the `.kpiProgressViewStyle(_:)` environment modifier:

```swift
struct MyProgressStyle: KPIProgressViewStyle {
    func makeBody(configuration: Configuration) -> some View {
        ZStack {
            FioriCircularProgressView(
                fraction: configuration.fraction,
                circleColor: .preferredColor(.primaryFill),
                completedFractionColor: .preferredColor(.tintColor),
                circleStyle: StrokeStyle(lineWidth: 4, lineCap: .round),
                lineWidth: 4
            )
            VStack(spacing: 2) {
                configuration.kpi.frame(maxWidth: 108)
                configuration.subtitle.frame(maxWidth: 94)
            }
        }
    }
}

_KPIProgressItem(data: .percent(0.72), subtitle: "Completion")
    .kpiProgressViewStyle(MyProgressStyle())
```

`FioriCircularProgressViewStyle` (the default) accepts per-`ControlState` color dictionaries for `fractionColors` and `foregroundColors`.

---

## Figma Variants → SwiftUI

| Figma Property | Figma Value | SwiftUI |
|---------------|-------------|---------|
| Type | Standard | `_KPIItem(data:subtitle:)` |
| Type | Time | ViewBuilder `_KPIItem(kpi:subtitle:)` with `HStack` |
| Type | With icon | ViewBuilder `_KPIItem(kpi:subtitle:)` with `HStack` |
| Type | Progress (large) | `_KPIProgressItem(data:subtitle:footnote:)` |
| Type | Progress (small) | `_KPIProgressItem` with custom style |
| Unit label | Left | `.unitInFront` in `KPIItemData.components([...])` |
| Unit label | Right | `.unit` in `KPIItemData.components([...])` |
| Tappable | Yes | Pass `action:` to `_KPIItem` — value auto-renders in `.tintColor` |
| Tappable | No | Omit `action:` — value in `.primaryLabel` |
| Progress label | Inside | `configuration.kpi` inside `makeBody` of custom style |
| Progress label | Below | Use `footnote:` parameter |
| Placement | Header | Horizontal `HStack`, center-aligned |
| Placement | Modal | `LazyVGrid` 2-column, not tappable |

---

## Do's ✓ / Don'ts ✗

**Do:**
- Always include both a numeric value (A) and a label (C) — both are mandatory
- Show the value in tint color when the KPI is tappable
- Use unit labels to clarify ambiguous metrics ($ for currency, h for hours)
- Use the large progress view in headers, small in cards
- Center-align KPIs in header and content area arrangements

**Don't:**
- Make cell/modal KPIs tappable
- Use more than 2 unit labels on a single KPI
- Use more than 1 unit label when the KPI has an icon
- Use long, descriptive KPI labels — keep them concise

---

## Related Components

- [kpi-header.md](kpi-header.md) — KPI strip at the top of a screen
- [object-header.md](object-header.md) — KPI in right accessory of object header
- [cards.md](cards.md) — KPI in card body
