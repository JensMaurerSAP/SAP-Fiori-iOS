# Signature Capture — Component Guidelines

> SAP Fiori for iOS | Category: Fiori SDK
> Figma: `https://www.figma.com/community/file/1450853598524410675/sap-fiori-for-ios-ui-kit`
> Development reference: UIKit `FUISignatureCaptureController`, `FUISignatureCaptureView`, SwiftUI `SignatureCaptureView`

## What is it?

Signature capture allows users to input their signature on screen to authorize workflows. The signature is captured as a `UIImage` with an optional watermark and/or timestamp embedded in the saved image.

**For forms requiring careful review of content during signing, use [Inline Signature Form Cell](inline-signature-form-cell.md) instead.**

---

## When to Use

- Signing forms or documents
- Approving orders
- Confirming tasks
- Indicating receipt of an article
- Signing off for another person

---

## Anatomy

```
A. Navigation Bar: [Cancel]   "Signature"   [Done]

B. Signature Canvas (rounded corners, background fill)
   _______________________________________________
  |                                               |
  |              ← freehand drawing →             |
  |_______________________________________________|

C. Toolbar: [Clear]

D. Watermark + timestamp (optional — burned into saved UIImage)
```

**A. Navigation Bar** — "Cancel" and "Done" buttons. Title is customizable.
**B. Signature Canvas** — freehand `DragGesture`-based drawing; bounded to canvas rect; strokes rendered as `Path` with `.round` line cap and join.
**C. Toolbar** — "Clear" button to erase all drawings.
**D. Watermark / timestamp** (optional) — text rendered into the `UIImage` at save time, positioned near the bottom of the canvas.

---

## SDK Architecture

The visible `SignatureCaptureView` is built on top of two internal components:

| Type | Role |
|------|------|
| `Drawing` | Stores a single stroke as `[CGPoint]` with `minX/maxX/minY/maxY` helpers |
| `_ScribbleView` | Internal SwiftUI drawing canvas; manages `drawings: [Drawing]`, stroke rendering, drag gesture, and `UIImage` export |
| `ScribbleView` | Newer variant of `_ScribbleView` with SwiftUI-based watermark rendering via `ImageRenderer` |

### `Drawing`

```swift
struct Drawing {
    var points: [CGPoint]
    var maxX: CGFloat  // max x across all points
    var maxY: CGFloat  // max y
    var minX: CGFloat  // min x
    var minY: CGFloat  // min y
}
```

Each `DragGesture.onEnded` appends the completed `Drawing` to `drawings` and resets `currentDrawing`.

### Image Export

On save, `_ScribbleView` / `ScribbleView`:
1. Converts `[Drawing]` to a `UIBezierPath` via `createUIBezierPath(drawings:lineWidth:)` — strokes use `.round` cap and join
2. If `cropsImage: true`, crops to the bounding box of the path + `signaturePadding` (16 pt all sides)
3. If a watermark is present, expands the image height by up to `watermarkTopToViewBottom` (48 pt) to accommodate it
4. Renders the image in `.light` trait collection — the saved signature is always light-mode regardless of device appearance
5. Returns a `UIImage` via `onSave`

---

## `_ScribbleView` Key Properties

> These are properties on the internal view. `SignatureCaptureView` exposes them as its own parameters.

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `strokeWidth` | `CGFloat` | `3.0` | Pen stroke width in points |
| `strokeColor` | `Color` | `.primaryLabel` | Stroke color; resolved to light mode in exported image |
| `drawingViewBackgroundColor` | `Color` | `.primaryBackground` | Canvas fill color |
| `cropsImage` | `Bool` | `false` | Crops exported image to signature bounding box + padding |
| `watermarkText` | `String?` | `nil` | Plain text watermark burned into the image |
| `watermarkTextColor` | `Color` | `.tertiaryLabel` | Watermark text color |
| `watermarkTextFont` | `UIFont` | `.caption1` | Watermark font |
| `watermarkTextAlignment` | `NSTextAlignment` | `.natural` | Watermark text alignment |
| `addsTimestampInImage` | `Bool` | `false` | Appends `"MM/dd/YYYY hh:mma zzz"` timestamp below watermark |
| `timestampFormatter` | `DateFormatter?` | `nil` | Custom timestamp format; if `nil`, uses default |

**Watermark layout:** rendered 48 pt from the canvas bottom (`watermarkTopToViewBottom`), with 24 pt horizontal padding (`watermarkHorizontalPadding`). If the watermark text is taller than the reserved space, the image height expands to fit with a minimum 16 pt bottom margin.

**Timestamp default format:** `"MM/dd/YYYY hh:mma zzz"` (e.g. `"07/21/2026 10:30AM PDT"`)

---

## Entry Flows

### From a Button Form Cell
Tapping a button in a form opens the signature capture modal. After signing, replace the trigger with an inline signature preview.

### As Part of a Navigation Flow
Signature capture can be integrated as a full step in a screen flow (e.g. approval workflow step 3 of 4).

---

## Adaptive Design

