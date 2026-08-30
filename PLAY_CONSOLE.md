# Play Console Submission Worksheet — PDF Tools

Fill in every `<...>` blank, then copy the answers into the Play Console as you go.
Sections are ordered the way the Console walks you through them. Pre-filled draft
text is provided where this app's behavior is already known — edit to taste.

App-specific facts baked in below (verify against the shipped release):
- Package / applicationId: `com.pdftools.app`
- Free app, contains ads (banner, interstitial, rewarded, app-open via AdMob)
- Has a **paid in-app product**: remove-ads / premium (`premium_remove_ads`)
- Uses **camera** (document scanner) + **Google Drive** with the narrow `drive.file` scope
- Core PDF processing is **fully offline**; Drive is opt-in only
- minSdk 26 (Android 8.0), targetSdk 36 (Android 16)

---

## 0. Pre-upload technical checklist (do before touching the Console)

- [ ] Release **signing config** wired into `build.gradle.kts` (keystore from `local.properties`)
- [ ] Upload keystore generated and **backed up somewhere safe** (losing it = can't update the app)
- [ ] `./gradlew :app:bundleRelease` produces a **signed** `app-release.aab`
- [ ] Release build uses **real AdMob unit IDs** (not the Google test fallbacks) — verify `local.properties` values are present
- [ ] `GOOGLE_OAUTH_SERVER_CLIENT_ID` set in `local.properties` (Drive sign-in fails silently without it)
- [ ] `versionCode` / `versionName` correct for this release (currently `1` / `1.0.0`)
- [ ] Smoke-tested a release build on a real device: every tool, ads show, Drive connect works

---

## 1. Create the app (Console → All apps → Create app)

| Field | Value |
|---|---|
| App name (30 char max) | `<PDF Tools>` |
| Default language | `<English (United States) – en-US>` |
| App or game | App |
| Free or paid | Free |
| Declarations | ☑ Developer Program Policies · ☑ US export laws |

---

## 2. Store listing (Console → Grow → Store presence → Main store listing)

### Text
| Field | Limit | Value |
|---|---|---|
| App name | 30 | `<PDF Tools>` |
| Short description | 80 | `<Merge, split, scan, compress & edit PDFs — fast, offline, private.>` |
| Full description | 4000 | see draft below |

**Full description draft:**
```
PDF Tools is an all-in-one PDF toolkit that works completely offline. Your files
never leave your device unless you choose to connect Google Drive.

FEATURES
• Merge, split, reorder and delete PDF pages
• Compress PDFs to shrink file size
• Convert images to PDF and PDF pages to images
• Scan documents with the camera — automatic edge detection, perspective
  correction and enhancement
• OCR text recognition and searchable PDFs
• Annotate: draw, highlight, add text, shapes and signatures
• Protect with a password, and more

PRIVATE BY DESIGN
All PDF processing happens on your device. There are no servers involved for
editing. Optional Google Drive support only accesses files you create or open
with the app (drive.file scope) — never your whole Drive.

Free to use, supported by ads. A one-time in-app purchase removes ads.
```

### Graphics (upload PNG/JPEG; no alpha on icon/feature graphic)
| Asset | Spec | Status |
|---|---|---|
| App icon | 512×512, 32-bit PNG | `<todo>` |
| Feature graphic | 1024×500 | `<todo>` |
| Phone screenshots (2–8) | 16:9 or 9:16, min 320px | `<todo>` |
| 7" tablet screenshots (optional) | | `<optional>` |
| 10" tablet screenshots (optional) | | `<optional>` |

---

## 3. Store settings (Console → Store presence → Store settings)

| Field | Value |
|---|---|
| App category | `<Productivity>` |
| Tags | `<PDF, documents, scanner>` |
| Contact email (public) | `ranviapp@gmail.com` |
| Contact phone (optional) | `<...>` |
| Contact website (optional) | `<...>` |
| Privacy policy URL (**required**) | `<https://.../privacy>` |

> A hosted privacy policy is **mandatory** because the app collects data (AdMob
> ad IDs, Drive account). See §7 for what it must disclose.

---

## 4. App content / policy declarations (Console → Policy → App content)

### Privacy policy
- URL: `<https://.../privacy>`

### Ads
- Contains ads? **Yes** (AdMob: banner, interstitial, rewarded, app-open)

### App access
- Is any functionality behind login? **No** (Drive sign-in is optional, not required to use the app)
- If a reviewer needs anything special, add instructions here: `<none / N/A>`

### Content rating (fill the questionnaire → IARC rating issued)
| Prompt | Answer |
|---|---|
| Category | Utility / Productivity |
| Violence / sexual / profanity / drugs | No |
| User-generated content shared? | No |
| Expected rating | `<Everyone / PEGI 3>` |

### Target audience
| Field | Value |
|---|---|
| Target age groups | `<18+>` (recommended — avoids Families policy + ad restrictions) |
| Appeals to children? | No |

### Data safety (Console → App content → Data safety) — **be accurate, mismatches cause rejection**
| Question | Answer |
|---|---|
| Does the app collect or share user data? | Yes |
| Data encrypted in transit? | Yes |
| Users can request deletion? | `<Yes/No — describe>` |

Data types to declare:
| Data type | Collected | Shared | Purpose | Notes |
|---|---|---|---|---|
| Device or other IDs (advertising ID) | Yes | Yes | Advertising | via AdMob |
| App activity / diagnostics (crash, performance) | `<Yes if you use crash reporting; else No>` | | Analytics | |
| Email address / account | `<Yes if Drive connected>` | No | App functionality | only when user connects Drive |
| Files & docs | No collection off-device | No | | processed locally; Drive access is user-initiated |

> AdMob's advertising ID collection **must** be declared — this is the most common
> rejection cause. Camera images and PDFs processed locally are **not** "collected"
> (they don't leave the device), so they aren't declared as collected data.

### Sensitive permissions / APIs
| Permission | Justification (enter in Console if prompted) |
|---|---|
| CAMERA | `<Document scanning — capture pages to create PDFs.>` |
| (No All-files / MANAGE_EXTERNAL_STORAGE) | App uses scoped storage / SAF only — confirm none declared |

### Government apps / Financial / Health
- Not applicable — answer **No** to each.

---

## 5. Google Drive / OAuth verification (do in Google Cloud Console, not Play)

| Item | Value |
|---|---|
| OAuth scope used | `https://www.googleapis.com/auth/drive.file` (narrow — no security assessment needed) |
| OAuth consent screen published? | `<todo>` |
| Any sensitive/restricted scopes? | **No** — keep it `drive.file` only, or a paid annual security review is triggered |

---

## 6. Pricing & distribution (Console → Monetize / Release → Countries)

| Field | Value |
|---|---|
| Free/paid | Free |
| Countries/regions | `<All / select list>` |
| Contains ads flag | Yes |
| In-app products | Create managed product `premium_remove_ads` (matches `PREMIUM_PRODUCT_ID`) — set price `<e.g. $2.99>` |

> Create the in-app product **before** or alongside the first release so purchases
> resolve. Billing testing requires the app to be on a test track first.

---

## 7. Privacy policy — must disclose (checklist for whatever page you host)

- [ ] What data is collected: advertising ID (AdMob), Google account email (only if Drive connected)
- [ ] That PDF/image processing is on-device and files aren't uploaded except to the user's own Drive on request
- [ ] Third parties: Google AdMob (ads), Google Drive API (optional)
- [ ] Links to Google's ad policy / how to opt out of personalized ads
- [ ] Data retention & deletion contact
- [ ] Developer contact email

---

## 8. Release — testing tracks first (Console → Test and release)

New/personal developer accounts must run **closed testing with ≥12 testers for 14
days** before production access unlocks. Plan for this lead time.

1. **Internal testing** — upload `app-release.aab`, add your own account as tester,
   install via opt-in link, verify everything on a real device.
2. **Closed testing** — recruit ≥12 testers, keep them opted-in ≥14 days.
3. Complete **App content** (§4) and **Data safety** — production is blocked until green.
4. **Production** — create release, upload the same/newer `.aab`, add release notes,
   submit for review.

### Release details (per track)
| Field | Value |
|---|---|
| Release name | `<1.0.0 (1)>` |
| App bundle | `app-release.aab` |
| Release notes (`<en-US>`) | `<Initial release: merge, split, scan, compress, OCR, annotate PDFs — fully offline.>` |

---

## 9. Final submit checklist

- [ ] Store listing complete (text + all required graphics)
- [ ] Content rating questionnaire done
- [ ] Data safety form done and matches actual behavior
- [ ] Ads declaration = Yes
- [ ] Privacy policy URL live and accurate
- [ ] Target audience set
- [ ] Signed `.aab` uploaded, no R8/signing errors
- [ ] targetSdk = 36 (Play requirement)
- [ ] Closed-testing requirement satisfied (new accounts)
- [ ] Reviewed for policy: ads spacing, no misleading claims, permissions justified
- [ ] Submitted for review
