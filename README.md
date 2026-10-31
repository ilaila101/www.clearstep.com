# ClearStep Finance — iOS & Android app

ClearStep site is packaged as a native app with [Capacitor 8](https://capacitorjs.com). One codebase (`www/`) produces both the iOS and the Android project.

## What's in the box

| Path | What it is |
|---|---|
| `www/index.html` | The site, with the changes listed below |
| `www/native.js` | Native extras: share-sheet backups, lesson reminders, Android back button, system bar theme |
| `capacitor.config.json` | App id, name, splash settings |
| `scripts/copy-libs.js` | Copies Chart.js + Font Awesome into `www/vendor` so the app works offline |
| `www/Logo.jpeg` | The logo (shown in the app header) |
| `assets/` | App icon + splash art built from logo |
| `docs/privacy-policy.html` | Privacy policy page to host publicly (required by both stores) |

### Changes to site

- **Works offline:** Chart.js and Font Awesome are bundled instead of loaded from CDNs. All data already stored on the device (`localStorage`), so nothing else changes.
- **Phone layout:** notch / home-bar safe areas, a swipeable tab bar, 2-column summary cards, a compact calendar, the credit-card view stacks on narrow screens, 16px inputs (stops iOS zooming on focus).
- **Save / Load backup:** browsers can't "download" inside an app, so **Save** now opens the phone's share sheet (Save to Files, Drive, email, AirDrop, Telegram…). **Load** is unchanged. On desktop browsers Save still downloads a file.
- **Lesson reminders (new, app only):** Student Record → *Lesson Reminders*. Builds weekly notifications from each active student's lesson days and time (at lesson time / 15 / 30 / 60 min before). They update automatically when you edit students.
- **Android back button:** closes an open dialog → returns to Dashboard → exits.
- **Two bug fixes:**
  - New entries defaulted to the *UTC* date, i.e. **yesterday** between local midnight and 05:00 in Yekaterinburg. They now use local date.
  - `switchView()` relied on `event.currentTarget`, which broke the highlighted tab when it was triggered from code (editing a student, tapping a summary card).

## First-time setup

**Requirements:** Node 22+. For Android: Android Studio 2025.2.1 or newer. For iOS: a Mac with **Xcode 26** or newer (see "No Mac?" below).

1. **App id.** It is set to `io.github.ilaila101.clearstep` in `capacitor.config.json`, which is based on the GitHub username and needs no domain. It is permanent once you publish. Only change it to `com.clearstep.finance` (or similar) if you can prove you own the domain clearstep.com, for example from domain registrar account. A GitHub repository name being available does not show that.
2. *(Optional)* The icon uses the climbing figure and star from your logo, and the splash screen uses the full logo. The source logo is only 421×370 pixels, so the icon is enlarged and slightly soft. For a crisper result, replace the PNGs in `assets/` with art made from a larger copy of the logo and re-run `npm run assets`. `icon-only.png` and `icon-background.png` are 1024×1024, `icon-foreground.png` is 1024×1024 with transparency and the artwork kept inside the centre two-thirds, splash images are 2732×2732.
3. Run:

```bash
npm install
npx cap add android
npx cap add ios        # Mac only
npm run assets         # generates every icon/splash size from assets/
npm run android        # syncs, then opens Android Studio  (or: npm run ios)
```

Press Run in Android Studio / Xcode to launch on an emulator, simulator or phone.

After any change to `www/`, run `npm run sync` to copy it into the native projects.

## Test checklist (on a real phone before you publish)

I could not run the native build in my environment (the sandbox blocks the npm registry), so please check these on a device:

- [ ] Header and bottom of the page aren't hidden behind the notch / gesture bar
- [ ] Add an income + a card expense; charts render with Wi-Fi off
- [ ] **Save** opens the share sheet; the saved file loads back with **Load**
- [ ] Add a student with lesson days, turn on Lesson Reminders, accept the permission prompt, and confirm a notification arrives (set a lesson a few minutes ahead)
- [ ] Android: back button behaves as described; on Android 12+ reminders may arrive a few minutes late unless "Alarms & reminders" is allowed for the app
- [ ] Rotate / switch light–dark theme

## Publishing

**Both stores require a privacy policy URL.** A ready-to-host draft is in `docs/privacy-policy.html` (contact: Dick Pere Ilaye, dickilaye@gmail.com). It is a draft, not legal advice, so read it and adjust it to your situation. For a free public URL:

1. In the public repository `www.clearstep.com` (github.com/ilaila101/www.clearstep.com), use *Add file → Upload files* to upload `privacy-policy.html` into a folder named `docs`. Keep the app source code out of this public repository unless you want it public (see the licence note below).
2. Repository → *Settings → Pages* → Source: *Deploy from a branch*, branch `main`, folder `/docs`.
3. After a minute the page should be at `https://ilaila101.github.io/www.clearstep.com/privacy-policy.html`. Open it on your phone, then paste that URL into App Store Connect and Play Console. If it shows a 404, check the exact address on the Pages settings screen.

**Store privacy forms.** The app sends none of your entries anywhere, so the likely answers are "Data Not Collected" on Apple's App Privacy form and "no data collected or shared" on Google Play's Data safety form. The one third-party connection is the Unsplash photos (Unsplash sees the device's IP address). If you want that answer to be completely clear-cut, replace those photos with local images or colour gradients so the app makes no network requests at all. Check each form's current wording yourself, since the stores change them.

**Google Play** (one-time $25 developer account)
1. Android Studio → *Build → Generate Signed App Bundle* → create a keystore and **back it up** (losing it means you can't update the app).
2. Upload the `.aab` in Play Console. Google Play requires new apps and updates to target Android 16 / API 36 from Aug 31, 2026, so check that `targetSdkVersion` in `android/variables.gradle` is 36 or higher before you build.
3. New personal developer accounts may have to run a closed test before going to production; Play Console shows the current rules.

**Apple App Store** ($99/year developer account)
1. Open `ios/App/App.xcworkspace` (or `.xcodeproj`, depending on how Capacitor generated it) in Xcode 26+, set your Team and Bundle Identifier.
2. *Product → Archive → Distribute App → App Store Connect*.
3. Complete the listing in App Store Connect (screenshots, description, privacy policy URL, App Privacy answers).
4. Apple sometimes rejects apps that are only a website in a frame (guideline 4.2). This one has offline use, share-sheet backups and scheduled reminders, which helps, but there's no guarantee, so describe those features in the review notes.

**No Mac?** iOS builds need macOS. You can use a cloud Mac build service (for example Codemagic, or GitHub Actions `macos` runners) to build and upload from your repository. You still need the paid Apple Developer account.

## Licences

- **Your code:** `LICENSE` is set to "all rights reserved" (proprietary). To go open source instead, replace it with the MIT text and change `"license"` in `package.json`.
- **Third-party libraries:** `npm install` / `npm run sync` runs `scripts/gen-notices.js`, which builds `www/THIRD_PARTY_NOTICES.txt` from the licence files inside the installed packages. The app's footer links to it ("Open-source licenses"). Read the output once: the script warns if a package has no licence file, and I could not confirm the licence of the Capacitor plugin packages myself.
- **Keep your app source private** if it is proprietary. Your GitHub repository `www.clearstep.com` is public, so only put the privacy policy page there.

## Good to know

- **Your data lives on the phone.** Uninstalling the app deletes it, and moving to a new phone doesn't carry it over. Use **Save** now and then and keep the file somewhere safe.
- The decorative header photos load from Unsplash and are simply dark panels when offline. To make the app fully self-contained, download them into `www/img/` and point the URLs at them.
- iOS allows at most 64 pending local notifications; the app schedules up to 60 (about 20 students × 3 lessons a week).
- The project is on Capacitor 8 (`^8.0.0`). When a newer major comes out, `npx cap migrate` upgrades the project.