Supported in compact and regular width on iPhone and iPad. Can be presented as:
- Modal sheet
- Bottom sheet
- Form sheet
- Popover

---

## SwiftUI Code Examples

### Basic usage (bottom sheet)

```swift
import FioriSwiftUICore

@State private var showSignature = false
@State private var signatureImage: UIImage? = nil

FioriButton { _ in
    Text(signatureImage == nil ? "Add Signature" : "Re-sign")
}
.fioriButtonStyle(FioriSecondaryButtonStyle())
.onTapGesture { showSignature = true }
.sheet(isPresented: $showSignature) {
    SignatureCaptureView(
        onSave: { image in
            signatureImage = image
            showSignature = false
        },
        onDelete: { signatureImage = nil }
    )
    .presentationDetents([.medium, .large])
}
```

### With watermark and timestamp

```swift
SignatureCaptureView(
    onSave: { image in
        signatureImage = image
    },
    watermarkText: "Authorized by \(currentUser.name)",
    addsTimestampInImage: true,
    cropsImage: true
)
```

### Custom timestamp format

```swift
let formatter = DateFormatter()
formatter.dateFormat = "yyyy-MM-dd HH:mm"

SignatureCaptureView(
    onSave: { image in signatureImage = image },
    addsTimestampInImage: true,
    timestampFormatter: formatter,
    cropsImage: true
)
```

### Custom stroke appearance

```swift
SignatureCaptureView(
    onSave: { image in signatureImage = image },
    strokeWidth: 2.0,
    strokeColor: .preferredColor(.primaryLabel),
    drawingViewBackgroundColor: .preferredColor(.secondaryBackground),
    cropsImage: true
)
```

### As part of a navigation flow

```swift
NavigationStack {
    ApprovalStep1View()
        .navigationDestination(for: ApprovalStep.self) { step in
            switch step {
            case .signature:
                SignatureCaptureView(onSave: saveAndProceed)
            default:
                EmptyView()
            }
        }
}
```

### Displaying the saved signature

```swift
if let sig = signatureImage {
    VStack(alignment: .leading, spacing: 8) {
        Text("Signature")
            .font(.fiori(forTextStyle: .subheadline))
            .foregroundStyle(Color.preferredColor(.secondaryLabel))

        Image(uiImage: sig)
            .resizable()
            .scaledToFit()
            .frame(maxHeight: 80)
            .padding(8)
            .background(Color.preferredColor(.secondaryBackground))
            .clipShape(RoundedRectangle(cornerRadius: 8))

        FioriButton { _ in Text("Re-sign") }
            .fioriButtonStyle(FioriTertiaryButtonStyle())
            .onTapGesture { showSignature = true }
    }
}
```

---

## Figma Variants → SwiftUI

| Figma Property | Figma Value | SwiftUI |
|---------------|-------------|---------|
| Watermark | Yes | `watermarkText: "..."` |
| Watermark | No | Omit `watermarkText` |
| Timestamp | Yes | `addsTimestampInImage: true` |
| Timestamp format | Custom | `timestampFormatter: myFormatter` |
| Crop | Yes (recommended) | `cropsImage: true` |
| Stroke width | Thin / Standard | `strokeWidth: 1.5` / `strokeWidth: 3.0` |
| Presentation | Bottom sheet | `.sheet` + `.presentationDetents([.medium, .large])` |
| Presentation | Form sheet | `.sheet` (iPad default) |
| Presentation | Popover | `.popover` (iPad) |
| Entry | From button | `FioriButton` + `.sheet` |
| Entry | Navigation flow | `NavigationStack` destination |

---

## Do's ✓ / Don'ts ✗

**Do:**
- Use `cropsImage: true` to remove surrounding whitespace before storing — the default `false` includes the full canvas size
- Add a watermark with the signer's name or context when the signature has legal or audit significance
- Use `addsTimestampInImage: true` to embed a timestamp when the signing moment matters
- Replace the trigger button with an inline signature preview after saving
- Present in a sheet or as a navigation destination — not as an overlay on top of a form
- Use [Inline Signature Form Cell](inline-signature-form-cell.md) when the user needs to review form content while signing

**Don't:**
- Use signature capture as a security or authentication mechanism
- Store raw uncropped images when `cropsImage: false` — cropped images are smaller and display better
- Rely on the device's current color scheme for the saved image — it is always rendered in light mode regardless of dark mode setting

---

## What's Not in the Provided Source

The following are used by `_ScribbleView` / `ScribbleView` but their definitions were not in the provided files:

- **`SignatureCaptureView`** — the public-facing component; the exact initializer parameters (title, onDelete, etc.) are not in the provided source
- **`Watermark`** view — used by `ScribbleView` for SwiftUI-based watermark rendering
- **`watermarkStyle` environment key** — consumed by `ScribbleView` via `@Environment(\.watermarkStyle)`

If you have `SignatureCaptureView.swift`, sharing it would allow documenting the full public API.

---

## Related Components

- [inline-signature-form-cell.md](inline-signature-form-cell.md) — signature within a form (review content while signing)
- [button-form-cell.md](button-form-cell.md) — trigger button for signature capture
