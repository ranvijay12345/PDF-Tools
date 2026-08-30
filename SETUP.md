# PDF Tools — Setup & Credentials Checklist

Everything you must provide for the paid/cloud features of **PDF Tools - Edit &
More** (`com.pdftools.app`) to work on a real build. **The app builds and runs
offline with zero setup** — the values below only unlock AdMob (real ads),
Google Drive, and Play Billing (premium). Sections 6–7 cover release signing and
publishing to Google Play.

All secrets go in **`local.properties`** (git-ignored) or CI environment
variables. Nothing sensitive is committed. Each key falls back to a safe
test/placeholder default if omitted, so the project always compiles.

---

## 0. `local.properties` template

Add these lines to `local.properties` in the project root (create it if
missing — Android Studio already keeps `sdk.dir` there):

```properties
# --- AdMob (leave as-is to use Google TEST ads) ---
ADMOB_APP_ID=ca-app-pub-XXXXXXXXXXXXXXXX~XXXXXXXXXX
ADMOB_BANNER_ID=ca-app-pub-XXXXXXXXXXXXXXXX/XXXXXXXXXX
ADMOB_INTERSTITIAL_ID=ca-app-pub-XXXXXXXXXXXXXXXX/XXXXXXXXXX
ADMOB_REWARDED_ID=ca-app-pub-XXXXXXXXXXXXXXXX/XXXXXXXXXX
ADMOB_APP_OPEN_ID=ca-app-pub-XXXXXXXXXXXXXXXX/XXXXXXXXXX

# --- Play Billing (premium / remove ads) ---
PREMIUM_PRODUCT_ID=premium_remove_ads

# --- Google Drive (OAuth Web client ID from Google Cloud) ---
GOOGLE_OAUTH_SERVER_CLIENT_ID=XXXXXXXXXXXX-xxxxxxxx.apps.googleusercontent.com
```

> ⚠️ Until you replace the AdMob IDs with your own, **you must keep Google's
> test IDs** (the defaults). Showing real ads on test IDs — or test ads on a
> published app — violates AdMob policy.

---

## 1. AdMob (real ads)

1. Create/sign in at <https://apps.admob.com>.
2. **Apps → Add app** → Android → get the **App ID**
   (`ca-app-pub-…~…`) → put in `ADMOB_APP_ID`.
3. Create 4 ad units (Banner, Interstitial, Rewarded, App Open) → copy each
   unit ID into the matching `ADMOB_*_ID`.
4. Rebuild. The App ID is injected into the manifest via a placeholder; the
   unit IDs reach code through `BuildConfig`. (App-Open shows when the app
   returns to the foreground, skipping the first cold start and throttled so
   it appears at most once every few minutes.)

*Consent:* the UMP (User Messaging Platform) consent flow is already wired and
runs before ads initialize. For EEA testing you can add a test device in the
AdMob **Privacy & messaging** settings.

### 1a. `app-ads.txt` (authorized sellers — do this after publishing)

`app-ads.txt` is **not** part of the app/APK. It is a plain-text file you host
at the **root of your developer website**; AdMob crawls it to confirm you are
authorized to sell your app's ad inventory (fraud prevention). It only matters
once the app is **published with your real AdMob IDs** — it does nothing for the
test IDs above.

**Contents (AdMob-only apps — exactly one line):**

```
google.com, pub-XXXXXXXXXXXXXXXX, DIRECT, f08c47fec0942fa0
```

- `google.com` — literal, always.
- `pub-XXXXXXXXXXXXXXXX` — **the only part you change.** Your AdMob *publisher*
  ID = `pub-` + the 16 digits **before the `~`** in your App ID
  (`ca-app-pub-XXXXXXXXXXXXXXXX~XXXXXXXXXX`). Find it in **AdMob → Settings →
  Account information**.
- `DIRECT` — literal, always.
- `f08c47fec0942fa0` — literal, always (Google's certification-authority ID,
  identical for every publisher).

If you later add other networks/mediation (AppLovin, Unity, Meta, …), add **one
line per network** — each network gives you its own line to paste.

**Hosting it (free, via GitHub Pages):**
1. Create a **public** repo named exactly `<your-username>.github.io`.
2. Add a file named `app-ads.txt` (lowercase) with the line above.
3. Repo **Settings → Pages** → Source **Deploy from a branch** → **main / (root)**
   → Save.
