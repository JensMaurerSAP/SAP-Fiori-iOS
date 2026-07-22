# Attachment Form Cell — Component Guidelines

> SAP Fiori for iOS | Category: Fiori SDK — Input and Selection
> Figma: `https://www.figma.com/community/file/1450853598524410675/sap-fiori-for-ios-ui-kit`
> Development reference: UIKit `FUIAttachmentsFormCell`, SwiftUI `AttachmentGroup` / `Attachment`

## What is it?

An attachment control allows users to upload files — images, audio, video, text, PDFs, CSVs, and presentations — to a form. The "Add (+)" button enables users to upload from the photo library, files app, camera, or document scanner.

---

## When to Use

**Do:**
- Use as the **last piece of information** in a create screen or edit mode workflow
- Place at the **bottom** of a create modal, object page, or object detail page

**Don't:**
- Use for uploading profile images — use an avatar picker instead

---

## Anatomy

```
A. Label: "Attachments (3)"     [B. Add (+) button]   ← hidden in read-only
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│  C. Preview  │ │  C. Preview  │ │  C. Preview  │  → overflow to next row
│  D. Name     │ │  D. Name     │ │  D. Name     │
│  D. Size     │ │  D. Size     │ │  D. Date     │
└──────────────┘ └──────────────┘ └──────────────┘
```

### A. Label
Shows the count of attachments in the container.

### B. Add (+) Button
- **Active state** — visible; allows adding attachments
- **Read-only / disabled state** — hidden

### C. Attachment Item
Preview of the image, video, or document. If no preview is available, shows a doctype icon determined by file extension. Tapping opens a full-screen `QLPreviewController` previewer with swipe-through support.

### D. Attachment Text Content
- Line 1 — **mandatory**: attachment item name
- Line 2 — optional: file size, date uploaded, or other metadata
- Line 3 — optional

---

## States

| State | Add (+) | View | Delete |
|-------|---------|------|--------|
| Active (`.normal`) | Visible | ✓ | ✓ (in previewer) |
| Read-only (`.readOnly`) | Hidden | ✓ | ✗ (delete icon disabled) |
| Disabled | Hidden | ✗ | ✗ |

---

## SDK Architecture

The attachment system is built around three collaborating types:

| Type | Role |
|------|------|
| `AttachmentContext` | `@Observable` coordinator shared via SwiftUI environment; holds picker state, delegates upload/delete to `AttachmentDelegate` |
| `AttachmentGroupConfiguration` | Value type holding `attachments: [AttachmentInfo]`, `maxCount`, and `errorMessage: AttributedString?` |
| `AttachmentDelegate` | Protocol the app implements to perform the actual upload and delete operations |

### `AttachmentInfo` — the attachment state enum

```swift
public enum AttachmentInfo: Identifiable, Hashable {
    case uploading(sourceURL: URL)
    case uploaded(destinationURL: URL, sourceURL: URL, extraInfo: (any AttachmentExtraInfo)? = nil)
    case error(sourceURL: URL, message: String)
}
```

Key computed properties:
- `primaryURL` — destination URL for `.uploaded`, source URL otherwise
- `attachmentName` — last path component of `primaryURL`
- `errorMessage` — non-nil only in `.error` state

Array extensions on `[AttachmentInfo]`:
- `.isUploading` — true if any item is `.uploading`
- `.hasError` — true if any item is `.error`
- `.hasAttachmentsMoreThan(_ count:)` — count check on uploaded items only

### `AttachmentDelegate` protocol

```swift
public protocol AttachmentDelegate: AnyObject {
    func upload(contentFrom provider: NSItemProvider,
                onStarting: ((URL) -> Void)?,
                onCompletion: @escaping (URL?, Error?) -> Void)

    func delete(url: URL, onCompletion: @escaping (URL, Error?) -> Void)
}
```

Errors are typed as `AttachmentError`:
```swift
public enum AttachmentError: Error {
    case failedToUploadAttachment(String)
    case failedToDeleteAttachment(String)
}
```

### `BasicAttachmentDelegate` — local-storage convenience

`BasicAttachmentDelegate` is a ready-made delegate that stores attachments in a temporary local folder. Use it for prototyping or apps that manage files locally.

```swift
let delegate = BasicAttachmentDelegate(localFolderName: "MyAppAttachments")
// localFolder URL is available for reading files back
```

Subclass and override `saveLocally(url:identifier:onCompletion:)` to redirect to a remote server.

---

## Source-Selection Menu Items

The add (+) button uses composable menu items that communicate through `AttachmentContext` in the SwiftUI environment:

