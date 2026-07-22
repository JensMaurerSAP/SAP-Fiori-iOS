# Calendar — Component Guidelines

> SAP Fiori for iOS | Category: Fiori SDK
> Figma: `https://www.figma.com/community/file/1450853598524410675/sap-fiori-for-ios-ui-kit`
> Development reference: UIKit `FUICalendarView`, SwiftUI `CalendarView`

## What is it?

The calendar view provides a visual overview of a week, a month, or multiple months. It can also display event indicators on dates, support date range or multi-date selection, and display alternate calendar systems alongside the primary calendar.

---

## When to Use

**Do:**
- Display a visual overview of a week, a month, multiple months, or a range of selected dates
- Visually present correlations between two dates — consecutive holidays, peak vs. off-season pricing, hotel availability
- Combine with object cells (agenda) or timeline (timesheet) in the space below the calendar

**Don't:**
- Use calendar view when a text-based or list-based picker can fulfill the primary requirement — use `DateTimePicker` or `DateRangePicker` instead

---

## Anatomy

The calendar is composed of four parts:

```
┌─────────────────────────────────────┐
│  D. Month Label (nav bar / title)   │
│─────────────────────────────────────│
│  B. Week Label Container            │
│  Su  Mo  Tu  We  Th  Fr  Sa        │
│─────────────────────────────────────│
│                                     │
│  C. Date Cell Container             │
│   1   2   3   4   5   6   7        │
│   8   9  10  11  12  13  14        │
│  ...                                │
│                                     │
└─────────────────────────────────────┘  ← A. Calendar View Container

[ Space for agenda / timeline below ]
```

### A. Calendar View Container
Outer container. The space below it is available for different configurations — e.g. `ObjectItem` rows as an agenda, or `TimelineItem` rows as a timesheet.

### B. Week Label Container
Row of day-of-week abbreviations. Start day is configurable via `CalendarModel.firstWeekday`. Today's weekday column is highlighted in tint color.

### C. Date Cell Container
Grid of date cells. Appearance is driven by `CalendarDayState` (see States section).

### D. Month Label
Displayed as navigation bar title. Format is configurable via the `monthHeaderDateFormat` environment value (default `"MMMM"`).

---

## SDK Architecture

The calendar is driven by a single `@Observable` model class:

| Type | Role |
|------|------|
| `CalendarModel` | Central `@Observable` data model — holds style, selection, disabled dates, scroll position, and title |
| `CalendarStyle` | Enum controlling the display and selection mode |
| `CalendarDayState` | Enum describing every visual state a day cell can be in |
| `CalendarDisabledDates` | Value type describing which dates are non-selectable |
| `CalendarWeekInfo` | Model for one week row — week number + 7 dates |
| `CalendarMonthModel` | Model for one month — year, month, array of `CalendarWeekInfo` |

---

## `CalendarStyle` — Display and Selection Modes

```swift
public enum CalendarStyle {
    case month           // one month, vertical scroll, single select
    case fullScreenMonth // full-screen month, vertical scroll, single select
    case week            // one week strip, horizontal scroll, single select
    case expandable      // month ↔ week toggle via drag handle, single select
    case rangeSelection  // full-screen month, vertical scroll, contiguous date range
    case datesSelection  // full-screen month, vertical scroll, non-contiguous dates
}
```

- `.month` and `.fullScreenMonth` always render 6-week rows per month for consistent height
- `.rangeSelection` and `.datesSelection` force `isPersistentSelection = true` regardless of the parameter passed
- `.fullScreenMonth`, `.rangeSelection`, and `.datesSelection` hide out-of-month dates

---

## `CalendarModel` — Initializer and Key Properties

