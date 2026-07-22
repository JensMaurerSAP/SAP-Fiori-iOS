# Data Table — Component Guidelines

> SAP Fiori for iOS | Category: Fiori SDK
> Figma: `https://www.figma.com/community/file/1450853598524410675/sap-fiori-for-ios-ui-kit`
> Development reference: UIKit `FUIDataTable`, SwiftUI `DataTable`

## What is it?

A data table is a grid of labeled columns and rows used to present numbers, text, images, dates, times, durations, or list-picker values. It supports horizontal scrolling, sticky header row, optional sticky first column, pinch-to-zoom, row selection, and inline cell editing. On compact screens it can convert to a list report automatically.

---

## When to Use

**Do:**
- Use when users need to **compare multiple attributes** across items in a large data set
- Use with sticky headers and optional sticky first column for wide tables

**Don't:**
- Use in preview views (cards, object header, etc.)
- Use when users don't need to compare multiple attributes — use a list (`ObjectItem`) instead

---

## Anatomy

```
A. Header Row  (sticky — always visible during vertical scroll)
┌──────────────┬───────────────┬───────────────┬───────────────┐
│ B. Col 1 ↕  │    Column 2   │    Column 3   │    Column 4   │
├──────────────┼───────────────┼───────────────┼───────────────┤
│  [leading]  Row 1 data                              [trailing]│
├──────────────┼───────────────┼───────────────┼───────────────┤
│  [leading]  Row 2 data                              [trailing]│
└──────────────┴───────────────┴───────────────┴───────────────┘
 ↑ C. Persistent column (optional — sticks left during horizontal scroll)
```

### A. Header Row
Always at the top. Contains column labels. Sticky when `isHeaderSticky: true`.

### B. Rows of Data
Each row = one `TableRowItem`. Each column = one `DataItem` in that row's `data` array.

### C. Persistent Column (optional)
`isFirstColumnSticky: true` pins the leftmost column during horizontal scrolling.

---

## SDK Architecture

| Type | Role |
|------|------|
| `DataTable` | SwiftUI `View` that renders the table; takes a `TableModel` |
| `TableModel` | `ObservableObject` holding all data, layout, and interaction settings |
| `TableRowItem` | One row — holds an array of `DataItem`, leading/trailing accessories, and read-only state |
| `DataItem` (protocol) | Base protocol for all cell content types |
| `ColumnAttribute` | Per-column width, text alignment, read-only flag, and date/duration formatters |
| `DataTableChange` | Change record emitted by `model.valueDidChange` during inline editing |

---

## Cell Data Types (`DataItem` conformers)

| Type | `DataItemType` | Stored value | Displayed as |
|------|---------------|-------------|-------------|
| `DataTextItem` | `.text` | `String` | Text |
| `DataImageItem` | `.image` | `Image` | Icon (not editable inline) |
| `DataDateItem` | `.date` | `Date` | Formatted date string |
| `DataTimeItem` | `.time` | `Date` | Formatted time string |
| `DataDurationItem` | `.duration` | `TimeInterval` | `"%d Hrs %d Min"` (localizable) |
| `DataListItem` | `.listitem` | `String` | Text + chevron; opens a searchable list picker inline |

All text-based items share these optional properties:
- `font: Font?` / `uifont: UIFont?` — custom font (if both set, `uifont` wins)
- `textColor: Color?`
- `lineLimit: Int?`
- `isReadonly: Bool?`
- `binding: ObjectViewProperty.Text?` — maps cell to `ObjectItem` slot in list view

`DataImageItem` has `tintColor: Color?` and `binding: ObjectViewProperty.Image?`.

All item types have two initializer forms:
```swift
// Positional (SwiftUI Font)
DataTextItem("Invoice #001", .fiori(forTextStyle: .body), .preferredColor(.primaryLabel))

// Labeled (UIFont)
DataTextItem(text: "Invoice #001", uifont: UIFont.preferredFioriFont(forTextStyle: .body))
```

---

## `TableRowItem` — Row Structure

```swift
public struct TableRowItem {
    public let leadingAccessories: [AccessoryItem]  // icons or buttons on the left
    public let trailingAccessory: AccessoryItem?    // icon or button on the right
    public var data: [DataItem]                      // cell content
    public let selectedImage: Image?                 // custom checkmark in .select mode
    public let deSelectedImage: Image?               // custom circle in .select mode
    public var isReadonly: Bool?                     // row-level read-only override
}

public enum AccessoryItem {
    case button(AccessoryButton)  // tappable button with image/title + action
    case icon(Image)              // static decorative icon
}
```

