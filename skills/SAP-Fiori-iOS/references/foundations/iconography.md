# Iconography — Design Foundations

> SAP Fiori for iOS | Foundation
> Resources: SAP Fiori Icon Library, SF Symbols (Apple HIG)

## What is it?

Iconography is essential to a cohesive app experience. Icons communicate actions, information, feedback, and navigation. SAP icons have been redesigned for the Horizon visual theme — fresh, friendly, bold, with consistency of size, stroke, and visual balance for enterprise use.

---

## When to Use

**Do:**
- Use the SAP icon library for SAP Fiori apps
- Use icons to communicate important actions efficiently
- Use icons to indicate navigation to other screens

**Don't:**
- Use third-party icons
- Place icons too close together
- Use half-pixel increments for icon placement
- Use light-colored icons against light backgrounds

---

## SAP Design Principles for Icons

| Principle | Meaning |
|-----------|---------|
| Simple | Clear metaphors, minimal detail |
| Fresh | Modern, approachable aesthetic |
| Neutral | Enterprise-appropriate, not playful |
| Modern | Aligned with current design language |

---

## Icon Sources

### SAP Fiori Custom Symbols (SDK)

The SAP Fiori Icon Library is available as a **custom symbol library** in `FioriThemeManager`. Supports multiple weights and scales via `.font(.fiori(...))`.

```swift
import FioriThemeManager

// Access via namespace enum
FioriIcon.actions.add
FioriIcon.status.messageWarning
FioriIcon.travel.suitcaseFill

// Use directly as SwiftUI Image — it IS an Image
FioriIcon.objects.bell
    .font(.fiori(forTextStyle: .title3))
    .foregroundStyle(Color.preferredColor(.tintColor))

// Pass as Image parameter
let icon: Image = FioriIcon.time.timesheet
```

### SF Symbols (Apple)

