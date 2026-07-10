# Telegraher → Upstream 11.4.2 — Continuation / AI Handoff

> **Read this first if you're picking up this work.** It is the living source of truth for the
> rebase of the Telegraher fork from Telegram **9.3.3** onto upstream **11.4.2**. Update the
> **Progress Tracker** at the bottom on every commit. Companion docs: `ANALYSIS.md` (fork overview),
> `MICROTASKS.md` (task IDs B/F/H/M/N/G/R), `REBASE_BLUEPRINT.md` (original git recipe — but see the
> **Corrections** below; some of it is now outdated).

---

## Goal & decisions (agreed with repo owner)

- **MVP-first:** get the fork **compiling with core features** on upstream 11.4.2. Defer translations,
  cosmetics, and optional cleanup. Not full 246-file parity in one pass.
- **Verify via GitHub Actions CI** — the dev environment has **no Android SDK/NDK**; it cannot build
  the APK locally (native layer = 5,842 files). Push the branch, read CI logs, fix, repeat.
- **Branch:** all work goes on `claude/telegrapher-upstream-plan-ucozoi`. Never push elsewhere.
- **Target tag:** `release-11.4.2-5469` (latest reachable DrKLO/Telegram tag).

## Git remotes / refs you need
- `origin` = `intelQong/Telegraher`. `origin/graher_9.33` = **the fork source of truth (Telegram 9.3.3 base).**
- `upstream` = `DrKLO/Telegram`. Tags fetched: `release-9.3.3_3026` (fork base) and `release-11.4.2-5469` (target).
- Reference diffs live in **`patches/`**: `patches/full_fork.diff` (2.4 MB, every fork change vs 9.3.3)
  and `patches/tier1/<File>.diff` (18 per-file hook diffs). **These are the source of truth for
  re-applying hooks — always read the relevant tier1 diff before editing a hook file.**

## How this branch was built (Phase 0–2, DONE)
1. `git checkout -B claude/telegrapher-upstream-plan-ucozoi release-11.4.2-5469` → tree is now pure 11.4.2.
2. Carried forward planning docs (`ANALYSIS.md`, `REBASE_BLUEPRINT.md`, `MICROTASKS.md`) + `patches/`.
3. Copied the **9 `com.evildayz.code.telegraher` Java files** and **18 fork image assets**
   (telegraher_eyez / telegraher_notification drawables ×4 densities, icon_background(_round) ×5) verbatim
   from `origin/graher_9.33`.

