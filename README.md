# Reader — Minimal Offline Manga & Manhwa Reader

Reader is a native, distraction-free Android application built with Kotlin and Jetpack Compose designed exclusively for reading manga and manhwa from local `.zip` and `.cbz` archives stored on your device.

It is **100% offline**, requiring zero internet connectivity, no user accounts, and zero cloud services.

---

## 1. Initial Storage Setup

When Reader is launched for the first time:

1. **Welcome Screen**: Explains the storage setup and invites you to pick a storage location.
2. **Select Storage Location**: Tapping **Select Storage Location** launches Android's system folder picker (Storage Access Framework).
3. **Automatic Reader Directory**:
   - You can choose any folder (e.g. `Documents`, `Downloads`, or `Internal Storage`).
   - Reader will automatically create a folder named `Reader` inside your selected directory (or reuse an existing `Reader` directory if one is already present).
4. **Persistent Permission**:
   - Persistent read and write permissions are retained across device restarts and app restarts.
5. **Folder Structure**:
   ```
   Internal Storage/
   └── Reader/
       ├── One Piece/
       │   ├── Chapter 001.cbz
       │   ├── Chapter 002.cbz
       │   └── Chapter 003.zip
       ├── Solo Leveling/
       │   ├── Chapter 001.cbz
       │   └── Chapter 002.zip
       └── Boruto/
           ├── Chapter 001.zip
           └── Chapter 002.cbz
   ```
   - Each immediate subfolder inside `Reader` represents one **Manga / Manhwa**.
   - Each `.zip` or `.cbz` file inside that subfolder represents one **Chapter**.

---

## 2. Implemented Features

### Local Storage & Scanning
- **Android Storage Access Framework (SAF)**: Secure, privacy-preserving local storage access with persistent URI permissions.
- **Direct Local Reading**: Reads directly from the user's storage without copying or altering originals.
- **Natural Numeric Sorting**: Alphanumeric sorting ensures chapters and images sort logically (`Chapter 1`, `Chapter 2`, `Chapter 10`, `Chapter 1095`, `1.jpg`, `2.jpg`, `10.jpg`).
- **Archive Support**: Unpacks and streams images from `.zip` and `.cbz` archives containing JPG, JPEG, PNG, WEBP, and GIF files while ignoring non-image entries (`Thumbs.db`, `.DS_Store`, `__MACOSX`, metadata).
- **Temporary Cache Management**: Chapter images are extracted only on-demand to the app's internal cache and automatically pruned to prevent disk bloat.
- **Empty Folder Filtering**: Folders without valid chapter archives are automatically hidden.

### Three Primary Sections (Bottom Navigation)
1. **Library**:
   - Displays all detected manga folders alphabetically in an adaptive grid.
   - Shows chapter counts and extracted cover thumbnails when available.
   - One-tap refresh button and settings dialog.
2. **Bookmarks**:
   - Displays all manually bookmarked chapters grouped neatly under their parent manga.
   - Direct chapter opening and instant bookmark removal without affecting reading progress.
3. **Continue Reading**:
   - Automatically tracks the last-read chapter and exact page index for every manga.
   - Resumes seamlessly from the saved page on tap.

### Reading Interface
- **Single Page Mode**:
  - Fits image to screen width while maintaining the original aspect ratio (never cropped).
  - Multi-touch pinch-to-zoom (up to 5x) and pan navigation with double-tap reset.
  - Horizontal swipe pagination respecting reading direction.
- **Vertical Continuous Mode (Webtoon / Manhwa)**:
  - Continuous vertical layout with zero spacing/margins between pages.
  - Lazy-loaded pages to maintain smooth scrolling and low memory usage.
  - Visible page tracker.
- **Reading Directions**:
  - Right-to-Left (RTL) — Manga standard (default).
  - Left-to-Right (LTR) — Western comic standard.
- **Distraction-Free Controls Overlay**:
  - Controls are hidden by default; a single tap on the screen reveals the overlay.
  - Includes previous/next chapter navigation, quick page scrubber slider, reading mode toggle, reading direction toggle, chapter bookmark button, and back button.

### Settings
- Change Reader folder via system folder picker.
- Rescan library.
- Configure default reading mode (Single Page vs Vertical Continuous).
- Configure default reading direction (RTL vs LTR).

### Privacy & Offline Guarantees
- No `android.permission.INTERNET` declared.
- No network libraries, cloud sync, analytics, trackers, or ads.

---

## 3. Build & Installation Instructions

### Prerequisites
- JDK 11 or higher
- Android SDK (API 36 compile SDK, min SDK 24)

### Building the Project
To compile the debug APK:
```bash
gradle assembleDebug
```
The output APK will be located at:
`app/build/outputs/apk/debug/app-debug.apk`

To compile the release APK:
```bash
gradle assembleRelease
```

### Running Unit and Robolectric Tests
```bash
gradle :app:testDebugUnitTest
```

---

## 4. Tests Performed and Verified

1. **`NaturalOrderComparatorTest`**:
   - Verified natural sorting of image files (`1.jpg`, `2.jpg`, `3.jpg`, `10.jpg`, `11.jpg`, `20.jpg`).
   - Verified natural sorting of chapters (`Chapter 1.cbz`, `Chapter 2.cbz`, `Chapter 10.zip`, `Chapter 1095.cbz`, `Chapter 1100.cbz`).
   - Verified leading zero handling (`001.png`, `002.png`, `010.png`).
   - Verified null safety and tie-breaker comparisons.

2. **`ArchiveManagerTest`**:
   - Verified `isSupportedImage()` whitelist filtering (`.jpg`, `.jpeg`, `.png`, `.webp`, `.gif`) and blacklist filtering (`.DS_Store`, `Thumbs.db`, `__MACOSX`, `.txt`, `.xml`).
   - Verified chapter archive extraction and image ordering from dynamically created `.cbz` archives.
   - Verified graceful handling and meaningful error messages for empty zip archives.
   - Verified corrupted archive recovery without crashing the application.

3. **`ReaderDatabaseTest`**:
   - Verified saving and retrieving `ReadingProgressEntity` with chapter name, page index, and timestamp.
   - Verified progress updates for the same manga maintain a single reading position.
   - Verified manual `BookmarkEntity` addition, queries, and deletion.
   - Verified bookmark operations do not affect reading progress data.
   - Verified app settings persistence for folder URI, default reading mode, and direction.

4. **`ExampleRobolectricTest`**:
   - Verified application resources and `app_name` string ("Reader").

5. **Gradle Build Verification**:
   - `compile_applet` passed successfully.
   - `gradle :app:testDebugUnitTest` passed all test suites (27 actionable tasks, BUILD SUCCESSFUL).

---

## 5. Known Limitations
- Supported archive formats are strictly `.zip` and `.cbz` (proprietary RAR/CBR and PDF are excluded by design specifications).
- Reading progress requires that archive filenames remain unchanged within the manga folder; if a chapter is renamed outside the app, Reader gracefully handles the missing file and prompts the user.