4. Confirm it opens as plain text at
   `https://<your-username>.github.io/app-ads.txt`.

**Linking it to AdMob:**
1. Set this URL as your **developer website**: Play Console → **Settings → Developer
   account → Developer details → Website** (this is the URL AdMob crawls). Also set
   the per-app **Website** under **Store presence → Store settings** if present.
2. **AdMob → Apps → View all apps → [your app] → app-ads.txt** tab → copy the
   expected line shown there (guarantees the correct `pub-` ID) → **Check for
   updates**.
3. Status becomes **Authorized** after the crawl — allow 24 h up to a few days
   for a new file.

> Gotchas: file must be at the **root** (`/app-ads.txt`), served as plain text;
> the developer/website domain must **match exactly** (watch `www.` vs non-`www.`);
> the app must be published/linked so AdMob has a website to crawl. Nothing in the
> Android code, manifest, or Gradle changes for this step.

---

## 2. Google Drive integration

The app uses **Google Sign-In** for consent to the least-privilege
`drive.file` scope, plus the **Drive REST API**. You need a Google Cloud
project, an OAuth client for your app's signing certificate, and the Drive API
enabled.

### 2a. Create the Google Cloud project
1. <https://console.cloud.google.com> → create a project (e.g. "PDF Tools").
2. **APIs & Services → Library → Google Drive API → Enable.**

### 2b. OAuth consent screen
1. **APIs & Services → OAuth consent screen.**
2. User type **External**; fill app name, support email, developer email.
3. **Scopes:** add `.../auth/drive.file` (per-file access — the only scope the
   app requests).
4. While in **Testing**, add each Google account you'll test with under **Test
   users**. (Publishing/verification is only needed for public release; the
   `drive.file` scope is *not* a restricted scope, so verification is light.)

### 2c. Get your signing SHA-1 fingerprints
Android OAuth clients are bound to your app's **package name + SHA-1**. You need
one client per signing key. Get the SHA-1:

```bash
# Debug key (auto-generated by Android Studio):
keytool -list -v -keystore ~/.android/debug.keystore \
  -alias androiddebugkey -storepass android -keypass android

# Release key (your upload/release keystore):
keytool -list -v -keystore /path/to/release.keystore -alias <your-alias>
```

On Windows the debug keystore is at
`%USERPROFILE%\.android\debug.keystore`.

> **Important:** the debug build uses application ID
> **`com.pdftools.app.debug`** (`.debug` suffix) and release uses
> **`com.pdftools.app`**. Register **both** package names, and add the SHA-1 for
> **every** keystore you build/publish with (debug, release, and Play App
> Signing — see 2e).

### 2d. Create OAuth client IDs
In **APIs & Services → Credentials → Create credentials → OAuth client ID**:

1. **Android** client for `com.pdftools.app` + release SHA-1.
2. **Android** client for `com.pdftools.app.debug` + debug SHA-1 (so sign-in
   works during development).
3. **Web application** client — copy its **Client ID** into
   `GOOGLE_OAUTH_SERVER_CLIENT_ID`. (Google Sign-In needs a Web client ID as the
   "server client ID"; the Android clients authorize the app itself.)

### 2e. Play App Signing SHA-1 (for published builds)
If you distribute via Play, Google re-signs your app. After uploading your
first build, copy the **App signing key certificate SHA-1** from
**Play Console → your app → Setup → App integrity** and add it as another
Android OAuth client for `com.pdftools.app`. Without this, Drive sign-in fails
only on the Play-installed build.

### 2f. `google-services.json` (optional here)
This app talks to Drive via the REST client and Google Sign-In configured in
code, so a `google-services.json` / the Google Services Gradle plugin is **not
required** to compile or run. If you later add Firebase or FCM, drop
`google-services.json` into `app/` and apply the plugin then.

---

## 3. Play Billing (premium / remove ads)

Real purchases require the app to be uploaded to the Play Console and the
in-app product to be created.

1. **Play Console → Create app**, using application ID `com.pdftools.app`.
2. Upload at least one build (internal testing track is fine) so Play knows the
   app.
3. **Monetize → Products → In-app products → Create product.**
   - **Product ID:** must match `PREMIUM_PRODUCT_ID`
     (default `premium_remove_ads`).
   - Type: **one-time** (non-consumable). Set a price and **Activate** it.
4. **Testing:** add testers under **Setup → License testing** (they can
   purchase without being charged). Install the app from an internal-testing
   link with a tester account — sideloaded/`.debug` builds cannot query Play
   products.