| View | Source |
|------|--------|
| `PhotosPickerMenuItem` | System Photos picker |
| `FilesPickerMenuItem` | System Files / iCloud picker |
| `CameraMenuItem` | Device camera (photo + video) |
| `PdfScannerMenuItem` | Document scanner → PDF |

All accept optional `title: String?` and `icon: String?` (SF Symbol name) parameters.

---

## Operation Presentation Modifiers

Attach menu items to the add (+) button using one of three presentation styles:

| Modifier | Presentation |
|----------|-------------|
| `.operationsMenu { }` | Native `Menu` (long-press or tap popover) |
| `.operationsDialog { }` | `confirmationDialog` action sheet |
| `.operationsOverlay { }` | Inline overlay above the button |

All three wrap `.defaultOperations()` internally, which wires up the `fileImporter`, `PhotosPicker`, camera, and scanner sheets.

---

## Behavior & Interaction

### Adding Attachments
1. Tap "Add (+)" → menu / dialog with source options
2. Select source → system picker or camera launches
3. After selection, `AttachmentContext` calls `AttachmentDelegate.upload(contentFrom:onStarting:onCompletion:)`
4. Item appears as `.uploading` immediately; transitions to `.uploaded` or `.error` on completion

The "Add (+)" button **stays in place** as attachments are added. New attachments appear to the right. When the row is full, attachments overflow to the next row.

### Viewing Attachments
Tap an attachment item → `AttachmentPreview` (`QLPreviewController`) opens. Users can swipe through all attachment items. Delete button is shown in the previewer when `controlState == .normal`.

### Deleting Attachments
- `.uploaded` items: `AttachmentContext.delete(attachment:)` calls `AttachmentDelegate.delete(url:onCompletion:)` and removes on success
- `.uploading` and `.error` items: removed immediately without delegate involvement
- Previewer shows an `UIAlertController` confirmation before deleting

### Error Handling
If `AttachmentDelegate` calls back with an `AttachmentError`, the item transitions to `.error` state (red icon). `AttachmentGroupConfiguration.errorMessage: AttributedString?` carries a semantic error message displayed below the grid.

### `AttachmentDefaultGuesturesModifier`
Apply standard tap-to-preview, context-menu, and accessibility actions to any attachment view:

```swift
myAttachmentView
    .attachmentDefaultGuestures(configuration: attachmentConfiguration)
```

Behaviour varies by `controlState`:
- `.normal` — preview + delete actions
- `.readOnly` — preview only
- other — no actions

---

## Thumbnail Generation

Default thumbnails are chosen by file extension (via `AttachmentThumbnailStyle.getDefaultThumbnail(url:)`):

| Extension group | Icon |
|----------------|------|
| Audio (mp3, wav, aac…) | `FioriIcon.documents.attachmentAudio` |
| Video (mp4, mov, avi…) | `FioriIcon.documents.attachmentVideo` |
| Images (jpg, png, heif…) | `FioriIcon.documents.attachmentPhoto` |
| PDF | `FioriIcon.documents.pdfAttachment` |
| Word (doc, docx) | `FioriIcon.documents.docAttachment2` |
| Excel / CSV | `FioriIcon.documents.excelAttachment` |
| PowerPoint / Keynote | `FioriIcon.documents.pptAttachment` |
| Zip / rar | `FioriIcon.documents.attachmentZipFile` |
| HTML / XML | `FioriIcon.documents.attachmentHtml` |
| Text / RTF | `FioriIcon.documents.attachmentTextFile` |
| Fallback | `FioriIcon.documents.document` |

For actual file previews, apply `AttachmentThumbnailWithPreviewStyle`, which generates a `QLThumbnailGenerator` image asynchronously and falls back to the icon while loading.

```swift
AttachmentThumbnail(configuration)
    .attachmentThumbnailStyle(AttachmentThumbnailWithPreviewStyle())
```

---

## Layout Constants (`AttachmentConstants`)

Public constants available for custom layouts to stay consistent with SDK defaults:

| Constant | Value |
|----------|-------|
| `thumbnailWidth` / `thumbnailHeight` | 109 pt |
| `thumbnailCornerRadius` | 16 pt |
| `iconWidth` / `iconHeight` | 24 pt |
| `horizontalPadding` | 16 pt |
| `verticalPadding` | 10 pt |
| `cellVerticalSpacing` | 10 pt |
| `thumbnailBorderLineWidth` | 1.5 pt |

---

## Adaptive Design

| Context | Layout |
|---------|--------|
| iPhone (compact) | Single-column grid |
| iPad modal compact | Compact grid in modal |
| iPad regular-readable | Wider grid |

