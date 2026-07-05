# Telegraher Repository — Microtask Breakdown

**Total:** 386 changed files | 15,791 insertions / 9,289 deletions | 9 new fork-specific files | ~108 microtasks

---

## P0 — CRITICAL (must do first, foundation)

### Build System (18 tasks)

| # | Task | Complexity |
|---|------|------------|
| B1 | `build.gradle` (root) — remove Google Services & Huawei plugin deps | Easy |
| B2 | `settings.gradle` — remove Huawei/HockeyApp/Standalone module includes | Easy |
| B3 | `gradle.properties` — set `APP_PACKAGE=com.evildayz.code.telegraher2`, version 9.33.31 | Easy |
| B4 | `gradle-wrapper.properties` — update Gradle 7.5 distribution URL | Easy |
| B5 | `TMessagesProj/build.gradle` — strip Google Play deps (maps, auth, vision, wearable, location, wallet, MLKit, billing, integrity, safetynet) | Medium |
| B6 | `TMessagesProj/build.gradle` — add osmdroid, commons-lang3, isoparser dependencies | Easy |
| B7 | `TMessagesProj/build.gradle` — remove `apply plugin: 'com.google.gms.google-services'` | Easy |
| B8 | `TMessagesProj_App/build.gradle` — adapt compileSdk 33, targetSdk 31, minSdk 19, NDK 21.4.7075529, signing config | Medium |
| B9 | `TMessagesProj_App/build.gradle` — configure product flavors (bundle, arm64_v8a, armeabi_v7a, afat) | Medium |
| B10 | `TMessagesProj/proguard-rules.pro` — fork-specific keep rules | Easy |
| B11 | `Dockerfile` — CI build container definition | Easy |
| B12 | `Dockerfile_bundle` — bundle APK Docker build | Easy |
| B13 | `.github/workflows/Dockerfile_bundle.yml` — GitHub Actions CI workflow | Easy |
| B14 | `builditbitch.sh` — Docker CI build script | Easy |
| B15 | Gradle wrapper — keep `gradlew`, `gradlew.bat`, `gradle/wrapper/gradle-wrapper.jar` | Easy |
| B16 | Remove Huawei App module build files (`TMessagesProj_AppHuawei/build.gradle`) | Easy |
| B17 | Remove HockeyApp module build files (`TMessagesProj_AppHockeyApp/build.gradle`) | Easy |
| B18 | `apkdiff.py` — APK diff utility script | Easy |

### Fork-Specific Java Files (9 tasks — zero conflicts, direct copy)

| # | File | Lines |
|---|------|-------|
| F1 | `com/evildayz/code/telegraher/ThePenisMightierThanTheSword.java` — core utility: DC geo info, video/photo size multipliers, fonts, notification icon selector, account management, sticker sizes, JSON helpers | 185 |
| F2 | `com/evildayz/code/telegraher/TelegraherSettingsActivity.java` — main settings UI with ~50 toggle options | 1,146 |
| F3 | `com/evildayz/code/telegraher/ThMessageHistoryActivity.java` — message history viewer | 351 |
| F4 | `com/evildayz/code/telegraher/ThShadowbanManagerActivity.java` — shadowban management UI | 343 |
| F5 | `com/evildayz/code/telegraher/session/ThSessionManagerActivity.java` — session manager for multi-account | 382 |
| F6 | `com/evildayz/code/telegraher/session/THSessionInfoActivity.java` — session info details (device, IP, DC) | 456 |
| F7 | `com/evildayz/code/telegraher/session/THDeviceSpoofingEditActivity.java` — device spoofing editor (brand, model, SDK) | 235 |
| F8 | `com/evildayz/code/telegraher/ui/ThTextDetailCell.java` — custom UI cell component | 201 |
| F9 | `com/evildayz/code/telegraher/ui/ThTextCheckShadowbanCell.java` — shadowban toggle cell UI | 354 |

### Resources — Critical

| # | Task |
|---|------|
| R20 | Update all `AndroidManifest.xml` variants (permissions, services, activities) — main + config/debug + config/release |

### Google Removal — Critical

| # | Task |
|---|------|
| G6 | Remove `google-services.json` |

---

## P1 — HIGH (core features, compile-breaking if missing)

### Feature Hooks