Convenience initializer (no accessories):
```swift
TableRowItem(data: [DataTextItem("A"), DataTextItem("B")])
```

---

## `ColumnAttribute` — Per-Column Settings

```swift
public struct ColumnAttribute {
    public enum Width {
        case fixed(CGFloat)  // exact width
        case flexible        // auto-size to content, capped at 50% of container
        case infinity        // auto-size, then expand to fill remaining space
    }

    public var textAlignment: TextAlignment = .leading
    public var width: Width = .flexible
    public var isReadonly: Bool?           // column-level read-only
    public var dateFormatter: DateFormatter?
    public var durationTextFormat: String  // default: "%d Hrs %d Min"
}
```

By default the first column with no explicit `.infinity` setting is treated as `.infinity` — it expands to fill any remaining width after other columns are measured.

---

## `TableModel` — Configuration

```swift
public class TableModel: ObservableObject {
    // Data
    public var headerData: TableRowItem?
    public var rowData: [TableRowItem]
    public var columnAttributes: [ColumnAttribute]

    // Layout
    public var isHeaderSticky: Bool          // default: false
    public var isFirstColumnSticky: Bool     // default: false
    public var showListView: Bool            // default: false (show grid; true = list on compact)
    public var rowAlignment: RowAlignment    // .top (default) or .baseline
    public var minRowHeight: CGFloat         // default: 48
    public var minColumnWidth: CGFloat       // default: 48
    public var allowsPartialRowDisplay: Bool // default: true (set false for Table Card)

    // Appearance
    public var backgroundColor: Color       // default: .secondaryGroupedBackground
    public var showRowDivider: Bool         // default: true
    public var rowDividerHeight: CGFloat    // default: 1
    public var rowDividerColor: Color
    public var everyNumOfRowsToShowDivider: Int  // default: 1
    public var showColoumnDivider: Bool     // default: true (note: typo in SDK)
    public var columnDividerWidth: CGFloat  // default: 1
    public var columnDividerColor: Color
    public var headerCellPadding: EdgeInsets?
    public var dataCellPadding: EdgeInsets?
    public var isPinchZoomEnable: Bool      // default: false

    // Edit mode
    public var editMode: EditMode           // .none | .select | .inline
    public var selectedIndexes: [Int]       // read/write; header not counted

    // Callbacks
    public var didSelectRowAt: ((_ index: Int) -> Void)?
    public var valueDidChange: ((DataTableChange) -> Void)?
    public var validateDataItem: ((_ rowIndex: Int, _ columnIndex: Int, _ dataItem: DataItem) -> (Bool, String?))?
    public var listItemDataAndTitle: ((_ rowIndex: Int, _ columnIndex: Int) -> (listItems: [String], title: String))?
    public var cellTapped: ((Int, Int) -> Void)?
    public var keyboardDidShowOrHide: ((CGRect) -> Void)?
    public var didScroll: ((_ contentOffset: CGPoint, _ indexOfRows: [Int], _ indexOfColumns: [Int]) -> Void)?
}
```

### `TableModel.EditMode`

| Mode | Behaviour |
|------|----------|
| `.none` | Display only; tap calls `didSelectRowAt` |
| `.select` | Rows show leading checkboxes; `selectedIndexes` updates; tap toggles selection |
| `.inline` | Cells become editable; `DataTextItem` uses a `TextField`; `.date`/`.time`/`.duration` use a popover picker; `.listitem` opens a searchable sheet |

### Read-Only Priority (cell > row > column)

When `isReadonly` is set on multiple levels, the highest-priority non-nil value wins:
1. **Cell** (`DataItem.isReadonly`) — highest
2. **Row** (`TableRowItem.isReadonly`)
3. **Column** (`ColumnAttribute.isReadonly`) — lowest

`DataImageItem` ignores `isReadonly` — images are never inline-editable.

---

## `DataTable` View Modifiers

These are fluent modifiers that return `DataTable` and update the underlying model:

```swift
DataTable(model: model)
    .backgroundColor(.preferredColor(.primaryBackground))
    .headerSticky(true)
    .firstColumnSticky(true)
    .pinchZoomEnable(true)
    .showListView(false)
    .showColumnDivider(true)
    .showRowDivider(true)
    .dataCellPadding(EdgeInsets(top: 12, leading: 8, bottom: 12, trailing: 8))
    .headerCellPadding(EdgeInsets(top: 6, leading: 8, bottom: 6, trailing: 8))
    .minRowHeight(52)
    .minColumnWidth(60)
    .allowsPartialRowDisplay(false)
    .rowAlignment(.baseline)
```