```swift
@Observable
public class CalendarModel {
    public var calendarStyle: CalendarStyle
    public var selectedDate: Date?          // single select
    public var selectedDates: Set<Date>?    // datesSelection style
    public var selectedRange: ClosedRange<Date>?  // rangeSelection style
    public var disabledDates: CalendarDisabledDates?
    public var firstWeekday: Int            // 1=Sun … 7=Sat, default = system
    public var scrollToDate: Date           // programmatic scroll target
    let startDate: Date                     // calendar range start
    let endDate: Date                       // calendar range end
    let isPersistentSelection: Bool
    var title: String?                      // auto-updated month/range title
}
```

Default date range when neither `startDate` nor `endDate` is provided: current year's Jan 1 through next year's Dec 31.

If `startDate > endDate`, the SDK falls back to the same default range.

---

## `CalendarDayState` — Day Cell Visual States

All 12 states that control a day cell's appearance:

| State | When applied |
|-------|-------------|
| `.normal` | Default |
| `.today` | Today's date, not selected |
| `.outOfMonth` | Date belongs to a different month (only shown in `.month` / `.expandable` styles) |
| `.singleSelected` | Selected in single-select mode |
| `.singleSelectedAndToday` | Today and selected |
| `.multiSelectedStart` | First day of a selected range |
| `.multiSelectedMiddle` | Interior day of a selected range |
| `.multiSelectedEnd` | Last day of a selected range |
| `.disabled` | Disabled, not today |
| `.disabledAndToday` | Disabled and today |
| `.disabledInMultiSelection` | Disabled date inside a selected range |
| `.disabledAndTodayInMultiSelection` | Disabled today inside a selected range |

**Computed state helpers:**

| Property | True for |
|----------|---------|
| `isSelected` | singleSelected, singleSelectedAndToday, multiSelectedStart/Middle/End |
| `isDisabled` | outOfMonth, disabled, disabledAndToday, disabledInMultiSelection, disabledAndTodayInMultiSelection |
| `isTitleBold` | today, any selected state, disabledAndToday, disabledAndTodayInMultiSelection |
| `isSingleSelected` | singleSelected, singleSelectedAndToday |
| `isMultiSelected` | multiSelectedStart/Middle/End, disabledInMultiSelection, disabledAndTodayInMultiSelection |

---

## `CalendarDisabledDates` — Disabling Dates

```swift
public struct CalendarDisabledDates {
    public var beforeDate: Date?   // all dates ≤ this date are disabled
    public var afterDate: Date?    // all dates ≥ this date are disabled
    public var weekdays: [Int]     // 1=Sun … 7=Sat; all matching weekdays disabled
    public var others: [Date]      // individual arbitrary dates
}
```

`isDisabled(_:)` checks all four rules in order — any match disables the date. Date comparisons use `.day` granularity via `Calendar.autoupdatingCurrent`.

---

## Alternate Calendars

Secondary calendar display is controlled via the `alternateCalendarType` environment value:

```swift
public enum AlternateCalendarType: CaseIterable {
    case none     // no secondary calendar (default)
    case chinese
    case hebrew
    case islamic
}
```

A separate `alternateCalendarLocale: Locale?` environment value overrides the default locale for the secondary calendar.

---

## Environment Customization

All visual and behavioral customization is applied through SwiftUI environment values:

| Environment Key | Type | Default | Effect |
|----------------|------|---------|--------|
| `showsWeekNumbers` | `Bool` | `false` | Show ISO week number column |
| `hasEventIndicator` | `Bool` | `true` | Show event dot below date number |
| `customLanguageId` | `String?` | `nil` | Override locale for day/month labels |
| `customBundle` | `Bundle?` | `nil` | Override bundle for localized strings |
| `alternateCalendarType` | `AlternateCalendarType` | `.none` | Secondary calendar system |
| `alternateCalendarLocale` | `Locale?` | `nil` | Locale for alternate calendar |
| `selectionSingleColor` | `Color` | `.tintColor` | Highlight color for single selection |
| `selectionRangeColor` | `Color` | `.tintColor` | Highlight color for range selection |
| `eventViewColor` | `Color` | `.tertiaryLabel` 60% | Event dot color (enabled dates) |
| `eventViewColorDisabled` | `Color` | `.quaternaryLabel` 60% | Event dot color (disabled dates) |
| `monthHeaderDateFormat` | `String` | `"MMMM"` | Date format template for month header |
| `calendarItemTintAttributes` | `[CalendarPropertyRef: [CalendarItemControlState: Color]]` | `[:]` | Per-property, per-state foreground color overrides |
| `bannerMessageBackgroundColor` | `Color?` | `nil` | Background color for banner messages |

