# Dimension Selector — Component Guidelines

> SAP Fiori for iOS | Category: Fiori SDK
> Figma: `https://www.figma.com/community/file/1450853598524410675/sap-fiori-for-ios-ui-kit`
> Development reference: UIKit `FUIDimensionSelector`, SwiftUI `_DimensionSelector`

## What is it?

A dimension selector is a horizontal bar of two or more mutually exclusive pill-shaped buttons. It allows users to switch between different measures, views, or chart ranges. Selection is mutually exclusive by default; tapping the active item can optionally deselect it.

---

## When to Use

**Do:**
- Use for switching between chart time ranges, view modes, or related data dimensions
- Place below a navigation bar, inside a chart, or in a modal/popover
- Ensure all dimensions are semantically related

**Don't:**
- Use when dimensions are unrelated
- Use to filter or narrow a list — use `FilterFeedbackBar` / `SortFilter` instead
- Use more than one dimension selector on the same screen

---

## Anatomy

```
[ Inactive ] [ Active ] [ Inactive ] [ Inactive ]
```

**A. Inactive segment** — rounded pill, border + fill from `.normal` `SegmentAttributes`
**B. Active segment** — rounded pill, tint-colored border + fill from `.selected` `SegmentAttributes`

---

## SDK Type Name

The SwiftUI type is `_DimensionSelector` (leading underscore). This is the current public name — the old name `SegmentedControl` is deprecated and unavailable:

```swift
@available(*, unavailable, renamed: "_DimensionSelector")
public typealias SegmentedControl = _DimensionSelector
```

Always use `_DimensionSelector`.

---

## Initializer

```swift
public init(
    segmentTitles: [String],
    interItemSpacing: CGFloat = 6,
    titleInsets: EdgeInsets = EdgeInsets(top: 8, leading: 8, bottom: 8, trailing: 8),
    selectedIndex: Int? = nil,
    contentInset: EdgeInsets? = nil
)
```

- `segmentTitles` — the button labels (not `titles:`)
- `interItemSpacing` — gap between pills, default 6 pt
- `titleInsets` — padding inside each pill, default 8 pt all sides
- `selectedIndex` — pre-selected segment (`nil` = nothing selected)
- `contentInset` — outer padding of the whole bar; if not set, defaults to 16 pt leading/trailing on compact, 48 pt on regular

> **Note:** `selectedIndex` is **not** a SwiftUI `@Binding`. It is an `Int?` value; observe changes via `selectionDidChangePublisher` (Combine).

---

## Key Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `titles` | `[String]` | — | Segment labels (settable after init) |
| `selectedIndex` | `Int?` | `nil` | Currently selected segment; set to `nil` to deselect, negative values cancel selection |
| `isEnable` | `Bool` | `true` | When set to `false`, clears current selection and disables interaction |
| `allowEmptySelection` | `Bool` | `true` | If `true`, tapping the active segment deselects it; if `false`, selection is always pinned |
| `segmentWidthMode` | `SegmentWidthMode` | `.intrinsic` | How segment widths are calculated |
| `segmentAttributes` | `[ControlState: SegmentAttributes]` | See defaults below | Per-state visual styling |
| `titleInsets` | `EdgeInsets` | 8 pt all sides | Padding inside each segment |
| `contentInset` | `EdgeInsets` | adaptive | Outer padding of the bar |
| `interItemSpacing` | `CGFloat` | `6` | Gap between segments |

---

## `SegmentWidthMode`

```swift
@frozen public enum SegmentWidthMode: Equatable {
    case fixed(CGFloat)  // all segments at an exact width
    case intrinsic       // each segment sized to its content (default); bar scrolls horizontally if needed
    case maximum         // all segments match the widest segment
    case equal           // all segments share equal width to fill available space
}
```

- `.intrinsic` and `.maximum` wrap in a horizontal `ScrollView`
- `.equal` stretches segments to fill the frame — no scroll view

