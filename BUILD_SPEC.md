# PDF Tools — Subagent Build Spec (READ FIRST)

Native Android, Kotlin + Jetpack Compose, package root `com.pdftools.app`.
Source root: `D:\AI-Projects\PDF-tools\app\src\main\java\com\pdftools\app`.
Compose BOM 2024.09.03, Material3, min SDK 24, target 34. Manual DI (no Hilt).

## Conventions (MATCH EXACTLY)

### Getting a ViewModel with the app container
```kotlin
import com.pdftools.app.core.containerViewModel
val vm: MyViewModel = containerViewModel { container -> MyViewModel(container) }
```
`AppContainer` exposes: `pdfEngine: PdfEngine`, `ads: AdsManager`, `settings: SettingsRepository`, `recentFiles: RecentFilesRepository`. ViewModels take `AppContainer` (or the specific deps) as constructor args and use `viewModelScope`.

### Feature screen skeleton
Every feature screen is `@Composable fun XxxScreen(onBack: () -> Unit, onOpenResult: (Uri) -> Unit = {})` (some take `onOpenPdf`). Wrap content in `ToolScaffold`:
```kotlin
import com.pdftools.app.ui.common.ToolScaffold
import com.pdftools.app.ui.common.ProgressOverlay
ToolScaffold(title = "Merge", onBack = onBack) { padding ->
    Box(Modifier.padding(padding)) {
        // content
        ProgressOverlay(state)   // shows overlay while OpState.Running
    }
}
```
`ToolScaffold` already renders the top bar (with back), a bottom `AdBanner`, and a `bottomBar` slot. DO NOT add your own AdBanner.

### File pickers (in ui/common/Pickers.kt)
```kotlin
val pick = rememberPickDocuments(arrayOf("application/pdf")) { uris -> vm.setInputs(uris) }
val pickOne = rememberPickSingleDocument(arrayOf("application/pdf")) { uri -> vm.setInput(uri) }
val pickImages = rememberPickImages { uris -> vm.setImages(uris) }
// call pick() / pickOne() / pickImages() from a Button onClick
```

### OpState (core/OpState.kt)
`sealed interface OpState<out T> { Idle; Running(progress: Float?, message); Success(data); Error(throwable, message) }`. ViewModels expose `val state: StateFlow<OpState<...>>`. Collect with `val state by vm.state.collectAsStateWithLifecycle()`.

### Result handling + ads + recents (do this after every successful op)
```kotlin
// In ViewModel after producing a result File:
val uri = FileUtils.exportToDocuments(context, resultFile, "Merged_<ts>.pdf")
recentFiles.record(RecentFile(uri.toString(), name, "Merge", size, pages, System.currentTimeMillis()))
// then emit Success(uri)
// In the screen, on Success -> show a result card with "Open", "Share" buttons and call:
ads.maybeShowInterstitial(activity) { /* optional */ }
```
Get the Activity in a composable: `val activity = LocalContext.current as Activity`. Get AdsManager: `LocalAdsManager.current`.

### Sharing (create ShareUtils if not present, ui/common or core)
Use `FileUtils.shareUri(context, file)` -> `Intent(ACTION_SEND)` with `type="application/pdf"`, `FLAG_GRANT_READ_URI_PERMISSION`. Provide "Share", "Email" (ACTION_SEND with mailto-ish extras), and WhatsApp (`setPackage("com.whatsapp")`, fallback to chooser).

## PdfEngine API (com.pdftools.app.pdf) — ALL suspend unless noted
```kotlin
suspend fun pageCount(uri: Uri): Int
suspend fun merge(inputs: List<Uri>, onProgress: (Float)->Unit = {}): File
suspend fun splitToPages(input: Uri, selection: PageSelection? = null, onProgress): List<File>
suspend fun extractRange(input: Uri, selection: PageSelection, onProgress): File
suspend fun rotatePages(input: Uri, pageIndices: Set<Int>, degrees: Int, onProgress): File
suspend fun deletePages(input: Uri, pageIndices: Set<Int>): File
suspend fun reorderPages(input: Uri, order: List<Int>): File
suspend fun readMetadata(input: Uri): PdfMetadata
suspend fun writeMetadata(input: Uri, meta: PdfMetadata): File
suspend fun encrypt(input, userPassword, ownerPassword, permissions: PdfPermissions, keyLengthBits=256): File
suspend fun decrypt(input: Uri, password: String): File
suspend fun renderPage(uri: Uri, pageIndex: Int, dpi: Int = 96): Bitmap
// extension ops (same package, just call on engine instance):
suspend fun compress(input, level: CompressionLevel, onProgress): File
suspend fun pdfToImages(input, format: ImageFormat, dpi: Int, selection: PageSelection?=null, onProgress): List<File>
suspend fun imagesToPdf(images: List<Uri>, preset: PageSizePreset, fit: ImageFit, customWidthPt?, customHeightPt?, onProgress): File
suspend fun textWatermark(input, text, fontSize=48f, opacity=0.25f, rotationDeg=45.0, onProgress): File
suspend fun imageWatermark(input, watermark: Bitmap, opacity=0.25f, scale=0.5f, onProgress): File
suspend fun renderThumbnails(uri, dpi=48, onEach:(Int,Bitmap)->Unit)
suspend fun renderThumbnailList(uri, dpi=60): List<Bitmap>
```
Models (pdf/PdfModels.kt): `CompressionLevel{HIGH,MEDIUM,LOW}`, `ImageFormat{JPEG,PNG}`, `PageSizePreset{A4,LETTER,LEGAL,FIT_IMAGE}`, `ImageFit{FIT,FILL,STRETCH}`, `PageSelection.parse(expr, pageCount)`, `PdfMetadata(title,author,subject,keywords)`, `PdfPermissions(canPrint,canModify,canCopy,canAnnotate)`.

## FileUtils (core/FileUtils.kt)
`copyToCache`, `queryDisplayName(ctx,uri)`, `querySize(ctx,uri)`, `exportToDocuments(ctx, file, displayName, mime)`, `shareUri(ctx, file)`, `humanSize(bytes)`, `workDir(ctx)`, `PDF_MIME`.

## Data
`RecentFile(uri, displayName, tool, sizeBytes, pageCount, createdAt)` + `RecentFilesRepository.observeAll()/record()/remove()/clear()`.
`SettingsRepository`: `isPremium: Flow<Boolean>`, `themeMode: Flow<Int>` (0 sys,1 light,2 dark), `setPremium`, `setThemeMode`, `incrementOpCount()`.

## Rules
- Use `androidx.lifecycle.compose.collectAsStateWithLifecycle` (dep present).
- ViewModels take `AppContainer` as their single constructor arg: `class MergeViewModel(private val container: AppContainer) : ViewModel()`. The container exposes `appContext: Context`, so ViewModels never touch Activity/Composable context — use `container.appContext` for FileUtils/export calls. Get the VM in the screen via `containerViewModel { MergeViewModel(it) }`.
- Every screen: empty state (pick a file), configured state (options + action button), running (ProgressOverlay), success (result card with Open/Share), error (snackbar or inline text).
- Keep comments matching existing density (concise, purposeful). Kotlin official style.
- Do NOT edit AppRoot.kt, Routes.kt, build.gradle.kts, or any file outside your assigned screens unless creating a NEW shared helper in ui/common (coordinate: only create if missing). AppRoot.kt already declares your screen's signature — match it exactly.
