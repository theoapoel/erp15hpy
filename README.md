### HPY

HPY custom branding and tweaks for ERPNext v15 (navbar theme, translations that rebrand "ERPNext" strings to "HPY").

### What this app changes

- **Colors**: orange navbar gradient (`hpy_theme.bundle.css`, `hpy_web.bundle.css`) — applied automatically to desk and website once installed and built.
- **Text/captions**: translation overrides in `hpy/translations/en.csv` rename a few default strings, e.g. "ERPNext Settings" → "HPY Settings", "ERPNext Integrations" → "HPY Integrations" — applied automatically once installed.
- **Logo**: **not** included in this app. The default HPY logo (`hpy2.png`) and the 4 `Website Settings` fields that reference it (`app_logo`, `brand_html`, `splash_image`, `banner_image`) are per-site data, not app code, so they must be set up manually on every new site (see step 4 below). Missing `splash_image`/`banner_image` is easy to overlook since they don't show up in the desk navbar — check the login/loading splash screen specifically to confirm all 4 are set. Note: `hpy2.png` is the generic HPY logo, safe to reuse as the default for any new site — do not reuse a client-specific logo (e.g. a customer's own brand mark) as the default; set that only on that one client's site.

### Installation (new site)

```bash
cd $PATH_TO_YOUR_BENCH
bench get-app https://github.com/theoapoel/erp15hpy --branch version-15
bench --site $SITE_NAME install-app hpy
bench build --app hpy
bench --site $SITE_NAME clear-cache
```

**Step 4 — logo (manual, per site):**

1. Copy the logo file into the new site's public files folder:
   ```bash
   cp hpy2.png sites/$SITE_NAME/public/files/
   chown frappe:frappe sites/$SITE_NAME/public/files/hpy2.png
   ```
2. Register it as a `File` record and point `Website Settings` at it:
   ```bash
   bench --site $SITE_NAME console
   ```
   ```python
   import frappe
   if not frappe.db.exists("File", {"file_url": "/files/hpy2.png"}):
       frappe.get_doc({"doctype": "File", "file_name": "hpy2.png", "file_url": "/files/hpy2.png", "is_private": 0}).insert(ignore_permissions=True)

   ws = frappe.get_single("Website Settings")
   ws.app_logo = "/files/hpy2.png"
   ws.brand_html = "<img src='/files/hpy2.png' style='max-width: 100px;'>"
   ws.splash_image = "/files/hpy2.png"
   ws.banner_image = "/files/hpy2.png"
   ws.save(ignore_permissions=True)
   frappe.db.commit()
   ```
3. Restart the site's web workers if they were already running before this app was installed (so the newly installed Python module is picked up):
   ```bash
   sudo supervisorctl restart <bench-program-group>-web:*
   ```

### Contributing

This app uses `pre-commit` for code formatting and linting. Please [install pre-commit](https://pre-commit.com/#installation) and enable it for this repository:

```bash
cd apps/hpy
pre-commit install
```

Pre-commit is configured to use the following tools for checking and formatting your code:

- ruff
- eslint
- prettier
- pyupgrade

### License

mit
