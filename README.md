# Bayside Teams App — Packaging & Installation

This README covers **only** how to package and publish our tiny Teams app to the tenant app catalog, and how to install/upgrade it for users.

---

## 1) Prerequisites

- **Manifest ID (App ID):** A GUID we choose in `manifest.json` → `"id"`. Keep it stable across versions.
- **Entra App (client) ID:** Goes in `manifest.json` → `webApplicationInfo.id`. This ties the Teams app to our AAD app used by Rock.
- **Personal tab:** `staticTabs` with `"scopes": ["personal"]` so the app is installable/visible.
- **Icons:**
  - `color.png` — **192×192** full‑color.
  - `outline.png` — **32×32**, white on transparent.
- **Versioning:** Bump `"version"` on every change (e.g., `1.0.3`).

> Tip: If the app has **no** personal tab/bot/extension (no scope), it won’t appear in Setup policies and the client won’t show an **Add** button.

---

## 2) Package layout

The app package must contain **exactly three files at the ZIP root** (no parent folder):

```
manifest.json
color.png         # 192x192
outline.png       # 32x32 (white on transparent)
```

**Do not** place these inside a folder and then zip the folder. Teams expects the files at the top level of the ZIP.

---

## 3) Build the ZIP

### macOS
1. Put `manifest.json`, `color.png`, `outline.png` in the same directory.
2. Select the three files → right‑click → **Compress 3 Items**.
3. Ensure the ZIP root shows the three files (open the ZIP to verify).

### Windows
1. Put the three files together.
2. Select them → right‑click → **Send to → Compressed (zipped) folder**.
3. Open the ZIP to verify the three files are at the root.

**Verify (optional):**
```
unzip -l your-app.zip
```
Output should list `manifest.json`, `color.png`, `outline.png` with no leading folder path.

---

## 4) Publish to the tenant app catalog

1. Go to **Teams admin center** → **Teams apps → Manage apps**.
2. Click **Upload new app**, select your ZIP.
3. Open the app’s page and confirm:
   - **Status:** Allowed.
   - **Scope:** **Personal** (comes from your `staticTabs` with `"scopes": ["personal"]`).

If status is Blocked or custom apps are disabled, update org policy and allow the app.

---

## 5) Install for users

### A) Via **App setup policy** (recommended)
1. In Teams admin center → **Teams apps → Setup policies**.
2. Edit your policy (or create one).
3. Under **Installed apps**, **Add apps** → search for your app → **Add** → **Save**.
4. Optionally pin it in **Pinned apps**.

### B) Per‑user install (Admin Center)
- On the app’s page, click **Install** (if available) → choose users → **Install**.

### C) Per‑user install (Graph, optional)
- `POST /v1.0/users/{userId}/teamwork/installedApps`
- Body:
  ```json
  {
    "teamsApp@odata.bind": "https://graph.microsoft.com/v1.0/appCatalogs/teamsApps('{CATALOG_TEAMS_APP_ID}')"
  }
  ```
- The **catalog Teams app ID** is the server‑generated ID assigned when the package is uploaded (different from the manifest `"id"`).

---

## 6) Upgrading to a new version

1. Bump `"version"` in `manifest.json`.
2. Re‑zip the three files at the root.
3. In **Teams admin center → Manage apps → Your app**, click **Upload file** and select the new ZIP. The page should now show the new **Published version**.

**Update users’ installed copy:**  
- **Manual:** Uninstall → Add again.  
- **Graph (recommended):**
  1. Find the user’s installation:  
     `GET /v1.0/users/{userId}/teamwork/installedApps?$expand=teamsApp,teamsAppDefinition`
  2. Identify the entry where `teamsApp/externalId == "<manifest.json id>"`.
  3. Upgrade:  
     `POST /v1.0/users/{userId}/teamwork/installedApps/{installationId}/upgrade` (204 on success).

---

## 7) Troubleshooting

- **“There’s no manifest file in your app package.”**  
  The ZIP contains a parent folder. Re‑zip so `manifest.json`, `color.png`, `outline.png` are at the **root**.

- **“This app is not available.”**  
  Usually policy/scope: ensure the app is **Allowed** and has a **Personal** scope (e.g., a `staticTabs` entry). Then add it via **Setup policy**.

- **App doesn’t appear in Setup policy search.**  
  It has no installable scope. Confirm `staticTabs` with `"scopes": ["personal"]` (or add a bot/extension). Re‑upload and retry.

- **Users don’t see the new version.**  
  After publishing a new version, **upgrade** installed apps (Graph or reinstall). A simple sign‑out/in of the Teams client can also help refresh.

---

### Quick checklist before uploading

- `manifest.json` passes schema checks; `"version"` bumped.
- `webApplicationInfo.id` is set to the correct **Entra App ID**.
- `staticTabs` includes a personal tab; `"scopes": ["personal"]`.
- Icons at the correct sizes; files named exactly `color.png`, `outline.png`.
- ZIP contains only the three files at the **root**.