| # | File | Diff Lines | What It Does |
|---|------|-------------|--------------|
| H13 | `messenger/SharedConfig.java` | 706 | `activeAccounts`, `thAccounts`, `shadowBannedHM`, device spoofing persistence, `saveAccounts()` methods, disable auto-update check |
| H12 | `messenger/ApplicationLoader.java` | 257 | Loop replacements with `activeAccounts`, osmdroid init, push service fix, remove Google providers, hardcode files dir |
| H1 | `messenger/BuildVars.java` | 85 | Hardcode version/vendor/fingerprint, disable billing, set GitHub update URL, add `BUILD_VENDOR`/`BUILD_DUROV` constants |
| H17 | `ui/ChatActivity.java` | 876 | Shadowban filtering, deletion marks display, message history, sticker/spoiler hide, save-to-gallery everywhere, forwarding tabs, restrict-content bypass |
| H15 | `ui/ProfileActivity.java` | 787 | UID/DC display, copy-ID-on-click, admin member count, DC geo info, message history link |
| H14 | `ui/LaunchActivity.java` | 367 | Telegraher settings menu entry, multi-account drawer patches |

### Native JNI Patches

| # | File | Diff Lines | What It Does |
|---|------|-------------|--------------|
| N1 | `jni/tgnet/Defines.h` | 12 | Change `MAX_ACCOUNT_COUNT` from 4 to dynamic value |
| N2 | `jni/tgnet/ConnectionsManager.h` | 21 | Add device spoofing parameter signatures |
| N3 | `jni/tgnet/ConnectionsManager.cpp` | 58 | Device spoofing params through JNI, multi-account connection management |
| N4 | `jni/TgNetWrapper.cpp` | 105 | Multi-account JNI bindings, native account handling |

### Multi-Account Loop Rewrites (~113 replacements across ~80 files)

| # | Scope | Description |
|---|-------|-------------|
| M1 | `messenger/` package | Replace `for (int a = 0; a < UserConfig.MAX_ACCOUNT_COUNT; a++)` with `for (int a : SharedConfig.activeAccounts)` |
| M2 | `ui/` package | Same pattern replacement |
| M3 | `tgnet/` package | Same pattern replacement |
| M4 | All packages | Handle iterator variable name variations (`i`, `b`, etc.) |
| M5 | `UserConfig.java` | Remove/raise `MAX_ACCOUNT_COUNT` constant |
| M6 | `AccountInstance.java` | Multi-account support |
| M7 | `MessagesController.java` | ~10+ loop replacements |
| M8 | `MessagesStorage.java` | ~10+ loop replacements |
| M9 | `ContactsController.java` | ~5 loop replacements |
| M10 | `SendMessagesHelper.java` | ~5 loop replacements |
| M11 | `MediaController.java` | ~3 loop replacements |
| M12 | `DownloadController.java` | ~3 loop replacements |
| M13 | `StatsController.java` | ~3 loop replacements |
| M14 | Remaining ~70 files | 1-3 loop replacements each |

---

## P2 — MEDIUM (features, can compile without)

### Feature Hooks (Medium)

| # | File | Diff Lines | What It Does |
|---|------|-------------|--------------|
| H16 | `ui/Cells/ChatMessageCell.java` | 234 | Deletion marks rendering, sticker hiding, animated sticker hiding, emoji status disable |
| H9 | `messenger/NotificationsController.java` | 259 | Replace notification icon with `getNotificationIcon()`, custom notification behavior |
| H11 | `messenger/MediaDataController.java` | 228 | Filter premium emojis from listings and headers |
| H10 | `ui/CacheControlActivity.java` | 162 | Custom DB cleanup, WAL mode toggle, Telegraher menu entry |
| H8 | `messenger/voip/VoIPService.java` | 149 | Start/end beep disable, HD voice toggle, proximity sensor modes |
| H7 | `ui/Cells/DrawerUserCell.java` | 87 | Hide phone number in drawer, show username instead |
| H6 | `ui/LogoutActivity.java` | 59 | Account folder wipe + `activeAccounts.remove()` on logout |
| H5 | `messenger/FilesMigrationService.java` | 30 | Fix package path to `telegraher2` |
| H4 | `ui/Cells/DialogCell.java` | 30 | `starrMark()` call for emoji status |
| H3 | `ui/Cells/ProfileSearchCell.java` | 21 | `starrMark()` call for emoji status |
| H2 | `ui/Cells/UserCell.java` | 21 | `starrMark()` call for emoji status |

### Google Service Removal

| # | Task | Files |
|---|------|-------|
| G1 | Remove Google Maps provider | `GoogleMapsProvider.java`, `IMapsProvider.java`, `LocationController.java`, `LocationActivity.java`, `ChatAttachAlertLocationLayout.java`, `LocationActivityAdapter.java`, `LocationSharingService.java` |
| G2 | Remove Google Location provider | `GoogleLocationProvider.java` |
| G3 | Remove Play Billing | `BillingController.java`, `PaymentFormActivity.java` |
| G4 | Remove Google Vision/MLKit | `LanguageDetector.java`, `MrzRecognizer.java`, `CameraScanActivity.java`, `PassportActivity.java` |
| G5 | Remove SafetyNet / Play Integrity | Relevant files |