`sizeThatFits(_ size: CGSize)` and `rectForCell(at:columnIndex:isRelativeToContentOffset:)` are available for layout queries.

> **Note:** `.editingMode(_:)` is deprecated — use `model.editMode` directly.

---

## Inline Editing — Save / Cancel

After the user finishes editing, call `model.onSave(_:)` to commit or discard:

```swift
let changes = model.onSave(true)   // save; invalid changes are rolled back; returns [DataTableChange]
model.onSave(false)                 // cancel; all changes discarded
```

`DataTableChange` contains: `rowIndex`, `columnIndex`, `value: ValueType` (`.text`, `.date`, `.duration`, `.image`), `text: String`, and `selectedIndex: Int?` (for `.listitem`).

---

## List View Mode

When `showListView = true` (or on compact when `showListView` is enabled), `DataTable` renders rows as `ObjectItem` views instead of a grid. Cell-to-slot mapping is determined by `DataItem.binding`:

| `ObjectViewProperty.Text` slot | `ObjectViewProperty.Image` slot |
|-------------------------------|-------------------------------|
| `.title` | `.detailImage` |
| `.subtitle` | `.statusImage` |
| `.footnote` | `.substatusImage` |
| `.status` | |
| `.substatus` | |

If no `binding` is set, the SDK assigns them automatically left-to-right by position.

---

## Inline Error Handling

- Validation runs via `model.validateDataItem` — return `(false, "error message")` to flag a cell invalid
- Invalid cells show a red border (inline text editing) or red underline (non-focused display)
- A `BannerView` appears automatically at the top of the table when validation errors exist — it dismisses automatically when all errors are resolved
- Tapping a read-only cell in `.inline` mode shows a `ToastMessage`: "Tapped cell is read-only."

---

## Adaptive Design

| Size class | Default behavior |
|-----------|-----------------|
| Regular (iPad) | Full data table with horizontal scroll |
| Compact (iPhone) | Controlled by `model.showListView`; set `true` to show list, `false` to keep grid |

---

## SwiftUI Code Examples

### Basic data table

```swift
import FioriSwiftUICore

let headerRow = TableRowItem(
    leadingAccessories: [],
    trailingAccessory: nil,
    data: [
        DataTextItem("Invoice #"),
        DataTextItem("Vendor"),
        DataTextItem("Amount", textAlignment: .trailing),
        DataTextItem("Due Date", textAlignment: .center)
    ]
)

let rows = invoices.map { invoice in
    TableRowItem(data: [
        DataTextItem(invoice.number),
        DataTextItem(invoice.vendor),
        DataTextItem(invoice.amount),
        DataDateItem(invoice.dueDate)
    ])
}

let model = TableModel(
    headerData: headerRow,
    rowData: rows,
    isHeaderSticky: true,
    isFirstColumnSticky: true
)

DataTable(model: model)
```

### Column attributes

```swift
model.columnAttributes = [
    ColumnAttribute(textAlignment: .leading, width: .infinity),   // first col expands
    ColumnAttribute(textAlignment: .leading, width: .flexible),
    ColumnAttribute(textAlignment: .trailing, width: .fixed(100)),
    ColumnAttribute(textAlignment: .center, width: .flexible)
]
```

### Row selection

```swift
model.editMode = .select
model.didSelectRowAt = { rowIndex in
    print("Tapped row \(rowIndex)")
}

// Or check selected indexes after multi-select
model.selectedIndexes  // [Int], header not counted
```

### Inline editing with validation

```swift
model.editMode = .inline

model.validateDataItem = { rowIndex, columnIndex, dataItem in
    if columnIndex == 2, let item = dataItem as? DataTextItem {
        let isNumeric = Double(item.text) != nil
        return (isNumeric, isNumeric ? nil : "Amount must be a number")
    }
    return (true, nil)
}

model.valueDidChange = { change in
    print("Row \(change.rowIndex), col \(change.columnIndex): \(change.text)")
}
```

### List picker column

```swift
// Column with DataListItem opens a searchable sheet in inline edit mode
let row = TableRowItem(data: [
    DataTextItem("Invoice #001"),
    DataListItem("Draft")   // tapping shows a picker sheet
])

model.listItemDataAndTitle = { rowIndex, columnIndex in
    return (["Draft", "Pending", "Approved", "Rejected"], "Status")
}
```

### Duration column

```swift
// DataDurationItem stores TimeInterval (seconds)
let row = TableRowItem(data: [
    DataTextItem("Task A"),
    DataDurationItem(3600 * 2 + 60 * 30)  // 2 hours 30 minutes → "2 Hrs 30 Min"
])

// Custom format per column
var durationColumn = ColumnAttribute()
durationColumn.durationTextFormat = "%dh %dm"
model.columnAttributes = [ColumnAttribute(), durationColumn]
```

