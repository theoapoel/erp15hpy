### HPY

HPY custom branding and tweaks for ERPNext v15 (navbar theme, translations that rebrand "ERPNext" strings to "HPY").

### What this app changes

- **Colors**: orange navbar gradient (`hpy_theme.bundle.css`, `hpy_web.bundle.css`) — applied automatically to desk and website once installed and built.
- **Text/captions**: translation overrides in `hpy/translations/en.csv` rename a few default strings, e.g. "ERPNext Settings" → "HPY Settings", "ERPNext Integrations" → "HPY Integrations" — applied automatically once installed.
- **Logo**: **not** included in this app. The logo images (`hpy2.png` for app logo, `lobar.png` for navbar brand) and the `Website Settings` (`app_logo`, `brand_html`) that reference them are per-site data, not app code, so they must be set up manually on every new site (see step 4 below).

### Installation (new site)

```bash
cd $PATH_TO_YOUR_BENCH
bench get-app https://github.com/theoapoel/erp15hpy --branch version-15
bench --site $SITE_NAME install-app hpy
bench build --app hpy
bench --site $SITE_NAME clear-cache
```

**Step 4 — logo (manual, per site):**

1. Copy the logo files into the new site's public files folder:
   ```bash
   cp hpy2.png lobar.png sites/$SITE_NAME/public/files/
   chown frappe:frappe sites/$SITE_NAME/public/files/hpy2.png sites/$SITE_NAME/public/files/lobar.png
   ```
2. Register them as `File` records and point `Website Settings` at them:
   ```bash
   bench --site $SITE_NAME console
   ```
   ```python
   import frappe
   for fname, path in [("hpy2.png", "/files/hpy2.png"), ("lobar.png", "/files/lobar.png")]:
       if not frappe.db.exists("File", {"file_url": path}):
           frappe.get_doc({"doctype": "File", "file_name": fname, "file_url": path, "is_private": 0}).insert(ignore_permissions=True)

   ws = frappe.get_single("Website Settings")
   ws.app_logo = "/files/hpy2.png"
   ws.brand_html = "<img src='/files/lobar.png' style='max-width: 100px;'>"
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