So right now: **tree = upstream 11.4.2 (which itself builds) + 9 fork Java files that do NOT yet compile**
(they reference hooks/strings that aren't applied). That is the expected WIP state.

---

## Corrections to the old blueprint (IMPORTANT — verified against 11.4.2)
1. **Do NOT downgrade the toolchain.** 11.4.2 already uses AGP `8.4.2`, `compileSdkVersion 34`,
   `targetSdkVersion 34`, `ndkVersion "21.4.7075529"`, and AGP-8 `namespace`. Keep all of it.
   (MICROTASKS B8 / blueprint Phase 4 saying "SDK 33 / targetSdk 31 / NDK 21" describe the *old* fork —
   ignore.) The existing `Dockerfile` NDK `21.4.7075529` **matches** 11.4.2; only its base image
   `gradle:7.0.2-jdk11` must rise to **JDK 17** (AGP 8.4 requires JDK 17) and it must install `android-34`.
2. **Package ↔ google-services coupling.** Changing `APP_PACKAGE` to `com.evildayz.code.telegraher2`
   (fork identity) will **break the build** while `apply plugin: 'com.google.gms.google-services'`
   (TMessagesProj/build.gradle:208) + a `google-services.json` are present, because the plugin validates
   the applicationId against the json. So the package flip, google-services plugin removal, and
   `google-services.json` deletion **must happen together** (one commit), and Firebase push then needs the
   fork's manual init (the `ApplicationLoader` hook). Until then, keep `APP_PACKAGE=org.telegram.messenger`.
3. **Google removal is a fork VALUE, not a compile requirement.** For the fastest compiling MVP you may
   **keep** upstream's Play/Firebase deps initially (maps/push still work) and defer the Google-strip
   (Phase 5) — stripping `play-services-maps`/`mlkit`/`vision`/`billing` forces you to stub the code that
   uses them (`GoogleMapsProvider`, `BillingController`, `LanguageDetector`, `MrzRecognizer`, …), which is
   real work. Recommended MVP order: get it compiling **with** Google deps, then do the coordinated
   package-flip + Google-strip as its own milestone.

---

## Remaining work (execution order)

### Phase 3 — Build system (do when ready to compile)
Two sub-milestones; keep them separate so CI stays interpretable:
- **3a (compile-neutral, safe now):** `settings.gradle` → keep only `:TMessagesProj` + `:TMessagesProj_App`;
  remove `TMessagesProj_AppHuawei/HockeyApp/Standalone` includes; drop the Huawei maven repo + `agcp`
  classpath from root `build.gradle`. (Upstream still builds without those app flavors.)
- **3b (coordinated fork-identity milestone):** set `APP_PACKAGE=com.evildayz.code.telegraher2` in
  `gradle.properties`; remove `apply plugin: 'com.google.gms.google-services'` + delete `google-services.json`;
  add fork deps `org.osmdroid:osmdroid-android:6.1.14` + `org.apache.commons:commons-lang3:3.12.0`
  (`isoparser`, `gson` already present in 11.4.2). Requires the `ApplicationLoader` push hook to be in place.
- **CI:** change `.github/workflows/Dockerfile_bundle.yml` trigger from `pull_request:{types:[closed]}` on
  `graher_9.33` to `push` on `claude/telegrapher-upstream-plan-ucozoi` so every push compiles; bump the
  `Dockerfile` base image to a JDK-17 gradle image and ensure `platforms;android-34`.

### Phase 4 — Fork hooks (the real work; re-apply BY INTENT, easiest first)
For each file: `cat patches/tier1/<File>.diff`, locate the equivalent spot in the 11.4.2 file, re-apply the
*logic* (do NOT `git apply` — upstream moved too much). Compile via CI after each cluster.
- **4a easy (unblocks evildayz compile):** `BuildVars` (constants `BUILD_VENDOR`/`BUILD_DUROV`, GitHub
  update URL, disable billing) → `UserCell` / `ProfileSearchCell` / `DialogCell` (`starrMark()`) →
  `FilesMigrationService` → `LogoutActivity` → `DrawerUserCell`.
- **4b foundation:** `SharedConfig` (**`activeAccounts`, `thAccounts`, `shadowBannedHM`, device-spoofing
  persistence, `saveAccounts()`** — most other hooks depend on these symbols) → `ApplicationLoader`
  (activeAccounts loops, push-service fix; add osmdroid init only in 3b).
- **4c medium:** `VoIPService` → `NotificationsController` (`getNotificationIcon()`) → `CacheControlActivity`
  → `MediaDataController` (premium-emoji filter).
- **4d hard (biggest diffs):** `LaunchActivity` (settings menu entry) → `ProfileActivity` (ID/DC display,
  copy-id) → `ChatMessageCell` (deletion marks, sticker hiding) → `ChatActivity` (876-line diff: shadowban
  filter, deletion marks, message history, save-to-gallery, restrict-content bypass).
- **4e multi-account loop rewrite** (~113 sites, ~80 files, scriptable): regex
  `for \(int (\w+) = 0; \1 < UserConfig\.MAX_ACCOUNT_COUNT; \1\+\+\)` → `for (int $1 : SharedConfig.activeAccounts)`;
  then reconcile `UserConfig.MAX_ACCOUNT_COUNT`. Do this AFTER `SharedConfig.activeAccounts` exists.
- **4f JNI (4 files):** `jni/tgnet/Defines.h`, `ConnectionsManager.{h,cpp}`, `TgNetWrapper.cpp` — account
  limit + device-spoofing params. Only gates the *native* CI build, not Java compile.

### Phase 5 — Google-strip (fork value; after MVP compiles) — MICROTASKS G1–G5.
### Phase 6 — MVP English `values/strings.xml` entries referenced by the copied Java; then iterate CI green.
### Deferred: translations (ru/ar/fa/de/es/it/ko/nl/pt-BR), cosmetic colors/styles, Tier-3 cleanup,
vendored-lib patches (V1–V5), third-party C++ bugfixes (N5–N7).

---

## Verification
- **Primary:** push → GitHub Actions CI compiles in Docker. Read logs with the GitHub MCP tools
  (`actions_list` / `get_job_logs`). Fix compile errors iteratively.
- **Per-hook sanity (no SDK locally):** use `Grep`/`Read` to confirm each symbol a hook references exists
  in the 11.4.2 tree before relying on CI.
- **On-device smoke test (human, once CI emits an APK):** launch, open Telegraher settings, login, add 2nd
  account, no ads, deletion marks, shadowban, custom notification icon, save-to-gallery.

---

## Progress Tracker  (✅ done · 🟨 in progress · ⬜ todo · ⏭️ deferred)

| Phase | Item | Status |
|---|---|---|
| 0 | unshallow, add upstream, fetch 9.3.3 + 11.4.2 tags | ✅ |
| 0 | export `patches/full_fork.diff` + 18 tier1 diffs | ✅ |
| 1 | reset branch tree onto `release-11.4.2-5469`, carry docs | ✅ |
| 2 | copy 9 `evildayz` Java files | ✅ |
| 2 | copy 18 fork image assets | ✅ |
| 3a | settings.gradle trim + remove Huawei from root build.gradle | ⬜ |
| 3b | package flip + google-services removal + fork deps | ⬜ |
| 3 | CI trigger → push + Docker base → JDK17 | ⬜ |
| 4a | easy hooks (BuildVars, starrMark cells, FilesMigration, Logout, Drawer) | ⬜ |
| 4b | SharedConfig + ApplicationLoader (multi-account foundation) | ⬜ |
| 4c | VoIP, Notifications, CacheControl, MediaData | ⬜ |
| 4d | LaunchActivity, ProfileActivity, ChatMessageCell, ChatActivity | ⬜ |
| 4e | multi-account loop rewrite + UserConfig | ⬜ |
| 4f | JNI 4 files | ⬜ |
| 5 | Google-service code removal/stubs | ⏭️ (after MVP compiles) |
| 6 | MVP English strings + CI green | ⬜ |
| — | translations, cosmetics, cleanup, C++ bugfixes | ⏭️ |
