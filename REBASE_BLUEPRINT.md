# Telegraher Rebase Blueprint

## Step-by-Step Instructions for Applying Fork Patches to Upstream 11.4.2

---

## Pre-Rebase Summary

| Metric | Value |
|---|---|
| Fork base | `release-9.3.3_3026` |
| Target upstream | `release-11.4.2-5469` |
| Fork-specific files (new) | **9 Java files** in `com.evildayz.*` |
| Modified upstream Java files | **246 files** in `org.telegram.*` |
| Modified native (JNI) files | **7 files** |
| Modified resources | **69 files** (icons, strings, layouts) |
| Modified build/config files | **~10 files** |
| Total fork diff | **39,529 lines** |
| Full exported diff | [patches/full_fork.diff](file:///home/ubuntu/Telegraher/patches/full_fork.diff) |

---

## Patch Categories

The 246 modified upstream Java files fall into **5 categories** by type of change:

### Tier 1 — Fork Feature Hooks (17 files, ~538 fork-marker lines)
Direct `import com.evildayz.*` or calls to `ThePenisMightierThanTheSword`. These are the **critical** patches.

| File | Diff Lines | What the Hook Does |
|---|---|---|
| `messenger/SharedConfig.java` | ~200 | Multi-account storage, shadowban HashMap, device spoofing persistence, disable auto-update check |
| `messenger/ApplicationLoader.java` | ~200 | Replace `MAX_ACCOUNT_COUNT` loops with `activeAccounts` iteration, osmdroid init, push service fix, remove Google Maps/Location providers, hardcode files dir path |
| `messenger/BuildVars.java` | ~90 | Hardcode version/vendor/fingerprint as vanilla TG, disable billing, set GitHub as update URL |
| `messenger/NotificationsController.java` | ~260 | Custom notification icon via `ThePenisMightierThanTheSword.getNotificationIcon()` |
| `messenger/MediaDataController.java` | ~230 | Disable premium emojis in listing & headers |
| `messenger/FilesMigrationService.java` | ~30 | Package path fix for file migration |
| `messenger/voip/VoIPService.java` | ~150 | Disable start/end beep, HD voice settings |
| `ui/ChatActivity.java` | ~880 | Deletion marks, message history, shadowban filter, sticker hiding, spoiler disable, save-to-gallery everywhere, forwarding tabs, restrict-content bypass |
| `ui/LaunchActivity.java` | ~370 | Telegraher settings menu entry, multi-account drawer patches |
| `ui/ProfileActivity.java` | ~790 | UID/DC display, copy-ID-on-click, admin member listing, DC geo info |
| `ui/LogoutActivity.java` | ~60 | Account folder wipe on logout |
| `ui/CacheControlActivity.java` | ~160 | Custom DB cleanup, WAL mode toggle |
| `ui/Cells/ChatMessageCell.java` | ~230 | Deletion marks rendering, sticker/animated-sticker hiding, emoji status disable |
| `ui/Cells/DialogCell.java` | ~30 | Emoji status disable |
| `ui/Cells/DrawerUserCell.java` | ~90 | Phone number hiding, username display |
| `ui/Cells/UserCell.java` | ~20 | Emoji status disable |
| `ui/Cells/ProfileSearchCell.java` | ~20 | Emoji status disable |

### Tier 2 — Multi-Account Rewrite (~113 `MAX_ACCOUNT_COUNT` replacements)
All `for (int a = 0; a < UserConfig.MAX_ACCOUNT_COUNT; a++)` loops replaced with `for (int a : SharedConfig.activeAccounts)`. Affects **~80+ files** across `messenger/`, `ui/`, and `tgnet/`.

### Tier 3 — Code Cleanup (~82 refactoring lines)
Try-with-resources conversions, stream closing, memory leak fixes. Spread across **~50 files**. These are **optional** — upstream may have already applied similar fixes.

### Tier 4 — Google Service Removal
Removal of Google Maps, Google Location, Google Vision, Google ML Kit, Play Billing references. Replaced with osmdroid for maps.

### Tier 5 — Resources & Cosmetics (69 files)
Custom icons, Telegraher string translations (EN, RU, AR, FA, DE, ES, IT, KO, NL, PT-BR), notification layouts, shortcuts XML.

### Native (JNI) — 7 Files

| File | Purpose |
|---|---|
| `jni/TgNetWrapper.cpp` | Multi-account support in native layer |
| `jni/tgnet/ConnectionsManager.cpp` | Multi-account + device spoofing in connections |
| `jni/tgnet/ConnectionsManager.h` | Header changes for above |
| `jni/tgnet/Defines.h` | Account limit constant change |
| 3 third-party files | Minor bug fixes (libvpx, openh264, webrtc) |

---

## Phase 0: Preparation

```bash
# You should already be in the repo with upstream fetched
cd /home/ubuntu/Telegraher

# Verify remotes
git remote -v
# origin    git@github.com:intelQong/Telegraher.git
# upstream  https://github.com/DrKLO/Telegram.git

# Verify upstream tags exist
git tag -l 'release-11*'
# release-11.0.0-5143
# release-11.1.3-5244
# release-11.4.2-5469

# Export the full fork diff for reference (already done)
mkdir -p patches
git diff release-9.3.3_3026..graher_9.33 > patches/full_fork.diff
```

---

## Phase 1: Create Clean Upstream Branch

```bash
# Create new branch from upstream 11.4.2
git checkout -b graher_11.4 release-11.4.2-5469

# Verify you're on clean upstream code
git log --oneline -1
# Should show: "update to 11.4.2 (5469)"
```

---

## Phase 2: Copy Fork-Specific Files (Zero Conflicts)

These files don't exist in upstream, so they can be directly copied.

```bash
# Create the evildayz package directory
mkdir -p TMessagesProj/src/main/java/com/evildayz/code/telegraher/session
mkdir -p TMessagesProj/src/main/java/com/evildayz/code/telegraher/ui

# Checkout all 9 fork-specific files from graher_9.33
git checkout graher_9.33 -- \
  TMessagesProj/src/main/java/com/evildayz/code/telegraher/ThePenisMightierThanTheSword.java \
  TMessagesProj/src/main/java/com/evildayz/code/telegraher/TelegraherSettingsActivity.java \
  TMessagesProj/src/main/java/com/evildayz/code/telegraher/ThMessageHistoryActivity.java \
  TMessagesProj/src/main/java/com/evildayz/code/telegraher/ThShadowbanManagerActivity.java \
  TMessagesProj/src/main/java/com/evildayz/code/telegraher/session/ThSessionManagerActivity.java \
  TMessagesProj/src/main/java/com/evildayz/code/telegraher/session/THSessionInfoActivity.java \
  TMessagesProj/src/main/java/com/evildayz/code/telegraher/session/THDeviceSpoofingEditActivity.java \
  TMessagesProj/src/main/java/com/evildayz/code/telegraher/ui/ThTextDetailCell.java \
  TMessagesProj/src/main/java/com/evildayz/code/telegraher/ui/ThTextCheckShadowbanCell.java

git add -A && git commit -m "Phase 2: Add Telegraher fork-specific files (9 files)"
```

---

## Phase 3: Copy Fork Resources (Icons, Strings, Layouts)

```bash
# Checkout all fork-specific resources
git checkout graher_9.33 -- \
  TMessagesProj/src/main/res/drawable-hdpi/telegraher_eyez.png \
  TMessagesProj/src/main/res/drawable-hdpi/telegraher_notification.png \
  TMessagesProj/src/main/res/drawable-mdpi/telegraher_eyez.png \
  TMessagesProj/src/main/res/drawable-mdpi/telegraher_notification.png \
  TMessagesProj/src/main/res/drawable-xhdpi/telegraher_eyez.png \
  TMessagesProj/src/main/res/drawable-xhdpi/telegraher_notification.png \
  TMessagesProj/src/main/res/drawable-xxhdpi/telegraher_eyez.png \
  TMessagesProj/src/main/res/drawable-xxhdpi/telegraher_notification.png

# Checkout custom launcher icons (all densities)
git checkout graher_9.33 -- \
  TMessagesProj/src/main/res/mipmap-hdpi/ic_launcher_sa.png \
  TMessagesProj/src/main/res/mipmap-mdpi/ic_launcher_sa.png \
  TMessagesProj/src/main/res/mipmap-xhdpi/ic_launcher_sa.png \
  TMessagesProj/src/main/res/mipmap-xxhdpi/ic_launcher_sa.png \
  TMessagesProj/src/main/res/mipmap-xxxhdpi/ic_launcher_sa.png

# Checkout icon backgrounds and foregrounds
git checkout graher_9.33 -- \
  $(git diff --name-only release-9.3.3_3026..graher_9.33 -- \
    'TMessagesProj/src/main/res/drawable-*/icon_background_sa.png' \
    'TMessagesProj/src/main/res/mipmap-*/icon_background*' \
    'TMessagesProj/src/main/res/mipmap-*/icon_foreground*' \
    'TMessagesProj/src/main/res/mipmap-*/ic_launcher.png' \
    'TMessagesProj/src/main/res/mipmap-*/ic_launcher_round.png' \
  )

git add -A && git commit -m "Phase 3: Add Telegraher icons and drawables"
```

---

## Phase 4: Apply Build System Patches

This is **manual work** — you cannot cherry-pick these because the upstream build files changed significantly.

### 4.1 — `gradle.properties`

Edit to set Telegraher values while keeping upstream structure:

```properties
# Change these values:
APP_VERSION_CODE=5469     # keep upstream code version
APP_VERSION_NAME=11.4.2   # keep upstream version name  
APP_PACKAGE=com.evildayz.code.telegraher2  # fork package

# Remove this line (fork doesn't use it):
# IS_PRIVATE=false
```

### 4.2 — `build.gradle` (root)

Remove Huawei and GMS plugin lines:

```diff
 dependencies {
     classpath 'com.android.tools.build:gradle:8.4.2'
-    classpath 'com.google.gms:google-services:4.3.15'
-    classpath 'com.huawei.agconnect:agcp:1.9.1.301'
 }
```

Remove Huawei repo:

```diff
 repositories {
     mavenCentral()
     google()
-    maven { url 'https://developer.huawei.com/repo/' }
 }
```

### 4.3 — `TMessagesProj/build.gradle`

Strip Google Play dependencies, add fork dependencies:

```diff
 dependencies {
-    implementation 'com.google.android.gms:play-services-maps:18.1.0'
-    implementation 'com.google.android.gms:play-services-auth:20.4.0'
-    implementation 'com.google.android.gms:play-services-vision:20.1.3'
-    implementation 'com.google.android.gms:play-services-wearable:18.0.0'
-    implementation 'com.google.android.gms:play-services-location:21.0.1'
-    implementation 'com.google.android.gms:play-services-wallet:19.1.0'
-    implementation 'com.stripe:stripe-android:2.0.2'
-    implementation 'com.google.mlkit:language-id:16.1.1'
-    implementation 'com.android.billingclient:billing:6.0.1'
-    implementation 'com.google.android.play:integrity:1.3.0'
-    implementation 'com.google.android.gms:play-services-safetynet:18.0.1'
-    implementation 'com.google.android.gms:play-services-mlkit-subject-segmentation:16.0.0-beta1'
-    implementation 'com.google.android.gms:play-services-mlkit-image-labeling:16.0.8'
+    implementation 'org.osmdroid:osmdroid-android:6.1.14'
+    implementation "org.apache.commons:commons-lang3:3.12.0"
     implementation 'com.google.code.gson:gson:2.11.0'
+    implementation 'com.googlecode.mp4parser:isoparser:1.0.6'
 }
```

Remove `apply plugin: 'com.google.gms.google-services'` at the bottom.

### 4.4 — `settings.gradle`

Keep only the modules you need:

```gradle
include ':TMessagesProj'
include ':TMessagesProj_App'
// Remove: TMessagesProj_AppHuawei, _AppHockeyApp, _AppStandalone
```

### 4.5 — `TMessagesProj_App/build.gradle`

Adapt from old fork's `TMessagesProj_App/build.gradle` — update compileSdk/targetSdk to 34, keep signing config, keep product flavors.

```bash
git commit -am "Phase 4: Adapt build system for Telegraher fork"
```

---

## Phase 5: Apply Tier 1 — Fork Feature Hooks (17 files)

> [!CAUTION]
> These are **manual patches**. The upstream code has changed significantly, so you cannot `git apply` the old diffs. You must read each patch and re-apply the **intent** to the new code.

### Priority Order (start with easiest, build up):

#### 5.1 — Easy patches (apply first to get compiling)

| # | File | Diff Lines | What to Do |
|---|---|---|---|
| 1 | `BuildVars.java` | 90 | Hardcode values, remove billing imports, set GitHub URL, add `BUILD_VENDOR`/`BUILD_DUROV` constants |
| 2 | `UserCell.java` | 20 | Add `ThePenisMightierThanTheSword.starrMark()` call for emoji status |
| 3 | `ProfileSearchCell.java` | 20 | Same starrMark() pattern |
| 4 | `DialogCell.java` | 30 | Same starrMark() pattern |
| 5 | `FilesMigrationService.java` | 30 | Fix package path to `telegraher2` |
| 6 | `LogoutActivity.java` | 60 | Add account folder wipe + `activeAccounts.remove()` |
| 7 | `DrawerUserCell.java` | 90 | Hide phone, show username in drawer |

#### 5.2 — Medium patches

| # | File | Diff Lines | What to Do |
|---|---|---|---|
| 8 | `VoIPService.java` | 150 | Add beep disable checks, HD voice toggle |
| 9 | `NotificationsController.java` | 260 | Replace notification icon with `getNotificationIcon()` |
| 10 | `CacheControlActivity.java` | 160 | Add DB cleanup, WAL mode, Telegraher menu entry |
| 11 | `MediaDataController.java` | 230 | Filter premium emojis from listings |
| 12 | `ApplicationLoader.java` | 200 | Loop replacements, osmdroid init, push service fix, remove Google providers |
| 13 | `SharedConfig.java` | 200 | Add `activeAccounts`, `thAccounts`, `shadowBannedHM`, device spoofing storage, `saveAccounts()` methods |

#### 5.3 — Hard patches (most conflict-prone)

| # | File | Diff Lines | What to Do |
|---|---|---|---|
| 14 | `LaunchActivity.java` | 370 | Add Telegraher settings entry, drawer modifications |
| 15 | `ProfileActivity.java` | 790 | Add ID/DC display sections, admin count, copy-ID, message history link |
| 16 | `ChatMessageCell.java` | 230 | Deletion marks, sticker hiding, emoji status |
| 17 | `ChatActivity.java` | 880 | Shadowban filtering, deletion marks, message history, sticker/spoiler hide, save-to-gallery, forwarding tabs, restrict-content bypass |

### How to Apply Each Patch

For each file, follow this process:

```bash
# 1. View the old fork patch
git diff release-9.3.3_3026..graher_9.33 -- <path/to/File.java>

# 2. Open the NEW upstream file (on graher_11.4 branch)
# 3. Find the equivalent locations in the new code
# 4. Apply the same logical changes
# 5. Test compilation after each file

# Example for BuildVars.java:
git diff release-9.3.3_3026..graher_9.33 -- \
  TMessagesProj/src/main/java/org/telegram/messenger/BuildVars.java
# Then manually edit the 11.4.2 version of BuildVars.java
```

```bash
git commit -am "Phase 5: Apply Tier 1 fork feature hooks"
```

---

## Phase 6: Apply Tier 2 — Multi-Account Loop Rewrites

This is a systematic find-and-replace across ~80 files:

```bash
# The pattern to find (upstream uses):
for (int a = 0; a < UserConfig.MAX_ACCOUNT_COUNT; a++)

# Replace with:
for (int a : SharedConfig.activeAccounts)

# Also handle variations like:
for (int i = 0; i < UserConfig.MAX_ACCOUNT_COUNT; i++)
# → for (int i : SharedConfig.activeAccounts)
```

> [!TIP]
> Use your IDE's "Find & Replace in Files" with regex:
> ```regex
> for \(int (\w+) = 0; \1 < UserConfig\.MAX_ACCOUNT_COUNT; \1\+\+\)
> ```
> Replace with: `for (int $1 : SharedConfig.activeAccounts)`

Also need to update `UserConfig.java` to raise/remove `MAX_ACCOUNT_COUNT`:

```bash
# Check what upstream sets it to
git show release-11.4.2-5469:TMessagesProj/src/main/java/org/telegram/messenger/UserConfig.java | grep MAX_ACCOUNT
```

```bash
git commit -am "Phase 6: Apply multi-account loop rewrites"
```

---

## Phase 7: Apply Native (JNI) Patches

4 meaningful native files to patch:

```bash
# View each native patch
git diff release-9.3.3_3026..graher_9.33 -- TMessagesProj/jni/TgNetWrapper.cpp
git diff release-9.3.3_3026..graher_9.33 -- TMessagesProj/jni/tgnet/ConnectionsManager.cpp
git diff release-9.3.3_3026..graher_9.33 -- TMessagesProj/jni/tgnet/ConnectionsManager.h
git diff release-9.3.3_3026..graher_9.33 -- TMessagesProj/jni/tgnet/Defines.h
```

Key changes:
- `Defines.h`: Change `MAX_ACCOUNT_COUNT` from 4 to dynamic/large value
- `ConnectionsManager.cpp/h`: Device spoofing parameters passed through JNI
- `TgNetWrapper.cpp`: Multi-account JNI bindings

```bash
git commit -am "Phase 7: Apply native JNI patches for multi-account and device spoofing"
```

---

## Phase 8: Apply Tier 3 — Code Cleanup (Optional)

The try-with-resources and memory leak fixes may already be addressed in upstream 11.4.2. **Check before applying.**

```bash
# For each refactored file, compare:
# 1. What the fork changed (try-with-resources, etc.)
# 2. What upstream 11.4.2 already has
# 3. Only apply if upstream still has the old pattern
```

> [!TIP]
> Skip this phase initially. These are quality-of-life improvements that don't affect functionality. Come back to them after the app compiles and runs.

---

## Phase 9: Apply String Resources & Translations

```bash
# Checkout Telegraher-specific strings
# IMPORTANT: Don't overwrite the full strings.xml — only ADD Telegraher entries

# View what strings were added:
git diff release-9.3.3_3026..graher_9.33 -- TMessagesProj/src/main/res/values/strings.xml | grep '^+'

# Manually append Telegraher-specific string entries to each locale:
# values/strings.xml        (English - primary)
# values-ru/strings.xml     (Russian)
# values-ar/strings.xml     (Arabic)
# values-fa/strings.xml     (Persian)
# values-de/strings.xml     (German)  — if applicable
# values-es/strings.xml     (Spanish) — if applicable
```

Also patch:
- `values/colors.xml` — Telegraher custom colors
- `values/styles.xml` — Notification styles
- `xml/shortcuts.xml` — Custom shortcuts
- `AndroidManifest.xml` — Permission changes, service declarations

```bash
git commit -am "Phase 9: Apply Telegraher string resources and translations"
```

---

## Phase 10: Google Services Removal

Upstream 11.4.2 has **more** Google dependencies than 9.3.3. Systematically remove:

```bash
# 1. Delete google-services.json (if present)
rm -f TMessagesProj/google-services.json TMessagesProj_App/google-services.json

# 2. Remove Google Maps provider references (upstream added more)
# Search for GoogleMapsProvider, GoogleLocationProvider in Java files
grep -r "GoogleMapsProvider\|GoogleLocationProvider\|IMapsProvider" \
  TMessagesProj/src/main/java/org/telegram/ --include="*.java" -l

# 3. Remove Play Billing references
grep -r "BillingController\|billingClient\|ProductDetails" \
  TMessagesProj/src/main/java/org/telegram/ --include="*.java" -l

# 4. Remove MLKit/Vision references
grep -r "com.google.mlkit\|com.google.android.gms.vision" \
  TMessagesProj/src/main/java/org/telegram/ --include="*.java" -l

# 5. Remove Play Integrity
grep -r "integrity\|SafetyNet\|SAFETYNET" \
  TMessagesProj/src/main/java/org/telegram/ --include="*.java" -l

# For each found reference, either:
# - Remove the code block
# - Replace with a stub (return false/null)
# - Replace with fork alternative (osmdroid for maps)
```

```bash
git commit -am "Phase 10: Remove Google Play Services dependencies"
```

---

## Verification Checklist

### Compile Test
```bash
./gradlew :TMessagesProj:compileDebugJavaWithJavac 2>&1 | head -100
# Fix errors iteratively until clean
```

### Full Build
```bash
./gradlew :TMessagesProj_App:assembleAfatRelease
```

### Functional Smoke Test
- [ ] App installs and launches
- [ ] Telegraher settings menu accessible
- [ ] Can log in with phone number
- [ ] Multi-account works (add 2nd account)
- [ ] Session manager displays accounts
- [ ] Device spoofing fields editable
- [ ] Ads are not displayed
- [ ] Deletion marks appear when messages deleted
- [ ] Shadowban feature works
- [ ] Custom notification icon shows
- [ ] Save-to-gallery option appears in menus
- [ ] Premium emojis are hidden (when setting enabled)
- [ ] Sticker hiding works
- [ ] KABOOM data wipe works

---

## Final Push

```bash
git push origin graher_11.4
```

---

## Future Updates

When upstream releases a new version (e.g., 11.5.0):

```bash
# 1. Fetch
git fetch upstream --tags

# 2. Check new tag
git log --oneline release-11.4.2-5469..release-11.5.0-XXXX -- TMessagesProj/src/ | wc -l

# 3. Merge (incremental updates are much easier than major version jumps)
git checkout graher_11.4
git merge release-11.5.0-XXXX

# 4. Resolve conflicts (should be minimal for minor updates)
# 5. Build and test
```

> [!TIP]
> After this initial rebase, future updates will be **incremental merges** (not full rebases), which are much easier. The biggest pain is this first jump from 9.3.3 → 11.4.2.
