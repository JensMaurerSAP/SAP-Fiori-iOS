# Timeline View — Component Guidelines

> SAP Fiori for iOS | Category: Fiori SDK — Timeline
> Figma: `https://www.figma.com/community/file/1450853598524410675/sap-fiori-for-ios-ui-kit`
> Development reference: UIKit `FUITimelineCell`, `FUITimelineMarkerCell`, SwiftUI `TimelineItem`, `TimelineMarker`

## What is it?

The timeline view displays a list of items (tasks, events, meetings) in chronological order with tappable cells. Items can be filtered and searched.

---

## When to Use

**Do:**
- Show a brief, concise overview of objects
- Organize objects by due date or time
- Indicate status of objects through node color and shape

**Don't:**
- Show all object details in the cell — tap to drill down for details

---

## Anatomy

### Full Timeline View
```
A. Header Cell: "My Tasks"
──────────────────────────────────────
 12:00  ◆ B. Timeline Marker (start)
        │
 14:30  ● C. Past object (filled blue)
        │
[Today] ● D. Current object (blue cell bg)
        │
 16:00  ○ Future object (grey unfilled)
        │
 18:00  ◆ B. Timeline Marker (end)
```

**A. Header Cell** — categorical overview.
**B. Timeline Marker Cell** — diamond-shaped node; marks start/end or visual breaks.
**C. Now Indicator** (optional) — indicates today's cell(s).
**D. Timeline Cells** — concise key info; tappable to drill down.

### Timeline Cell Anatomy
```
A. Timestamp  B. Node/Line  C. Title (3 lines max)
              │             D. Status Stack
              │             E. Description (2 lines max)
              │             F. Attributes (1 line max)
```

**A. Timestamp Column** — flexible format; optional non-interactive icon below.
**B. Node / Node Line** — color and shape communicate state.
**C. Title** — wraps to max 3 lines before truncating.
**D. Status Stack** — status of the object.
**E. Description** — wraps to max 2 lines before truncating.
**F. Attributes** — cannot exceed 1 line.

---

## Node States

| State | `TimelineNodeType` | Icon rendered |
|-------|-------------------|--------------|
| Before start | `.beforeStart` | `fiori.tag` |
| Start / milestone | `.start` | `fiori.rhombus.milestone.2` (diamond) |
| Open | `.open` | `fiori.circle.task` |
| In progress | `.inProgress` | `ellipsis.circle.fill` (SF Symbols) |
| Complete | `.complete` | `fiori.sys.enter.2` (checkmark) |
| Before end | `.beforeEnd` | `fiori.tag` |
| End / milestone | `.end` | `fiori.rhombus.milestone.2` (diamond) |
| Add | `.add` | `fiori.add` |
| Delete | `.delete` | `fiori.delete.fill` |

Use `TimelineNodeView(_ nodeType: TimelineNodeType)` to render the correct node icon:

```swift
TimelineNodeView(.inProgress)
TimelineNodeView(.complete)
TimelineNodeView(.start)
```

---

## Navigation Patterns

| App type | How timeline is reached |
|----------|------------------------|
| Flat navigation | Own tab in tab bar |
| Hierarchical navigation | Via timeline preview cell |
| Landing screen | Direct — no preview needed |

---

## SwiftUI Code Examples

### Basic timeline
```swift
import FioriSwiftUICore

VStack(spacing: 0) {
    ForEach(Array(events.enumerated()), id: \.element.id) { index, event in
        TimelineItem {
            Text(event.title)
                .font(.fiori(forTextStyle: .headline))
        } subtitle: {
            Text(event.subtitle)
                .foregroundStyle(Color.preferredColor(.secondaryLabel))
        } attribute: {
            Text(event.timestamp)
                .foregroundStyle(Color.preferredColor(.tertiaryLabel))
        } status: {
            Text(event.status)
                .foregroundStyle(event.statusColor)
        } timelineNode: {
            TimelineNodeView(event.nodeType)  // TimelineNodeType drives the icon
        }
        .onTapGesture { navigateToDetail(event) }
    }
}
```

### With start/end markers
```swift
VStack(spacing: 0) {
    // Start milestone
    TimelineItem {
        Text("Project Kickoff")
    } attribute: {
        Text("Jan 15, 09:00")
    } timelineNode: {
        TimelineNodeView(.start)  // diamond icon
    }

    // Regular items
    ForEach(milestones) { milestone in
        TimelineItem {
            Text(milestone.title)
        } attribute: {
            Text(milestone.date)
        } timelineNode: {
            TimelineNodeView(milestone.nodeType)
        }
    }

    // End milestone
    TimelineItem {
        Text("Project Completion")
    } attribute: {
        Text("Mar 31, 17:00")
    } timelineNode: {
        TimelineNodeView(.end)  // diamond icon
    }
}
```

### Current item (today indicator)
```swift
TimelineItem {
    Text("Sprint Review")
        .font(.fiori(forTextStyle: .headline))
} attribute: {
    VStack(alignment: .leading) {
        Text("Today")
            .foregroundStyle(Color.preferredColor(.tintColor))
            .font(.fiori(forTextStyle: .footnote, weight: .bold))
        Text("14:00 – 15:00")
            .foregroundStyle(Color.preferredColor(.tertiaryLabel))
    }
} timelineNode: {
    TimelineNodeView(.inProgress)
}
.background(Color.preferredColor(.tintColor).opacity(0.08))
```

### Combined with CalendarView
```swift
VStack(spacing: 0) {
    CalendarView(model: CalendarModel(calendarStyle: .week))
    Divider()
    ScrollView {
        VStack(spacing: 0) {
            ForEach(events(for: selectedDate)) { event in
                TimelineItem {
                    Text(event.title)
                } attribute: {
                    Text(event.time)
                } timelineNode: {
                    TimelineNodeView(event.nodeType)
                }
            }
        }
        .padding(.horizontal, 16)
    }
}
```

---

## Figma Variants → SwiftUI

| Figma Property | Figma Value | SwiftUI |
|---------------|-------------|---------|
| Node | Past / complete | `TimelineNodeView(.complete)` |
| Node | In progress | `TimelineNodeView(.inProgress)` |
| Node | Future / open | `TimelineNodeView(.open)` |
| Node | Start milestone | `TimelineNodeView(.start)` — diamond |
| Node | End milestone | `TimelineNodeView(.end)` — diamond |
| Node icon | Add | `TimelineNodeView(.add)` |
| Node icon | Delete | `TimelineNodeView(.delete)` |
| Last item | Yes | Last item in the list — remove connecting line via your container logic |
| Title wrapping | 3 lines max | `.lineLimit(3)` |
| Description | 2 lines max | `.lineLimit(2)` |
| Attributes | 1 line | `.lineLimit(1)` |

---

## Do's ✓ / Don'ts ✗

**Do:**
- Use `TimelineNodeView(_ nodeType: TimelineNodeType)` to render the correct Fiori node icon — do not use `TimelineMarker` with `state:` / `nodeShape:` parameters, which are not in the provided source
- Show concise cell content — detail belongs on the drill-down page
- Use `.start`/`.end` for milestone nodes (renders diamond icon)
- Combine with `CalendarView(model:)` for schedule views

**Don't:**
- Show all object details in the timeline cell

---

## Related Components

- [timeline-preview.md](timeline-preview.md) — horizontal preview that navigates to this view
- [calendar.md](calendar.md) — calendar + timeline combination