### Leading and trailing accessories

```swift
let row = TableRowItem(
    leadingAccessories: [
        .button(AccessoryButton(image: Image(systemName: "pencil"), action: { edit(row) })),
        .icon(Image(systemName: "exclamationmark.triangle"))
    ],
    trailingAccessory: .button(AccessoryButton(image: Image(systemName: "trash"), action: { delete(row) })),
    data: [DataTextItem("Row content")]
)
```

### Save / cancel inline edits

```swift
// In nav bar toolbar
Button("Done") {
    let changes = model.onSave(true)
    model.editMode = .none
    applyChanges(changes)
}
Button("Cancel") {
    model.onSave(false)
    model.editMode = .none
}
```

### Read-only cells

```swift
// Column-level: entire column is read-only
var column = ColumnAttribute()
column.isReadonly = true

// Row-level: entire row is read-only
let row = TableRowItem(data: [...], isReadonly: true)

// Cell-level (highest priority)
let cell = DataTextItem("Fixed value", isReadonly: true)
```

### Scroll and size utilities

```swift
// Preferred size for all rows/columns given a container width
let fitSize = DataTable(model: model).sizeThatFits(CGSize(width: 375, height: 0))

// Rect of a specific cell (useful for scrolling to it)
let cellRect = DataTable(model: model).rectForCell(at: 2, columnIndex: 1)
```

---

## Figma Variants → SwiftUI

| Figma Property | Figma Value | SwiftUI |
|---------------|-------------|---------|
| Sticky header | Yes | `isHeaderSticky: true` |
| Sticky column | Yes | `isFirstColumnSticky: true` |
| Edit mode | None | `model.editMode = .none` |
| Edit mode | Selection | `model.editMode = .select` |
| Edit mode | Inline | `model.editMode = .inline` |
| Column alignment | Leading | `ColumnAttribute(textAlignment: .leading)` |
| Column alignment | Trailing | `ColumnAttribute(textAlignment: .trailing)` |
| Column alignment | Center | `ColumnAttribute(textAlignment: .center)` |
| Column width | Fixed | `ColumnAttribute(width: .fixed(120))` |
| Column width | Flexible | `ColumnAttribute(width: .flexible)` |
| Column width | Fill remaining | `ColumnAttribute(width: .infinity)` |
| Row alignment | Top | `rowAlignment: .top` (default) |
| Row alignment | Baseline | `rowAlignment: .baseline` |
| Compact behavior | List | `model.showListView = true` |
| Compact behavior | Grid | `model.showListView = false` |
| Pinch to zoom | Enabled | `.pinchZoomEnable(true)` |
| Read-only cell | Yes | `isReadonly: true` on cell/row/column |
| Validation error | Shown | Return `(false, "message")` from `validateDataItem` |

---

## Do's ✓ / Don'ts ✗

**Do:**
- Set `isFirstColumnSticky: true` for wide tables — keeps row identity visible during horizontal scroll
- Right-align numeric and monetary columns (`textAlignment: .trailing`)
- Use `model.editMode = .select` for bulk-action workflows; read `model.selectedIndexes` for the selection
- Provide `model.validateDataItem` for all inline-editable text columns
- Call `model.onSave(_:)` explicitly when the user taps Done or Cancel in inline edit mode
- Set `ColumnAttribute.isReadonly = true` instead of making entire rows read-only when only some columns should be locked
- Set `allowsPartialRowDisplay = false` when embedding in a Table Card

**Don't:**
- Use `DataTableItem` — it is internal; use `DataTextItem`, `DataDateItem`, etc.
- Use the deprecated `isEditing` property — use `model.editMode` instead
- Use `.onSelectionChange`, `.allowsColumnSorting`, `.allowsHorizontalScrolling` — these modifiers do not exist
- Pass `isEditing: $isEditing` to `DataTable(model:)` — the initializer only takes `model:`
- Assume `didSelectRowAt` fires in `.inline` mode — it only fires in `.none` mode
- Rely on automatic binding assignment in list view for complex layouts — set `binding` explicitly

---

## Related Components

- [object-cell.md](object-cell.md) — list-based alternative; used automatically in `showListView` mode
- [filter-feedback-bar.md](filter-feedback-bar.md) — filtering data table results
- [banner.md](banner.md) — appears automatically inside the table on validation errors
- [toast-message.md](toast-message.md) — read-only cell tap feedback
