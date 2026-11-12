# Branding this rankescalate install

Three layers, all without touching rankescalate's source code:

| Layer | Where | Changes |
|---|---|---|
| 1. Settings | Web UI → Settings → General | Brand name, logo, favicon, sender name |
| 2. Custom CSS/JS | Web UI → Settings → Appearance | Admin screen logo, tab title, favicon, colours |
| 3. Template files | `./static/` folder (this project) | Public pages + system email HTML/CSS, logo & favicon files |

Do the steps in order: **3 → 1 → 2** (files first, then the settings that point at them).

---

## Step 3 — Template files (`./static/`)

This folder is a copy of listmonk **v7.0.0-beta.2**'s built-in templates on the `v7-test` branch (**v6.2.0** on `main`). rankescalate loads it
instead of its built-in copy (`--static-dir` in `docker-compose.yml`).

### Replace the logo and favicon

| Replace this file | With | Used on |
|---|---|---|
| `static/public/static/logo.png` | your logo (PNG, ~300px wide, transparent background) | admin top bar, public pages, system emails |
| `static/public/static/favicon.png` | your icon (PNG, 64×64) | browser tab on public pages + admin |

Keep the **same file names**. Restart after changing: `docker compose restart app`.

### Files you can edit

| File | What it is |
|---|---|
| `static/public/static/style.css` | Look of all public pages. Brand colour: replace `#0055d4` (rankescalate blue) |
| `static/public/templates/index.html` | Header (logo) + footer of all public pages. Footer shows `© <Site name>` |
| `static/public/templates/subscription-form.html` | Public signup page |
| `static/public/templates/subscription.html` | Manage-subscription / unsubscribe page |
| `static/public/templates/optin.html` | Opt-in confirmation page |
| `static/public/templates/archive.html` | Public newsletter archive |
| `static/public/templates/login.html` | Admin login page |
| `static/public/templates/message.html` | Generic message page ("You are unsubscribed", errors) |
| `static/email-templates/base.html` | Header/footer of **system** emails (opt-in confirmation, etc.) |
| `static/email-templates/default.tpl` | Starting campaign template, copied into the DB on **first** install only |

Already done: "Powered by rankescalate" removed from the public footer and the email footers.
In the email templates the footer is now an HTML comment. Put your own footer text there.

**Rule:** don't remove the `{{ ... }}` tags. They insert data (links, names, translations).
Change the HTML and CSS around them.

---

## Step 1 — Settings → General

After `docker compose up -d`, log in at http://localhost:9000, then **Settings → General**:

| Field | Value |
|---|---|
| **Site name** | `Rankescalate` |
| **Root URL** | `http://localhost:9000` for now. Later: `https://news.yourdomain.com` |
| **Logo URL** | `http://localhost:9000/public/static/logo.png` (use `<Root URL>/public/static/logo.png` once you're on a real domain. Emails need a public address to show the logo) |
| **Favicon URL** | leave empty (uses `static/public/static/favicon.png`) |
| **Default `from` email** | `Rankescalate <news@yourdomain.com>` |

Click **Save**.

Also: **Campaigns → Templates**. Edit the default template to put in your logo, colours and footer.

---

## Step 2 — Settings → Appearance

### Admin tab → Custom CSS (v7, new admin design)

Delete everything in the box first: the old v6 CSS colours every sidebar link in v7. Then paste:

```css
/* Rankescalate logo in the admin sidebar (hides the listmonk logo) */
.sidebar-branding img { display: none !important; }
.sidebar-branding a {
  display: block;
  width: 180px;
  height: 40px;
  background: url('/public/static/logo.png') no-repeat left center / contain !important;
}

/* Small icon in the top bar on phones */
.mobile-branding a img { visibility: hidden; }
.mobile-branding a {
  background: url('/public/static/favicon.png') no-repeat center / contain !important;
}

/* Rankescalate brand colour: (light theme, dark theme) */
* {
  --primary: light-dark(#e84a1f, #ff7a52);
  --ring: light-dark(#e84a1f, #ff7a52);
}
```

The admin tab title now uses **Site name** automatically. For the tab icon, set
**Settings → General → Favicon URL** to `<Root URL>/public/static/favicon.png`.

### Admin tab → Custom JavaScript (optional in v7)

Not needed in v7, because the Site name and Favicon URL settings cover it. The v6 script below still works if it's left in place.
Change `Your Brand` on the second line:

```js
(function () {
  var BRAND = 'Rankescalate';

  // Browser tab title: "Dashboard / listmonk" -> "Dashboard / Your Brand"
  // (keep 'listmonk' here: it's the text the app puts in the title, which we replace)
  function fixTitle() {
    if (document.title.indexOf('listmonk') !== -1) {
      document.title = document.title.replace(/listmonk/g, BRAND);
    }
  }

  function init() {
    // Browser tab icon -> your favicon
    var icon = document.querySelector('link[rel="icon"]');
    if (icon) icon.href = '/public/static/favicon.png';

    var title = document.querySelector('title');
    if (title) new MutationObserver(fixTitle).observe(title, { childList: true, characterData: true, subtree: true });
    fixTitle();
  }

  if (document.readyState === 'loading') document.addEventListener('DOMContentLoaded', init);
  else init();
})();
```

Click **Save**, then hard-refresh the browser (Ctrl+F5).

### Public tab

Not needed: `static/public/static/style.css` already covers public pages. Use this tab
only for small CSS tweaks you don't want to put in the file.

---

## What can't be changed without editing source code

- Text and layout inside the admin screen (menus, page layouts, built-in help texts)

Changing those means building your own Docker image from modified source (AGPL-3.0 applies).

---

## Upgrading rankescalate later (important)

`./static` overrides rankescalate's built-in templates, so it must match the app version.
If you upgrade with old templates, pages can break.

1. Back up the database first.
2. Download the new version's source and compare its `static/` folder with yours.
3. Copy your changes (logo, footer, CSS) into the new files.
4. Change the image tag in `docker-compose.yml`, then `docker compose up -d`.
