# Object Header — Component Guidelines

> SAP Fiori for iOS | Category: Fiori SDK — Headers
> Figma: `https://www.figma.com/community/file/1450853598524410675/sap-fiori-for-ios-ui-kit`
> Development reference: UIKit `FUIObjectHeader`, SwiftUI `_ObjectHeader`

## What is it?

The object header provides a quick view of the most important or most frequently used information about one instance of an object. It connects visually to the navigation bar via a seamless background and is visually separated from the content area below. Information should be clear and concise — a high-level summary, not a data dump.

---

## When to Use

**Do:**
- Use to provide a summary of an object detail screen

**Don't:**
- Overload with too much information — keep it to the most important details

---

## Anatomy

Only the **title** is required. All other elements are optional.

```
┌────────────────────────────────────────────────┐
│ [A. Thumbnail]  B. Title                       │
│                 B. Subtitle          D. Right  │
│                 C. Tags              Accessory │
│                                                │
│ E. Label items (location / date / time)        │
│ F. Description (optional)                      │
│                                                │
│ ← swipe →  G. Page control (if chart present) │
│ H. Header Chart (on second page)               │
└────────────────────────────────────────────────┘
```

### A. Thumbnail
Product image, avatar, logo, or icon. Helps the user visually identify the object.

### B. Main Content
Title (required) and subtitle.

### C. Tags
Complementary information displayed as tags — visually distinct from plain text, functioning as independent bits of information.

### D. Right Accessory
Object status or an important KPI.

### E. Label Items
Important quick-read details such as location, date, and time. Each label item can be text only or combined with an image.

### F. Description
Additional details — only use when it provides genuinely valuable information.

### G. Page Control
Appears when a header chart is placed on a second page. User swipes to navigate between the main header and the chart page.

### H. Header Chart
A chart (line or column) for quickly understanding relevant trend or status information. For full chart header guidance, see [chart-header.md](chart-header.md).

---

## Layout

### Compact layout
All elements stack vertically. On compact, `_ObjectHeader` uses a `TabView` (page style) internally to support up to 3 pages:
- **Page 1** — thumbnail + title + subtitle + tags + status + additional info (bodyText, footnote) or description
- **Page 2** — `detailContent` (e.g. `HeaderChart`)
- **Page 3** — description (only when additional info is also present)

A `PageIndicator` appears automatically below the tab view when more than 1 page is present.

### Regular (iPad)
More horizontal space. Layout is three-column: left (thumbnail + title + subtitle + additional info), middle (detailContent or description), right (status + substatus). Middle column is capped at 312 pt; right at 120 pt; left fills the rest (min 120 pt, max 740 pt).

The `title` field uses `.title3` on iOS 14+ and `.headline` on earlier versions.

---

## SwiftUI Code Examples

### Minimal (title only)
```swift
import FioriSwiftUICore

_ObjectHeader(title: "Invoice #1234")
```

### Standard object header (ViewBuilder form)
```swift
_ObjectHeader(title: {
    Text("Sales Order #5678")
}, subtitle: {
    Text("Acme Corporation")
        .foregroundStyle(Color.preferredColor(.secondaryLabel))
}, tags: {
    Tag("Open")
    Tag("High Priority")
}, detailImage: {
    Image(systemName: "cart.fill")
        .font(.fiori(forTextStyle: .largeTitle))
        .foregroundStyle(Color.preferredColor(.tintColor))
}, detailContent: {
    VStack(alignment: .trailing, spacing: 2) {
        Text("$14,500")
            .font(.fiori(forTextStyle: .KPI, weight: .light))
            .foregroundStyle(Color.preferredColor(.primaryLabel))
        Text("Total")
            .font(.fiori(forTextStyle: .footnote))
            .foregroundStyle(Color.preferredColor(.secondaryLabel))
    }
})
```

### With thumbnail (product image / avatar)
```swift
_ObjectHeader(title: {
    Text("Product XR-400")
}, subtitle: {
    Text("Electronics · In stock")
        .foregroundStyle(Color.preferredColor(.secondaryLabel))
}, detailImage: {
    AsyncImage(url: product.imageURL) { image in
        image.resizable()
            .aspectRatio(contentMode: .fill)
            .cornerRadius(6)
            .clipped()
    } placeholder: {
        RoundedRectangle(cornerRadius: 6)
            .fill(Color.preferredColor(.secondaryBackground))
    }
    .frame(width: 45, height: 45)  // compact: 45×45; regular: 70×70
})
```

### With bodyText, footnote, and descriptionText
```swift
// bodyText and footnote form the "additional info" area (tags, bodyText, footnote)
// descriptionText appears on its own page on compact when additionalInfo is also present
_ObjectHeader(title: {
    Text("Service Ticket #99")
}, subtitle: {
    Text("Plumbing · Building A")
}, tags: {
    Tag("Urgent")
}, bodyText: {
    HStack(spacing: 6) {
        Image(systemName: "location")
            .foregroundStyle(Color.preferredColor(.secondaryLabel))
        Text("Floor 3, Room 301")
            .font(.fiori(forTextStyle: .footnote))
            .foregroundStyle(Color.preferredColor(.secondaryLabel))
    }
}, footnote: {
    HStack(spacing: 6) {
        Image(systemName: "calendar")
            .foregroundStyle(Color.preferredColor(.secondaryLabel))
        Text("Due: 25 Jul 2026")
            .font(.fiori(forTextStyle: .footnote))
            .foregroundStyle(Color.preferredColor(.secondaryLabel))
    }
})
```