---

## SwiftUI Code Examples

### Minimal setup with `BasicAttachmentDelegate`

```swift
import FioriSwiftUICore

@State private var context = AttachmentContext()
@State private var config = AttachmentGroupConfiguration(attachments: [])

var body: some View {
    let delegate = BasicAttachmentDelegate(localFolderName: "MyAttachments")
    context.delegate = delegate
    context.configuration = config

    return AttachmentGroup(context: context)
        .environment(context)
}
```

### Add button with menu presentation

```swift
Image(systemName: "plus")
    .operationsMenu {
        PhotosPickerMenuItem()
        FilesPickerMenuItem()
        CameraMenuItem()
        PdfScannerMenuItem()
    }
    .environment(context)
```

### Add button with dialog (action sheet) presentation

```swift
Image(systemName: "plus")
    .operationsDialog {
        PhotosPickerMenuItem()
        FilesPickerMenuItem()
        CameraMenuItem()
    }
    .environment(context)
```

### Read-only display

```swift
// Pass .readOnly controlState — Add (+) hidden, delete disabled in previewer
AttachmentGroup(context: context, controlState: .readOnly)
    .environment(context)
```

### Custom upload delegate

```swift
class MyUploadDelegate: BasicAttachmentDelegate {
    override func saveLocally(url: URL, identifier: String,
                              onCompletion: @escaping (URL?, Error?) -> Void) {
        // Upload to server, call onCompletion with remote URL on success
        MyAPIClient.upload(fileAt: url) { remoteURL, error in
            onCompletion(remoteURL, error)
        }
    }
}
```

### Reacting to upload state

```swift
// AttachmentContext exposes isUploading; configuration.attachments is observable
if context.isUploading {
    ProgressView("Uploading…")
}

// Check array-level state
if config.attachments.hasError {
    Text("One or more attachments failed to upload.")
        .foregroundStyle(Color.preferredColor(.negativeLabel))
}
```

### Attachment previewer with delete

```swift
@State private var previewURL: URL = someURL
@State private var controlState: ControlState = .normal

AttachmentPreview(
    attachmentInfo: $config.attachments,
    previewURL: $previewURL,
    controlState: $controlState,
    onDelete: { url in
        context.delete(attachment: url)
    },
    onDismiss: {
        showPreview = false
    }
)
```

### Extra metadata on uploaded attachments

```swift
struct MyAttachmentMeta: AttachmentExtraInfo {
    let uploadedBy: String
    let serverID: String
}

context.onDefaultExtraInfo = {
    MyAttachmentMeta(uploadedBy: currentUser.name, serverID: UUID().uuidString)
}
```

### File info helpers (View extension)

```swift
// Available on any View via Attachment+Extensions
if let (url, name, size, modDate) = getFileInfo(fileUrl: localURL) {
    Text(name)
    if let size { Text(format(size: size)) }  // e.g. "2.4 MB"
}
```

---

## Figma Variants → SwiftUI

| Figma Property | Figma Value | SwiftUI |
|---------------|-------------|---------|
| State | Active | `controlState: .normal` (default) |
| State | Read-only | `controlState: .readOnly` |
| State | Disabled | `controlState: .disabled` |
| Add button | Visible | `.normal` state |
| Add button | Hidden | `.readOnly` or `.disabled` |
| Preview | Image / actual content | `AttachmentThumbnailWithPreviewStyle()` |
| Preview | No preview | Default icon via `getDefaultThumbnail(url:)` |
| Error | Upload failed | `.error` state on `AttachmentInfo` + `config.errorMessage` |
| Loading | In progress | `.uploading` state on `AttachmentInfo`; `context.isUploading` |

---

## Do's ✓ / Don'ts ✗

**Do:**
- Place at the **bottom** of the form — it is always the last field
- Use `AttachmentContext` and inject it via `.environment(context)` so child views share state
- Implement `AttachmentDelegate` to handle your upload/delete business logic
- Use `BasicAttachmentDelegate` for local/temporary storage or as a starting point for remote uploads
- Surface both the item-level `.error` state and `config.errorMessage` below the grid
- Optionally confirm before deleting (the built-in `AttachmentPreview` already does this)
- Update `config.maxCount` to cap the number of attachments

**Don't:**
- Use `AttachmentItem` — the real type is `AttachmentInfo`
- Assume `context.delete(attachment:)` calls the delegate for `.uploading` or `.error` items — it removes them directly
- Use for profile image uploads

---

## Related Components

- [forms.md](forms.md) — other input form cells
- [illustrated-message.md](illustrated-message.md) — empty state before any attachment is added