### `calendarItemTintAttributes` detail

`CalendarPropertyRef` targets:
- `.title` — day number text
- `.monthHeaderText` — month header text (`.normal` state only)
- `.weekDayText` — weekday abbreviations (`.normal` and `.highlighted` states only)
- `.weekNumberText` — week number text (`.normal` state only)

`CalendarItemControlState` values: `.normal`, `.selected`, `.disabled`, `.highlighted`

---

## Variations

### Calendar Styles

| Style | Scroll | Selection | Out-of-month dates | Full screen |
|-------|--------|-----------|-------------------|-------------|
| `.month` | Vertical | Single | Shown | No |
| `.fullScreenMonth` | Vertical | Single | Hidden | Yes |
| `.week` | Horizontal | Single | N/A | No |
| `.expandable` | Horizontal / toggle | Single | Shown (month mode) | No |
| `.rangeSelection` | Vertical | Contiguous range | Hidden | Yes |
| `.datesSelection` | Vertical | Non-contiguous set | Hidden | Yes |

### First Weekday

`firstWeekday` on `CalendarModel` (1 = Sunday … 7 = Saturday, default = system setting):

| Region | Value |
|--------|-------|
| US | 1 (Sunday) |
| Europe, Asia, Oceania | 2 (Monday) |
| Middle East | 7 (Saturday) |

---

## SwiftUI Code Examples

### Basic month calendar

```swift
import FioriSwiftUICore

@State private var model = CalendarModel()

var body: some View {
    CalendarView(model: model)
}
```

### Single-date selection with agenda below

```swift
@State private var model = CalendarModel(
    calendarStyle: .month,
    selectedDate: .now
)

var body: some View {
    VStack(spacing: 0) {
        CalendarView(model: model)

        Divider()

        List(invoices(for: model.selectedDate ?? .now)) { invoice in
            ObjectItem {
                Text(invoice.number)
            } subtitle: {
                Text(invoice.vendor)
            }
        }
    }
}
```

### Week strip with timesheet below

```swift
@State private var model = CalendarModel(calendarStyle: .week)

var body: some View {
    VStack(spacing: 0) {
        CalendarView(model: model)

        Divider()

        ScrollView {
            ForEach(entries(for: model.selectedDate ?? .now)) { entry in
                TimelineItem { Text(entry.project) }
            }
        }
    }
}
```

### Expandable (month ↔ week toggle)

```swift
// Starts in week mode, user can drag to expand to month
@State private var model = CalendarModel(
    calendarStyle: .expandable,
    expandableStyleInWeekMode: true
)

CalendarView(model: model)
```

### Range selection

```swift
@State private var model = CalendarModel(calendarStyle: .rangeSelection)

var body: some View {
    CalendarView(model: model)
    // model.selectedRange: ClosedRange<Date>? updates automatically
    // model.title shows "Jan 5, 2026 - Jan 12, 2026"
}
```

### Non-contiguous multi-date selection

```swift
@State private var model = CalendarModel(
    calendarStyle: .datesSelection,
    selectedDates: Set<Date>()
)

CalendarView(model: model)
// model.title shows "3 Dates Selected"
```

### Disabled dates

```swift
@State private var model = CalendarModel(
    disabledDates: CalendarDisabledDates(
        beforeDate: Calendar.current.date(byAdding: .day, value: -1, to: .now),
        weekdays: [1, 7]  // disable Sundays and Saturdays
    )
)
```

### Custom date range and start day

```swift
@State private var model = CalendarModel(
    startDate: someStartDate,
    endDate: someEndDate,
    firstWeekday: 2  // Monday
)
```