### With descriptionText
```swift
_ObjectHeader(title: {
    Text("Project Alpha")
}, subtitle: {
    Text("SAP Internal · R&D")
}, descriptionText: {
    Text("Cross-functional initiative to modernize the invoicing workflow across EMEA region.")
        .font(.fiori(forTextStyle: .subheadline))
        .foregroundStyle(Color.preferredColor(.primaryLabel))
})
```

### With header chart (two-page header on compact)
```swift
// On compact, page 2 shows the HeaderChart; PageIndicator appears automatically
_ObjectHeader(title: {
    Text("Revenue Dashboard")
}, subtitle: {
    Text("YTD 2026")
}, detailContent: {
    // detailContent slot receives the HeaderChart
    _HeaderChart(title: {
        Text("Revenue")
            .foregroundStyle(Color.preferredColor(.tintColor))  // tint = interactive
    }, trend: {
        HStack(spacing: 4) {
            Image(systemName: "arrow.up.right")
                .foregroundStyle(Color.preferredColor(.positiveLabel))
            Text("+12%")
                .foregroundStyle(Color.preferredColor(.positiveLabel))
        }
        .font(.fiori(forTextStyle: .subheadline))
    }, chart: {
        ChartView(ChartModel(
            chartType: .line,
            data: [[2.1, 2.4, 1.9, 2.8, 2.6, 3.1]],
            titlesForCategory: [["Jan", nil, nil, nil, nil, "Jun"]],
            colorsForCategory: [0: [0: Color.preferredColor(.chart1)]]
        ))
    })
    .onTapGesture { navigateToFullChart() }
})
```

### Inside a detail screen (standard pattern)
```swift
NavigationStack {
    ScrollView {
        VStack(spacing: 0) {
            _ObjectHeader(
                title: { Text(invoice.number) },
                subtitle: { Text(invoice.vendor)
                    .foregroundStyle(Color.preferredColor(.secondaryLabel)) },
                tags: { Tag(invoice.status) },
                status: {
                    Text(invoice.formattedAmount)
                        .font(.fiori(forTextStyle: .title3))
                        .foregroundStyle(Color.preferredColor(.primaryLabel))
                }
            )

            Divider()
            InvoiceDetailBody(invoice: invoice)
        }
    }
    .navigationTitle(invoice.number)
    .navigationBarTitleDisplayMode(.inline)
}
```

---

## Figma Variants → SwiftUI

| Figma Property | Figma Value | SwiftUI |
|---------------|-------------|---------|
| Thumbnail | Image | `detailImage: { AsyncImage(...).cornerRadius(6) }` |
| Thumbnail | Avatar | `detailImage: { Circle()... }` |
| Thumbnail | Icon | `detailImage: { Image(systemName:) }` |
| Thumbnail | None | Omit `detailImage` |
| Tags | Yes | `tags: { Tag("...") Tag("...") }` or `tags: [String]` |
| Right accessory | Status text | `status: { Text("...").foregroundStyle(...) }` |
| Right accessory | KPI | `detailContent: { Text(...).font(.fiori(forTextStyle: .KPI)) }` |
| Body text | Text only | `bodyText: { Text(...) }` |
| Body text | Text + icon | `bodyText: { HStack { Image + Text } }` |
| Footnote | Yes | `footnote: { Text(...) }` |
| Description | Yes | `descriptionText: { Text(...) }` |
| Header chart | Yes | `detailContent: { _HeaderChart {...} }` — page 2 auto |
| Header chart | None | Omit `detailContent` or use non-chart content |
| Size class | Compact | Tabbed pages, `PageIndicator` auto (internal) |
| Size class | Regular | Three-column layout (internal) |

---

## Do's ✓ / Don'ts ✗

**Do:**
- Keep content clear and concise — high-level summary only
- Use the title as the object's primary identifier (number, name, ID)
- Use tags for categorical labels, not status — express status in the right accessory
- Only include a description when it provides genuinely valuable additional context
- Use tint color for the chart title when the header chart is interactive
- Let the page control appear automatically — don't suppress it when a header chart is present

**Don't:**
- Overload with too many elements — every slot doesn't need to be filled
- Duplicate information already visible in the navigation bar title
- Use the header chart for multi-series or complex data — keep it to a single trend
- Place interactive actions in the object header — use the toolbar instead

---

## Related Components

- [chart-header.md](chart-header.md) — header chart component (H slot)
- [kpi-header.md](kpi-header.md) — alternative KPI-focused header
- [object-item.md](object-item.md) — list row version of object display
