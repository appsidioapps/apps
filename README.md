# Apps by Appsidio

The public catalog page used by the **More from Appsidio** row in each app.

## Live URL

`https://appsidioapps.github.io/apps/`

The apps fetch `https://appsidioapps.github.io/apps/index.html` and cache the
result locally, so the page keeps working offline after the first successful
load. Every app also ships a bundled copy in `assets/more_apps/index.html`.

## Files

| File | Purpose |
|---|---|
| `apps.json` | Source of truth for the catalog: title, package, tagline, category and icon path. |
| `template.html` | Page shell. `{{CARDS}}`, `{{YEAR}}` and `{{DEVELOPER_URL}}` are replaced by the build script. |
| `index.html` | Generated, self-contained page. Do not edit by hand. |
| `.nojekyll` | Tells GitHub Pages to serve the files as-is. |

## Update the app list

1. Edit `apps.json`.
2. From the workspace root, run:

   ```powershell
   powershell -File scripts/build_more_apps_site.ps1
   ```

   The script:

   - resizes each launcher icon to 128x128,
   - embeds the icons as base64 PNGs so the page has no external requests,
   - regenerates `index.html`,
   - copies the same page into every app's `assets/more_apps/index.html`.

3. Preview `index.html` in a browser at phone width.
4. Commit and push the site repo.
5. Rebuild any app if you want the new bundled fallback in the APK. The live
   page updates immediately for users with a connection.

Icons are taken from each app's
`android/app/src/main/res/mipmap-xxxhdpi/ic_launcher.png`. If a new app has no
`ic_launcher.png`, add one first or point `icon` at another square PNG.

The script uses `ffmpeg` when available for small paletted icons and falls back
to `System.Drawing` otherwise. Both paths preserve transparency.

## Publish on GitHub Pages (one time)

1. Create a public GitHub repository named `apps` under the `appsidioapps`
   account.
2. Push this folder to the repository root:

   ```powershell
   cd more-apps-site
   git init
   git add .
   git commit -m "Add Appsidio apps page"
   git branch -M main
   git remote add origin https://github.com/appsidioapps/apps.git
   git push -u origin main
   ```

3. In the repository, open **Settings -> Pages**.
4. Set **Source** to **Deploy from a branch**, branch `main`, folder `/ (root)`,
   then save.
5. Wait for the deployment and verify:

   ```powershell
   curl.exe -I https://appsidioapps.github.io/apps/
   ```

The first deployment usually takes one to two minutes.

## Adding the row to another app

Copy the module from an existing app (for example `wallpaper_app`):

- `lib/services/more_apps_cache.dart`
- `lib/screens/more_apps_screen.dart`
- `test/more_apps_cache_test.dart`
- `assets/more_apps/index.html`

Then:

1. Add `webview_flutter`, `http`, `shared_preferences` and `url_launcher` if
   the app does not already have them.
2. Register `assets/more_apps/` in `pubspec.yaml`.
3. Add the **More from Appsidio** row to the app's Settings screen and pass the
   app's package name to `MoreAppsScreen`.
4. Add `android.permission.INTERNET` to the main Android manifest if missing.
5. Run `flutter pub get`, `flutter analyze` and `flutter test`.
6. If the app is published on Google Play, add it to `apps.json` and rerun the
   build script so it appears on the page.

Keep the module copies identical except for the import paths and the package
name so future fixes are easy to apply everywhere.

## Page contract

The generated page contains this marker:

```html
<!-- appsidio-apps-page:v1 -->
```

The apps validate the marker before caching downloaded HTML. Keep the marker in
`template.html` if you restyle the page.

The page must stay self-contained (inline CSS, inline images, no JavaScript, no
external fonts) so a single downloaded HTML file works offline.

Keep the generated page under roughly 500 KB. The apps store the downloaded
HTML as one `SharedPreferences` string, which is passed over the Flutter
platform channel when saved. The current page is about 150 KB with 10 apps. If
the catalog grows well past that, switch `MoreAppsCache` to a file in the app
support directory.

## Changing the URL

If the GitHub Pages URL changes, update it in every app:

```text
lib/**/more_apps_cache.dart  ->  MoreAppsCache.pageUrl
```

Search the workspace for `appsidioapps.github.io` to find all copies.