Use SF Symbols when no SAP-specific icon is needed or for tab bar items (where only `systemImage:` string names are supported by SwiftUI's `.tabItem {}`).

```swift
// SF Symbol
Image(systemName: "plus.circle.fill")
    .foregroundStyle(Color.preferredColor(.tintColor))
    .font(.fiori(forTextStyle: .title2))

// Accessibility label required for icon-only interactive elements
Image(systemName: "trash")
    .accessibilityLabel("Delete")
```

> **Tab bar limitation:** SwiftUI's `Label("Title", systemImage:)` in `.tabItem {}` only accepts SF Symbol names as strings. Use `Label { Text(...) } icon: { FioriIcon... }` to use Fiori icons in tab bars.

---

## Usage Hierarchy

**Use SAP Fiori custom symbols when:**
- The icon represents an SAP-specific business action or object
- The designer has specified a Fiori icon from the icon library
- The icon is a standard SAP brand element

**Use SF Symbols when:**
- A matching system symbol exists for a common iOS action (add, delete, share, etc.)
- The icon is used in `.tabItem {}` with `systemImage:` string API

---

## Sizing — Icon to Text Style Mapping

Always size icons using `.font(.fiori(forTextStyle:))` — never `.font(.system(size:))`.

| Context | Text style | Approx pt |
|---------|-----------|-----------|
| Navigation bar icon | `.fiori(forTextStyle: .title3)` | 20pt |
| Toolbar / tab bar icon | `.fiori(forTextStyle: .title2)` | 22pt |
| List row accent / inline body | `.fiori(forTextStyle: .headline)` | 17pt |
| Inline body text | `.fiori(forTextStyle: .body)` | 17pt |
| Overflow / menu button | `.fiori(forTextStyle: .body)` | 17pt |
| Small / caption inline | `.fiori(forTextStyle: .footnote)` | 13pt |
| Large / hero | `.fiori(fixedSize: 48)` | 48pt fixed |

```swift
// Navigation bar icon
FioriIcon.actions.action
    .font(.fiori(forTextStyle: .title3))
    .foregroundStyle(Color.preferredColor(.tintColor))

// List row icon
FioriIcon.time.timesheet
    .font(.fiori(forTextStyle: .headline))
    .foregroundStyle(Color.preferredColor(.accentLabel6))

// Toolbar / add button
FioriIcon.actions.add
    .font(.fiori(forTextStyle: .title2))
    .foregroundStyle(Color.preferredColor(.tintColor))

// Overflow menu
FioriIcon.actions.action
    .font(.fiori(forTextStyle: .body))
    .foregroundStyle(Color.preferredColor(.secondaryLabel))
```

---

## Color

Always use Fiori color tokens for icon colors — never hardcode:

```swift
// Interactive icon
FioriIcon.actions.share
    .foregroundStyle(Color.preferredColor(.tintColor))

// Positive status icon
FioriIcon.status.statusPositive
    .foregroundStyle(Color.preferredColor(.positiveLabel))

// Warning status icon
FioriIcon.status.messageWarning
    .foregroundStyle(Color.preferredColor(.criticalLabel))

// Error status icon
FioriIcon.status.messageError
    .foregroundStyle(Color.preferredColor(.negativeLabel))

// Secondary / decorative icon
FioriIcon.status.information
    .foregroundStyle(Color.preferredColor(.secondaryLabel))

// Disabled icon
FioriIcon.actions.edit
    .foregroundStyle(Color.preferredColor(.quaternaryLabel))
```

---

## FioriIcon Namespace Catalog

1,900+ icons organized in namespaces. Use the namespace that best matches the semantic category.

### `FioriIcon.actions` — User actions and commands
Common: `add`, `edit`, `delete`, `share`, `filter`, `sort`, `search`, `settings`, `camera`, `download`, `upload`, `refresh`, `save`, `cancel`, `overflow`, `action`, `favorite`, `unfavorite`, `bookmark`, `show`, `hide`, `expand`, `collapse`, `menu`, `menuFill`, `notification2`, `uiNotifications`, `receipt`, `feedback`, `print`, `copy`, `paste`, `undo`, `redo`, `zoomIn`, `zoomOut`, `fullScreen`, `exitFullScreen`, `locateMe`, `multiSelect`, `sysAdd`, `sysCancel`, `sysEnter`, `sysFind`, `validate`, `accept`, `decline`, `forward`, `back`

### `FioriIcon.status` — Status and state indicators
Common: `alert`, `warning`, `warning2`, `error`, `information`, `messageWarning`, `messageError`, `messageSuccess`, `messageInformation`, `statusPositive`, `statusNegative`, `statusCritical`, `statusCompleted`, `statusInProcess`, `statusInactive`, `inProgress`, `pending`, `locked`, `unlocked`, `verified`, `flag`, `highPriority`, `notificationFill`, `notification`, `connected`, `disconnected`, `busy`, `away`

### `FioriIcon.objects` — Generic business objects
Common: `bell`, `bell2`, `businessObjectsMobile`, `businessObjectsMobileFill`, `camera`, `creditCard`, `email`, `globe`, `key`, `laptop`, `lightbulb`, `microphone`, `phone`, `suitcase`, `suitcaseFill`, `tag`, `tags`, `wallet`, `world`, `document`, `grid`, `map`, `mapFill`, `shield`, `video`, `checklist`, `product`, `productFill`

### `FioriIcon.time` — Time and scheduling
Common: `timesheet`, `timeAccount`, `timeEntryRequest`, `timeOvertime`, `createEntryTime`, `history`, `future`, `past`, `present`, `inProgress`, `pending`, `fobWatch`, `fobWatchFill`, `lateness`, `instance`

### `FioriIcon.travel` — Travel and hospitality
Common: `flight`, `suitcase`, `suitcaseFill`, `home`, `homeFill`, `travelExpense`, `travelItinerary`, `carRental`, `carRentalFill`, `taxi`, `busPublicTransport`, `subwayTrain`, `passengerTrain`, `meal`, `bed`, `world`, `doctor`

### `FioriIcon.transport` — Logistics and transport
Common: `flight`, `carRental`, `carRentalFill`, `busPublicTransport`, `subwayTrain`, `passengerTrain`, `cargoTrain`, `taxi`, `shippingStatus`, `travelRequest`, `travelExpense`, `travelExpenseReport`, `travelExpenseReportFill`, `tripReport`, `inventory`

### `FioriIcon.money` — Finance and expenses
Common: `expenseReport`, `receipt`, `travelExpense`, `travelExpenseReport`, `travelExpenseReportFill`, `currency`, `wallet`, `salesOrder`, `salesDocument`, `salesQuote`, `paymentApproval`, `moneyBills`, `loan`, `perDiem`, `collectionsManagement`, `batchPayments`

### `FioriIcon.shopping` — Commerce and procurement
Common: `cart`, `cartFull`, `cartApproval`, `cart2`–`cart5`, `basket`, `basketFill`, `retailStore`, `retailStoreManager`

### `FioriIcon.people` — People and HR
Common: `employee`, `manager`, `customer`, `customerFill`, `group`, `account`, `addEmployee`, `addContact`, `businessCard`, `businessCardFill`, `hrApproval`, `employeeApprovals`, `collaborate`, `collaborateFill`, `userSettings`, `userEdit`, `personPlaceholder`, `personPlaceholderFill`

### `FioriIcon.form` — Forms and approvals
Common: `approvals`, `form`, `receipt`, `survey`, `createForm`, `showEdit`, `detailView`, `customerOrderEntry`

### `FioriIcon.documents` — Documents and files
Common: `document`, `documents`, `addDocument`, `expenseReport`, `timesheet`, `salesOrder`, `salesDocument`, `request`, `writeNewDocument`, `pdfAttachment`, `excelAttachment`, `docAttachment`, `barCode`, `inspection`, `measurementDocument`

### `FioriIcon.places` — Locations and geography
Common: `home`, `homeFill`, `building`, `factory`, `map`, `mapFill`, `globe`, `world`, `locateMe`, `locateMe2`, `offSiteWork`, `retailStore`, `pool`, `theater`, `meal`

### `FioriIcon.calendars` — Calendar and scheduling
Common: `calendar`, `calendarFill`, `addCalendar`, `notification2`, `salesNotification`, `travelRequest`

### `FioriIcon.message` — Communication
Common: `email`, `call`, `notification2`, `uiNotifications`, `salesNotification`, `away`, `busy`, `appearOffline`

### `FioriIcon.charts` — Data visualization
Common: `areaChart`, `barChart`, `lineChart`, `bubbleChart`, `pieChart`, `columnChart`, `donutChart`, `horizontalBarChart`, `scatterChart`, `waterfallChart`, `kpiManagingMyArea`

### `FioriIcon.devices` — Hardware and devices
Common: `iphone`, `ipad`, `laptop`, `desktopMobile`, `keyboard`

---

## Tab Bar with Fiori Icons

SwiftUI's `.tabItem {}` accepts `Image` view builders — use Fiori icons directly:

```swift
TabView(selection: $selectedTab) {
    StartView()
        .tabItem {
            Label {
                Text("Start")
            } icon: {
                selectedTab == .start
                    ? FioriIcon.travel.homeFill   // active = filled variant
                    : FioriIcon.travel.home       // inactive = unfilled
            }
        }
        .tag(Tab.start)

    AppListView()
        .tabItem {
            Label {
                Text("Apps")
            } icon: {
                FioriIcon.objects.businessObjectsMobile
            }
        }
        .tag(Tab.apps)

    ApprovalsView()
        .tabItem {
            Label {
                Text("Approvals")
            } icon: {
                FioriIcon.form.approvals
            }
        }
        .tag(Tab.approvals)
}
.tint(Color.preferredColor(.tintColor))
```

---

## Do's ✓ / Don'ts ✗

**Do:**
- Add `.accessibilityLabel()` to every icon-only interactive element
- Use `.accessibilityHidden(true)` for purely decorative icons
- Match icon color to context: `.tintColor` for actions, `.secondaryLabel` for decorative
- Use consistent icon weights within the same component
- Size icons with `.font(.fiori(forTextStyle:))` — never `.font(.system(size:))`
- Use filled variants (e.g. `homeFill`, `suitcaseFill`) for active/selected states

**Don't:**
- Use third-party icon sets
- Use light icons on light backgrounds
- Hardcode icon sizes with `.font(.system(size:))`
- Use `Image(systemName:)` for SAP-specific business icons

---

## Resources

- [SAP Fiori Icon Library (SAPUI5 Icon Explorer)](https://sapui5.hana.ondemand.com/sdk/#/topic/21ea0ea94614480d9a910b2e93431291)
- [SF Symbols — Apple HIG](https://developer.apple.com/design/human-interface-guidelines/sf-symbols)