---

## `SegmentAttributes` — Per-State Styling

```swift
public struct SegmentAttributes {
    public var textColor: Color?
    public var font: Font?
    public var borderWidth: CGFloat?
    public var borderColor: Color?
    public var backgroundColor: Color?
}
```

Applied per `ControlState`: `.normal`, `.selected`, `.disabled`.

**SDK defaults (set in initializer):**

| State | `textColor` | `font` | `borderWidth` | `borderColor` | `backgroundColor` |
|-------|------------|--------|--------------|--------------|-----------------|
| `.normal` | `.secondaryLabel` | `.subheadline` | `0.33` | `.separator` | `.tertiaryFill` |
| `.selected` | `.tintColor` | `.subheadline` | `1.0` | `.tintColor` | `.secondaryGroupedBackground` |
| `.disabled` | `.secondaryLabel` | `.subheadline` | `0.33` | `.secondaryFill` | `.tertiaryFill` |

Segments render as `RoundedRectangle(cornerRadius: 10)`.

---

## Observing Selection — Combine Publisher

`_DimensionSelector` does not use SwiftUI bindings for selection. Observe changes with the Combine publisher:

```swift
import Combine

var cancellables: Set<AnyCancellable> = []

let selector = _DimensionSelector(segmentTitles: ["Day", "Week", "Month"])

selector.selectionDidChangePublisher
    .sink { selectedIndex in
        // selectedIndex is Int?
    }
    .store(in: &cancellables)
```

`_heightDidChangePublisher: CurrentValueSubject<CGFloat, Never>` emits the selector's height after layout — useful for companion views (e.g. charts) that need to adjust their frame when the selector height changes due to Dynamic Type.

---

## Adaptive Design

| Size class | Default `contentInset` leading/trailing |
|-----------|----------------------------------------|
| Compact (iPhone) | 16 pt |
| Regular (iPad) | 48 pt |

Max recommended segment count: **5** on compact, **7** on regular.

For `.intrinsic` (default) mode the bar scrolls horizontally when segments overflow the available width. Use `.equal` to distribute widths evenly across the full frame.

---

## SwiftUI Code Examples

### Basic dimension selector

```swift
import FioriSwiftUICore
import Combine

@State private var selector = _DimensionSelector(
    segmentTitles: ["Day", "Week", "Month", "Quarter", "Year"],
    selectedIndex: 0
)
@State private var selectedRange: Int? = 0
var cancellables: Set<AnyCancellable> = []

var body: some View {
    VStack {
        selector

        ChartView(range: selectedRange)
    }
    .onAppear {
        selector.selectionDidChangePublisher
            .assign(to: \.selectedRange, on: self)
            .store(in: &cancellables)
    }
}
```

### Imperative selection (no Combine)

```swift
var selector = _DimensionSelector(
    segmentTitles: ["1W", "1M", "3M", "6M", "1Y"]
)

// Pre-select programmatically
selector.selectedIndex = 1

// Clear selection
selector.selectedIndex = nil
```

### Equal-width segments (fills container)

```swift
var selector = _DimensionSelector(segmentTitles: ["Map", "List", "Calendar"])
selector.segmentWidthMode = .equal

selector
    .frame(maxWidth: .infinity)
    .padding(.horizontal, 16)
```

### Fixed-width segments

```swift
var selector = _DimensionSelector(segmentTitles: ["1D", "1W", "1M", "1Y"])
selector.segmentWidthMode = .fixed(64)
```

### Maximum-width segments (all match widest)

```swift
var selector = _DimensionSelector(segmentTitles: ["Short", "A Much Longer Label", "Mid"])
selector.segmentWidthMode = .maximum
```

### Custom appearance per state

