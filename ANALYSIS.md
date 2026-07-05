# Telegraher Fork — Repository Analysis

## Overview

**Telegraher** is a modified Android Telegram client (fork of [DrKLO/Telegram](https://github.com/DrKLO/Telegram)) by nikitasius, currently cloned to `intelQong/Telegraher`. It focuses on removing restrictions, disabling ads, adding privacy/power-user features, and removing Google dependencies.

| Property | Value |
|---|---|
| **Current Version** | `9.33.31` (versionCode `3026031`) |
| **Upstream Base** | Telegram `9.3.3` |
| **Latest Upstream** | **~11.x+** (as of July 2025) |
| **Branch** | `graher_9.33` (single branch, 619 commits) |
| **Package** | `com.evildayz.code.telegraher2` |
| **Language** | Java 11 (no Kotlin) |
| **Min SDK** | 19 (Android 4.4) |
| **Target SDK** | 31 (Android 12) |

> [!CAUTION]
> **The fork is ~2 major versions behind upstream Telegram (9.3.3 vs ~11.x).** This is the single biggest challenge for any update work.

---

## Project Architecture

### Module Structure

```
Telegraher/
├── TMessagesProj/          # Library module (core Telegram code + fork patches)
│   ├── src/main/java/
│   │   ├── org/telegram/   # Upstream Telegram code (modified)
│   │   │   ├── messenger/  # Core logic (119 files, massive ones)
│   │   │   ├── ui/         # UI layer (142 files + 8 subdirs)
│   │   │   ├── tgnet/      # Network/TL layer
│   │   │   └── SQLite/     # SQLite bindings
│   │   ├── com/evildayz/   # ★ Fork-specific code (9 files)
│   │   ├── com/google/     # Vendored Google code
│   │   └── androidx/       # Vendored RecyclerView
│   ├── jni/                # Native C/C++ (5842 files)
│   └── config/             # Signing keystores & manifests
├── TMessagesProj_App/      # Application module (APK assembly)
├── Tools/                  # Build utilities
├── Dockerfile              # CI build environment
└── .github/workflows/      # GitHub Actions CI
```

### Build Toolchain

| Tool | Version | Latest Stable | Status |
|---|---|---|---|
| Gradle | 7.5 | 8.10+ | ⚠️ **Outdated** |
| Android Gradle Plugin | 7.2.2 | 8.7+ | ⚠️ **Outdated** |
| NDK | 21.4.7075529 | 27+ | ⚠️ **Outdated** |
| CMake | 3.10.2 | 3.30+ | ⚠️ **Outdated** |
| compileSdk | 33 | 35 | ⚠️ **Outdated** |
| targetSdk | 31 | 35 | ⚠️ **Outdated (Play Store requires 34+)** |
| Docker base | `gradle:7.0.2-jdk11` | — | ⚠️ **Outdated** |

---

## Fork-Specific Code (9 custom files)

All custom fork logic lives in `com.evildayz.code.telegraher`:

| File | Purpose |
|---|---|
| [ThePenisMightierThanTheSword.java](file:///home/ubuntu/Telegraher/TMessagesProj/src/main/java/com/evildayz/code/telegraher/ThePenisMightierThanTheSword.java) | Core utility: DC geo info, video/photo size multipliers, fonts, notification icons, account management, sticker sizes, JSON helpers |
| [TelegraherSettingsActivity.java](file:///home/ubuntu/Telegraher/TMessagesProj/src/main/java/com/evildayz/code/telegraher/TelegraherSettingsActivity.java) | Main settings UI (1147 lines, ~50 toggle options) |
| [ThMessageHistoryActivity.java](file:///home/ubuntu/Telegraher/TMessagesProj/src/main/java/com/evildayz/code/telegraher/ThMessageHistoryActivity.java) | Message history viewer |
| [ThShadowbanManagerActivity.java](file:///home/ubuntu/Telegraher/TMessagesProj/src/main/java/com/evildayz/code/telegraher/ThShadowbanManagerActivity.java) | Shadowban management UI |
| [ThSessionManagerActivity.java](file:///home/ubuntu/Telegraher/TMessagesProj/src/main/java/com/evildayz/code/telegraher/session/ThSessionManagerActivity.java) | Session manager for multi-account |
| [THSessionInfoActivity.java](file:///home/ubuntu/Telegraher/TMessagesProj/src/main/java/com/evildayz/code/telegraher/session/THSessionInfoActivity.java) | Session info details |
| [THDeviceSpoofingEditActivity.java](file:///home/ubuntu/Telegraher/TMessagesProj/src/main/java/com/evildayz/code/telegraher/session/THDeviceSpoofingEditActivity.java) | Device spoofing editor |
| [ThTextDetailCell.java](file:///home/ubuntu/Telegraher/TMessagesProj/src/main/java/com/evildayz/code/telegraher/ui/ThTextDetailCell.java) | Custom UI cell |
| [ThTextCheckShadowbanCell.java](file:///home/ubuntu/Telegraher/TMessagesProj/src/main/java/com/evildayz/code/telegraher/ui/ThTextCheckShadowbanCell.java) | Shadowban toggle cell |

### Integration Points (files in `org/telegram/` that import `evildayz`)

The fork hooks into **17 upstream files** via imports/calls to `ThePenisMightierThanTheSword` or other fork classes:

| Layer | Modified Files |
|---|---|
| **Core** | `ApplicationLoader`, `SharedConfig`, `FilesMigrationService`, `NotificationsController`, `MediaDataController` |
| **UI** | `ChatActivity`, `LaunchActivity`, `ProfileActivity`, `LogoutActivity`, `CacheControlActivity` |
| **Cells** | `ChatMessageCell`, `DialogCell`, `DrawerUserCell`, `UserCell`, `ProfileSearchCell` |
| **VoIP** | `VoIPService` |

---

## Feature Summary

### Custom Features (from settings + changelog)

| Category | Features |
|---|---|
| **Anti-restriction** | Disabled ads, disabled remote deletions, full access in "restrict saving content" chats, full access in secret chats, timed medias don't expire |
| **Multi-account** | Unlimited accounts (from NekoX), session manager, device spoofing per account |
| **UI/UX** | Custom sticker sizes (0.25x–2x), hide stickers/animated/video stickers, notification icon selector, phone number hiding, deletion marks, message history |
| **Media** | Photo sizes up to 3840px, round video size/bitrate controls, HD GIFs (1080p), HD voices (48kHz), video up to 8K (H.264) |
| **Privacy** | Shadowban feature, emulator detection bypass, legit phone/SIM spoofing, "don't use Apple/iTunes" toggle, disabled spoilers |
| **Network** | Connection speed override (auto/slow/high), upload/download speedup ("Graherium") |
| **Premium bypass** | Disable premium emojis, disable emoji status, star mark for everyone/none |
| **Security** | "KABOOM" data wipe, kill app button, package spoofing as vanilla Telegram |

---

## Key Challenges for Updating

### 1. Massive Version Gap (9.3.3 → 11.x)

> [!WARNING]
> Telegram upstream has added **hundreds of features** since 9.3.3: Stories, channels 2.0, message translation, chat themes revamp, new media editor, Telegram Premium expansions, business features, mini-apps, and massive UI refactors.

The biggest files that will have **extreme merge conflicts**:

| File | Lines | Size |
|---|---|---|
| [TLRPC.java](file:///home/ubuntu/Telegraher/TMessagesProj/src/main/java/org/telegram/tgnet/TLRPC.java) | 65,545 | 2.6MB |
| [ChatActivity.java](file:///home/ubuntu/Telegraher/TMessagesProj/src/main/java/org/telegram/ui/ChatActivity.java) | 30,595 | 1.7MB |
| [ChatMessageCell.java](file:///home/ubuntu/Telegraher/TMessagesProj/src/main/java/org/telegram/ui/Cells/ChatMessageCell.java) | 19,127 | — |
| [PhotoViewer.java](file:///home/ubuntu/Telegraher/TMessagesProj/src/main/java/org/telegram/ui/PhotoViewer.java) | 17,495 | 877KB |
| [MessagesController.java](file:///home/ubuntu/Telegraher/TMessagesProj/src/main/java/org/telegram/messenger/MessagesController.java) | 17,216 | 901KB |
| [MessagesStorage.java](file:///home/ubuntu/Telegraher/TMessagesProj/src/main/java/org/telegram/messenger/MessagesStorage.java) | 15,479 | 804KB |

### 2. Build Toolchain Modernization Required

- Gradle 7.5 → 8.x (deprecated API removals)
- AGP 7.2.2 → 8.x (namespace requirements, `lintOptions` → `lint`, `dexOptions` deprecated)
- NDK 21 → 25+ (C++ toolchain changes)
- `targetSdkVersion` 31 → 34+ (required for Play Store, notification permissions, etc.)

### 3. API & TL Schema Changes

`TLRPC.java` (65K lines) is auto-generated from the TL schema. Every upstream version changes it substantially. This file alone will be the most painful to update.

### 4. Native Code (JNI)

5,842 native files including BoringSSL, FFmpeg, libwebp, opus, rlottie, and custom tgnet code. Upstream has updated these dependencies multiple times.

---

## Recommended Update Strategy

> [!IMPORTANT]
> Given the enormous gap, a **rebase onto fresh upstream** is more practical than a merge.

### Option A: Fresh Rebase (Recommended)

1. Clone latest upstream `DrKLO/Telegram` master
2. Create a new branch from it
3. Re-apply the 9 fork-specific files (`com.evildayz.*`)
4. Re-apply the ~17 hook modifications in `org.telegram.*` files
5. Update build toolchain (Gradle/AGP/NDK/SDK)
6. Fix compilation errors, test

**Effort**: ~2-4 weeks (focused work)

### Option B: Incremental Merge

1. Add upstream as a remote
2. Attempt merge of each upstream version tag
3. Resolve thousands of conflicts

**Effort**: ~4-8 weeks (conflict resolution hell)

---

## What I Can Help With Now

Given your intent to update this fork, I can:

1. **Modernize the build system** — Upgrade Gradle, AGP, SDK versions, fix deprecated APIs
2. **Audit and document all fork patches** — Create a precise diff of every modification to upstream files, making rebase easier
3. **Upgrade dependencies** — Update AndroidX, BoringSSL, etc.
4. **Code quality improvements** — Fix lint warnings, modernize Java patterns
5. **Add upstream as a git remote** — Set up the merge/rebase workflow

What would you like to focus on first?