### Programmatic scroll

```swift
// Scroll to a specific date without changing selection
model.scrollToDate = targetDate
```

### Environment customization

```swift
CalendarView(model: model)
    .environment(\.showsWeekNumbers, true)
    .environment(\.hasEventIndicator, true)
    .environment(\.alternateCalendarType, .chinese)
    .environment(\.selectionSingleColor, Color.preferredColor(.tintColor))
    .environment(\.monthHeaderDateFormat, "MMMM yyyy")
```

### Custom day/weekday colors

```swift
CalendarView(model: model)
    .environment(\.calendarItemTintAttributes, [
        .title: [
            .normal: .preferredColor(.primaryLabel),
            .selected: .preferredColor(.primaryBackground),
            .disabled: .preferredColor(.quaternaryLabel)
        ],
        .weekDayText: [
            .normal: .preferredColor(.tertiaryLabel),
            .highlighted: .preferredColor(.tintColor)
        ]
    ])
```

### Monday-first calendar (European locale)

```swift
@State private var model = CalendarModel(firstWeekday: 2)

CalendarView(model: model)
```

---

## Figma Variants → SwiftUI

| Figma Property | Figma Value | SwiftUI |
|---------------|-------------|---------|
| Style | Month | `CalendarModel(calendarStyle: .month)` |
| Style | Full-screen month | `CalendarModel(calendarStyle: .fullScreenMonth)` |
| Style | Week | `CalendarModel(calendarStyle: .week)` |
| Style | Expandable | `CalendarModel(calendarStyle: .expandable)` |
| Style | Range selection | `CalendarModel(calendarStyle: .rangeSelection)` |
| Style | Dates selection | `CalendarModel(calendarStyle: .datesSelection)` |
| Start day | Sunday | `firstWeekday: 1` |
| Start day | Monday | `firstWeekday: 2` |
| Start day | Saturday | `firstWeekday: 7` |
| Event indicator | Shown | `.environment(\.hasEventIndicator, true)` (default) |
| Event indicator | Hidden | `.environment(\.hasEventIndicator, false)` |
| Week numbers | Shown | `.environment(\.showsWeekNumbers, true)` |
| Alternate calendar | Chinese | `.environment(\.alternateCalendarType, .chinese)` |
| Alternate calendar | Hebrew | `.environment(\.alternateCalendarType, .hebrew)` |
| Alternate calendar | Islamic | `.environment(\.alternateCalendarType, .islamic)` |
| Persistent selection | On | `isPersistentSelection: true` |
| Persistent selection | Off | `isPersistentSelection: false` (default) |

---

## Do's ✓ / Don'ts ✗

**Do:**
- Drive all calendar state through `CalendarModel` — it is the single source of truth
- Set `firstWeekday` from `Calendar.current.firstWeekday` unless the user has a locale preference to override
- Use `scrollToDate` for programmatic navigation without changing selection
- Use `disabledDates.beforeDate` / `afterDate` to restrict selectable range to valid booking windows
- Use `.environment(\.calendarItemTintAttributes, ...)` for per-state color customization instead of trying to style individual cells
- Use the space below the calendar for contextual content (agenda, timesheet, event list)

**Don't:**
- Use `CalendarMonthView` or `CalendarWeekView` as separate types — they do not exist; use `CalendarModel(calendarStyle:)`
- Pass `isPersistentSelection: false` with `.rangeSelection` or `.datesSelection` — the SDK ignores it and forces `true`
- Rely on `model.title` for display in custom navigation bars without observing `model` — it updates asynchronously after scroll/selection changes
- Mix out-of-month date visibility — it is automatically controlled by `calendarStyle` and cannot be overridden per-cell

---

## Related Components

- [forms.md](forms.md) — `DateTimePicker`, `DateRangePicker` for text-based date input
- [timeline-view.md](timeline-view.md) — timesheet view below calendar
- [object-cell.md](object-cell.md) — agenda rows below calendar