```swift
var selector = _DimensionSelector(segmentTitles: ["Revenue", "Cost", "Margin"])
selector.segmentAttributes = [
    .normal: SegmentAttributes(
        textColor: .preferredColor(.primaryLabel),
        font: .fiori(forTextStyle: .footnote),
        borderWidth: 0.5,
        borderColor: .preferredColor(.separator),
        backgroundColor: .preferredColor(.primaryBackground)
    ),
    .selected: SegmentAttributes(
        textColor: .preferredColor(.tintColor),
        font: .fiori(forTextStyle: .footnote),
        borderWidth: 1.5,
        borderColor: .preferredColor(.tintColor),
        backgroundColor: .preferredColor(.tintColor).opacity(0.1)
    ),
    .disabled: SegmentAttributes(
        textColor: .preferredColor(.quaternaryLabel),
        font: .fiori(forTextStyle: .footnote),
        borderWidth: 0.5,
        borderColor: .preferredColor(.secondaryFill),
        backgroundColor: .preferredColor(.tertiaryFill)
    )
]
```

### Prevent empty selection (always keep one selected)

```swift
var selector = _DimensionSelector(segmentTitles: ["Day", "Week", "Month"], selectedIndex: 0)
selector.allowEmptySelection = false
// Tapping the active segment now does nothing — selection stays
```

### Disabled state

```swift
var selector = _DimensionSelector(segmentTitles: ["A", "B", "C"])
selector.isEnable = false  // clears selection and shows all segments in .disabled style
```

### Placement in navigation bar

```swift
NavigationStack {
    ChartView()
        .navigationTitle("Revenue")
        .toolbar {
            ToolbarItem(placement: .principal) {
                selector
                    .frame(maxWidth: 280)
            }
        }
}
```

---

## Figma Variants → SwiftUI

| Figma Property | Figma Value | SwiftUI |
|---------------|-------------|---------|
| Items | 2–5 (compact) / 2–7 (regular) | `segmentTitles` array |
| Segment state | Active | `selectedIndex == itemIndex` |
| Segment state | Inactive | All other indices |
| Segment state | Disabled | `isEnable = false` |
| Width mode | Content-sized | `segmentWidthMode = .intrinsic` (default) |
| Width mode | Equal / fill | `segmentWidthMode = .equal` |
| Width mode | Fixed | `segmentWidthMode = .fixed(n)` |
| Width mode | Uniform (match widest) | `segmentWidthMode = .maximum` |
| Allow deselect | Yes | `allowEmptySelection = true` (default) |
| Allow deselect | No | `allowEmptySelection = false` |
| Placement | Below nav bar | `VStack` with content below |
| Placement | In nav bar | `ToolbarItem(placement: .principal)` |

---

## What's Not in the Provided Source

The following items are referenced in the source but their definitions were not included in the files provided — you may want to supply them for complete documentation:

- **`ControlState`** enum — used as key for `segmentAttributes`; values `.normal`, `.selected`, `.disabled` are inferred from usage but the full definition (including any additional cases) was not provided

---

## Do's ✓ / Don'ts ✗

**Do:**
- Use `_DimensionSelector` — `SegmentedControl` is removed
- Observe selection via `selectionDidChangePublisher` (Combine), not a SwiftUI binding
- Use `segmentWidthMode = .equal` when the selector should span the full width (view switchers)
- Use `segmentWidthMode = .intrinsic` (default) for chart time-range selectors with short labels
- Set `allowEmptySelection = false` when a selection must always be active
- Use `isEnable = false` to disable the entire control — it also clears the current selection

**Don't:**
- Pass a `Binding<Int>` to `selectedIndex` — the parameter is `Int?`, not a binding
- Use `DimensionSelector` (no underscore) or `SegmentedControl` — both are unavailable
- Use more than 5 segments on compact or 7 on regular
- Place two dimension selectors on the same screen

---

## Related Components

- [filter-feedback-bar.md](filter-feedback-bar.md) — list filtering (use instead of dimension selector for filtering)
- [segmented-control.md](segmented-control.md) — native SwiftUI `Picker(.segmented)` alternative