---

## P3 — LOW (cosmetic, no functional impact)

### Resources

| # | Task | Details |
|---|------|---------|
| R7 | `values/strings.xml` (English) | ~50 new Telegraher string resources |
| R8 | `values-ru/strings.xml` (Russian) | ~50 new string resources |
| R9 | `values-ar/strings.xml` (Arabic) | ~30 new string resources |
| R10 | `values-fa/strings.xml` (Persian) | ~30 new string resources |
| R11 | `values-de/strings.xml` (German) | ~10 new string resources |
| R12 | `values-es/strings.xml` (Spanish) | ~10 new string resources |
| R13 | `values-it/strings.xml` (Italian) | ~10 new string resources |
| R14 | `values-ko/strings.xml` (Korean) | ~10 new string resources |
| R15 | `values-nl/strings.xml` (Dutch) | ~10 new string resources |
| R16 | `values-pt-rBR/strings.xml` (Brazilian Portuguese) | ~10 new string resources |
| R1 | Telegraher notification icons | 4 densities (`drawable-*`) |
| R2 | Telegraher eyez icon | 4 densities (`drawable-*`) |
| R3 | Custom launcher icons `ic_launcher_sa.png` | 5 densities (`mipmap-*`) |
| R4 | Icon backgrounds `icon_background_sa.png` | 5 densities |
| R5 | Icon foregrounds | 2+ files |
| R6 | Custom `ic_launcher.png` and `ic_launcher_round.png` | 10 files |
| R17 | `values/colors.xml` — Telegraher custom colors | ~5 entries |
| R18 | `values/styles.xml` — notification styles | ~3 entries |
| R19 | `xml/shortcuts.xml` — custom shortcuts | ~5 entries |
| R21 | `.editorconfig` — project code style | 1 file |

### Vendored Library Patches

| # | File | Description |
|---|------|-------------|
| V1 | `androidx/recyclerview/widget/ItemTouchHelper.java` | Minor patch |
| V2 | `com/google/android/exoplayer2/drm/HttpMediaDrmCallback.java` | Minor patch |
| V3 | `com/google/android/exoplayer2/upstream/cache/CachedContentIndex.java` | Minor patch |
| V4 | `com/google/android/exoplayer2/util/Util.java` | Minor patch |
| V5 | `com/google/zxing/multi/qrcode/QRCodeMultiReader.java` | Minor patch |

### Third-Party C++ Bug Fixes

| # | File | Diff Lines |
|---|------|-------------|
| N5 | `jni/third_party/libvpx/source/libvpx/vpx_util/vpx_debug_util.c` | 17 |
| N6 | `jni/third_party/openh264/src/codec/processing/src/downsample/downsample.cpp` | 15 |
| N7 | `jni/voip/webrtc/rtc_base/experiments/encoder_info_settings.cc` | 17 |

### Code Cleanup (Optional)

| # | Task | Scope |
|---|------|-------|
| C1 | Convert try-finally to try-with-resources | ~20 files |
| C2 | Fix memory leaks (stream closing) | ~15 files |
| C3 | Remove legacy SDK <=19 conditional branches | ~15 files |

---

## P4 — TRIVIAL (non-code, can defer indefinitely)

### Documentation

| # | File | Description |
|---|------|-------------|
| D1 | `README.md` | Project overview, feature list, links |
| D2 | `README_BUILD.md` | Build instructions |
| D3 | `README_CHANGES.md` | Full changelog (444 lines) |
| D4 | `REBASE_BLUEPRINT.md` | Step-by-step rebase guide to upstream 11.4.2 (527 lines) |
| D5 | `README_old.md` | Archived previous README |
| D6 | `ANALYSIS.md` | Third-party fork analysis |
| D7 | `Tools/EmojiTextureMaker.m` | macOS tool for emoji spritesheet generation |

### Patch Export

| # | Task | Description |
|---|------|-------------|
| P1 | `patches/full_fork.diff` | 2.5MB full diff (39,529 lines) |

---

## Execution Order

```
P0 (Critical) ──> P1 (High) ──> P2 (Medium) ──> P3 (Low) ──> P4 (Trivial)
    │                │               │               │               │
    ├─ B1-B18        ├─ H13,H12,H1   ├─ H16,H9,H11   ├─ R7-R21       ├─ D1-D7
    ├─ F1-F9         ├─ H17,H15,H14  ├─ H10,H8,H7    ├─ V1-V5        ├─ P1
    ├─ R20           ├─ N1-N4        ├─ H6-H2        ├─ N5-N7
    └─ G6            └─ M1-M14       └─ G1-G5        └─ C1-C3
```

Each tier must compile before moving to the next.