5. In the app: **Settings → Remove Ads (Premium)** shows the live price, runs
   the purchase, acknowledges it, and flips the ad-free flag. **Restore
   purchase** re-queries owned purchases.

> Billing only works for builds installed **through Play** with the **release
> application ID** and a signed build matching your Play upload key. The
> `.debug` build will not find the product.

---

## 4. Quick verification checklist

| Feature | Works offline w/o setup? | Needs credentials |
|---|---|---|
| All PDF tools (merge, split, OCR, edit text, …) | ✅ Yes | — |
| Ads (test) | ✅ Yes (test IDs) | Real IDs for production |
| Google Drive connect / upload / download | ❌ | §2 OAuth client + SHA-1 |
| Premium purchase / restore | ❌ | §3 Play product + Play install |

---

## 5. Build

```bash
./gradlew :app:assembleDebug      # dev build (com.pdftools.app.debug)
./gradlew :app:assembleRelease    # signed release (configure signingConfig)
./gradlew :app:bundleRelease      # signed .aab for Play upload
```

If a Drive dependency ever triggers a `META-INF` packaging clash, it's already
handled by the `packaging { resources { excludes … } }` block in
`app/build.gradle.kts` — add the offending path there if a new one appears.

---

## 6. Release signing

Play requires an **`.aab`** signed with a **release keystore** you keep forever
— lose it and you can no longer update the app.

1. **Create the keystore (one time).** Android Studio: **Build → Generate Signed
   App Bundle / APK → Android App Bundle → Create new…** Choose a path (e.g.
   `C:\Users\<you>\keystores\pdftools-release.jks`), a strong password, an alias,
   and validity ≥ 25 years. **Back up the file and passwords** — they are
   unrecoverable.
2. **Provide the credentials** (git-ignored) in `local.properties`:
   ```properties
   RELEASE_STORE_FILE=C:/Users/<you>/keystores/pdftools-release.jks
   RELEASE_STORE_PASSWORD=********
   RELEASE_KEY_ALIAS=pdftools
   RELEASE_KEY_PASSWORD=********
   ```
3. **Wire the signing config.** The release build's `signingConfig` must be set
   (either via the Android Studio wizard, or a `signingConfigs { release { … } }`
   block reading the values above from `local.properties`). Grab the release
   keystore **SHA-1** for the Drive OAuth client (§2c):
   ```bash
   keytool -list -v -keystore /path/to/pdftools-release.jks -alias <your-alias>
   ```

---

## 7. Publishing to Google Play

**Account prerequisites (start early):**
- One-time **$25** developer registration.
- Identity / D-U-N-S **verification** (required for recent accounts; can take days).
- Personal accounts created after Nov 2023 must run a **closed test with 12+
  testers for 14 days** before production unlocks — the internal/closed testing
  below satisfies this.

**Steps:**
1. **Play Console → Create app.** Name **PDF Tools - Edit & More**, type *App*,
   **Free** (with in-app purchases), application ID `com.pdftools.app`.
2. **Complete required declarations:** Privacy policy URL (host it on the same
   GitHub Pages site as `app-ads.txt`), **Data safety** (files stay on-device;
   Drive is opt-in; ads use device identifiers), Content rating, Target audience,
   **Ads = yes**.
3. **Store listing:** descriptions, 512×512 icon, 1024×500 feature graphic, ≥ 2
   phone screenshots.
4. **Upload to a testing track first.** **Testing → Internal testing → Create new
   release** → upload `app-release.aab`. On first upload Play enrolls you in
   **Play App Signing** — copy the **App signing key certificate SHA-1** and add
   it as another Drive OAuth client (§2e). Create/test the Billing product
   (`premium_remove_ads`) and add License testers (§3) on this track. Install via
   the tester link and verify tools, ads, Drive, and Premium.
5. **Promote to production.** **Production → Create new release** → reuse the
   bundle → set a staged rollout % → submit. First review typically takes a few
   days to ~a week. Live at
   `https://play.google.com/store/apps/details?id=com.pdftools.app`.

> Reminder: real AdMob IDs (§1), the Drive OAuth clients + all SHA-1s (§2), and
> the active Billing product (§3) must be in place before the published build's
> paid/cloud features work. The `.debug` build cannot query Billing or (without
> its own OAuth client) Drive.
