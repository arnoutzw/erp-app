# Release History

All notable changes to Lab Inventory, organized by git commit hash.

---

## 2026-02-01

### `8a9899f` - Replace version number with git hash and build date
- Remove APP_VERSION, use only GIT_HASH and BUILD_DATE
- Add VERSION_HISTORY array with commit history
- Add expandable version history in About section

### `836a1d2` - Replace low stock warnings with project overallocation warnings
- Remove low stock settings from Settings
- Remove minStock field from item form
- Warn only when items have more allocated to projects than available

### `bcb1994` - Add automatic version check with update banner
- Periodically checks for new version (every 5 minutes)
- Shows blue update banner at top when new version available
- Click banner to clear caches and reload to latest version

### `d8c5138` - Update scanner for CODE128 barcode support
- Rename QRScanner to BarcodeScanner
- Use rectangular scan area (300x100) optimized for linear barcodes
- Explicitly support CODE_128 and QR_CODE formats

### `c106270` - Update documentation for recent changes
- QR codes → CODE128 barcodes throughout
- Update navigation section (Locaties, Items, BoM, Map)
- Add new features documentation

### `77b9824` - Remove Boxes tab
- Boxes now accessible via Locaties nested menu

### `36239b4` - Add sync photos from product links feature
- Button in Settings to download images from item URLs
- Shows progress: syncing count, success, failed, skipped
- Skips items that already have photos stored

### `078e893` - Update lab map to match physical layout
- Dark theme map
- Left side: Blok 2, Blok 1, Blok 3 stacked vertically
- Right side: Plank 1, Plank 2 with vertical text

### `e05aecb` - Add nested menus for Locaties view
- Parse locations into parent/child hierarchy (e.g., "Blok 1 Lade A" → "Blok 1" > "Lade A")
- Two-level expandable menus in shelves view

### `6b6cc19` - Sort boxes alphabetically by label

### `76635ed` - Add missing JSON fields to GUI display and editing
- Add subcategory field to item form and display
- Add minStock per-item threshold
- Add product link field with clickable button
- Add product image URL field with thumbnail preview
- Add box type display

### `cad748c` - Switch from QR codes to CODE128 barcodes for label printing
- Replace qrcodejs with JsBarcode library
- Synchronous barcode generation for reliable printing

### `b42af99` - Change batch print to use separate pages per label
- Each label prints on separate 40x30mm page

### `e4b1be2` - Update lab map to show dynamic locations from Locaties tab

### `2c3aea7` - Fix QR code generation - generate directly in LabelImage

### `4b2aad8` - Remove PDF generation, use canvas images for all labels
- Canvas-based image generation for precise sizing
- 472x354px at 300 DPI = 40x30mm labels

### `39aaa0e` - Add force update button in About section

### `be8ee7d` - Switch single label printing to canvas images, add About section

### `f53369e` - Update all print functions to use PDF generation

### `9c64a7f` - Fix QR code printing - generate reliably before print

### `a6a5da5` - Add Locaties (shelves/drawers) view with collapsible boxes

### `2c52abf` - Improve project assignment UI with -/+ quantity buttons

### `1111e41` - Add interactive lab map view

### `3e686f7` - Fix dark mode text contrast for readability

### `f10f02e` - Hide delete button when no filter active

### `1d7d111` - Add dropdown for existing projects in ItemModal

### `ed50ea1` - Add value field and projects/BoM system
- Track item monetary value
- Assign items to projects with quantities
- Bill of Materials (BoM) view

### `2cec0ff` - Add batch delete for filtered items

### `4ae0127` - Pre-generate QR codes, separate from React completely

### `87894e7` - Add dark mode theme and Material Design icons

### `a3397d8` - Add delete prompt when item quantity becomes 0

### `d716df5` - Fix quantity button event handling

### `bc70971` - Fix memory leaks, race conditions, and error handling

### `7cfab0c` - Fix remaining QRCode React conflicts in print functions

### `5389a5d` - Fix QRCode React error #321 with proper deferral

### `cd05eca` - Add OWASP security improvements
- Content Security Policy (CSP)
- XSS prevention with output encoding
- Input sanitization
- File upload security

### `337a37e` - Fix QR code generation React conflicts in print functions

### `f88244d` - Fix QRCode CorrectLevel.M undefined error

### `55c2a76` - Add ZIP export/import with photos
- Export inventory as ZIP with photos folder
- Import from ZIP or legacy JSON

### `ee8c7c4` - Fix QRCode component React conflict error #321

### `39bcb4f` - Update README with latest features

### `5429e64` - Fix QRCode component crash with empty values

### `3c1dbcf` - Add 'Wis' button to clear search query

### `12d7fe5` - Add auto-update functionality to service worker

### `7a7bb5a` - Fix script tag parsing issue breaking the app

### `7f1ab2f` - Fix template literal syntax in item QR print function

### `e74f929` - Add QR code display for item 12NC codes

### `85b8e71` - Update README with IndexedDB, photos, and 12NC documentation

### `66390e9` - Add Philips 12NC identification codes to items
- Auto-generated 12-digit codes based on category
- Format: XXYY ZZZ ZZZZZ (product group, coding centre, product number)

### `0258d02` - Add IndexedDB storage with photo support
- Primary storage in IndexedDB
- Photo blobs stored locally
- Auto-resize to max 800px

### `2bebe09` - Fix print dialog not appearing due to popup blocker

### `2266e14` - Add README.md with project documentation

### `795405d` - Add printer emoji to print buttons for better visibility

### `7ca5282` - Add QR code printing functionality via CUPS

---

## 2026-01-31

### `9393842` - Change box click to navigate to Items tab with filter

### `9b5f1d1` - Fix +X more button to actually show more items

### `a21d399` - Fix box click modal display and add storage box creation feature

### `27c6221` - Initial upload
- Box-based inventory organization
- Category filtering (Electronics, 3D Printing, Mechanical, Other)
- PWA with offline support
- Mobile-optimized touch interface
