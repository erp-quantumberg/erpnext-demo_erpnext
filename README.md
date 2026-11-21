# Demo ERPNext

Demo ERPNext is a Frappe application that provides a public Vue 3 landing page
for an ERPNext manufacturing, project management, inventory, and accounting
demo. It is intended to help clients explore the ERPNext experience.

The feature descriptions on the landing page describe capabilities available
from ERPNext and its configured modules. This app itself supplies the landing
page and Frappe integration; it does not implement a separate manufacturing or
accounting engine.

## Features

- Guest-facing landing page served at the site root (`/`).
- Landing page sections for manufacturing, projects, accounting, benefits,
  team roles, and calls to action.
- Login entry point to the Frappe/ERPNext site.
- Vue 3 frontend built with Vite and Frappe UI.
- Frontend assets served by Frappe after the app is built.
- Guest users see the landing page; signed-in users retain Frappe's normal
  home-page and Desk routing.

The landing page highlights ERPNext capabilities such as:

- Production planning, bills of materials, work orders, and quality checks.
- Inventory and stock tracking.
- Projects, tasks, timesheets, and project costing.
- Ledgers, invoicing, payments, budgeting, and reports.

Availability of these business features depends on the ERPNext app, site
configuration, user permissions, and installed modules.

## Requirements

Install this app into a working Frappe Bench. The bench and site require the
normal Frappe services and infrastructure, including a supported database and
Redis.

### App dependencies

- **Frappe Framework (`frappe`) — required.** Install this app in a bench with
  Frappe and on a site where Frappe is installed.
- **ERPNext (`erpnext`) — optional.** Install ERPNext on the site to use the
  ERP business capabilities described above. The landing page itself does not
  require ERPNext.

This app does not declare `required_apps` in `demo_erpnext/hooks.py`, so Bench
will not install ERPNext automatically.

### Other requirements

- **Python:** `>=3.14`, as declared in `pyproject.toml`. The CI workflow uses
  Python 3.14.
- **Frappe Framework:** installed and managed by Bench; it is not vendored by
  this app.
- **Node.js:** required to build the frontend. CI uses Node.js 24.
- **Frontend dependencies:** Vue 3, Vue Router, Frappe UI, Vite, Tailwind CSS,
  and their build tooling are declared in `frontend/package.json`.
- **Database and Redis:** use versions supported by the Frappe/ERPNext release
  installed in your bench. The CI workflow uses MariaDB 11.8.

`pyproject.toml` does not declare additional app-specific Python runtime
packages. Bench installs the Frappe and site dependencies.

## Installation

The following assumes you already have a Frappe Bench and a site. Run these
commands from the bench directory:

```bash
bench get-app https://github.com/erp-quantumberg/erpnext-demo_erpnext.git --branch main
bench --site your-site install-app demo_erpnext
bench build
```

If you want the ERPNext capabilities highlighted on the landing page, install
ERPNext on the site before or after this app:

```bash
bench --site your-site install-app erpnext
```

The ERPNext app must be present in the bench first. If it is not, fetch the
ERPNext branch compatible with the Frappe version in your bench (for example,
`bench get-app erpnext --branch version-16` for a Frappe 16 bench). Skip the
command if ERPNext is already installed on the site. Replace `your-site` with
the name of your Frappe site. For a new environment, create and configure the
site using the standard `bench new-site` workflow before installing apps.

## Setup

### Build the frontend for Frappe

The frontend is in `frontend/`; the app's Python and Frappe integration are in
`demo_erpnext/`. Install frontend packages and build from the frontend
directory:

```bash
cd apps/demo_erpnext/frontend
npm ci
npm run build
cd ../..
bench build
```

The Vite build writes assets to `demo_erpnext/public/frontend/` and updates
`demo_erpnext/www/index.html` so the Frappe homepage references the generated
assets. Rebuild after changing frontend source files. `npm install` is for
`frontend/`; the repository root is a Frappe app, not the frontend npm project.

### Run the frontend development server

From `apps/demo_erpnext/frontend`, start Vite:

```bash
npm install
npm run dev
```

Open `http://your-site:8080` using a hostname that resolves to your local site.
The Vite configuration proxies requests to the Frappe web server using the
bench's `sites/common_site_config.json`; make sure the bench is running.

## Troubleshooting

- **`bench get-app` or `install-app` cannot find the app:** run `get-app` from
  the bench directory, verify the app is present under `apps/`, and check the
  site name with `bench list-sites`.
- **ERPNext features are missing:** confirm ERPNext is installed on the site
  with `bench --site your-site list-apps`, then check the signed-in user's
  roles and permissions.
- **The landing page does not appear at `/`:** it is for Guest users. Signed-in
  users intentionally follow Frappe's normal home-page routing. Test as Guest
  or sign out.
- **The page or its assets look stale or return 404:** run `npm run build`
  from `frontend/`, then `bench build`; clear the site's cache with
  `bench --site your-site clear-cache` and hard-refresh the browser.
- **Vite cannot read `common_site_config.json`:** run the frontend from the
  app directory inside an initialized bench, not from a standalone clone.
- **The dev server cannot reach Frappe:** start the bench, check the site
  hostname and web-server port in the bench configuration, and retry
  `http://your-site:8080`.
- **A frontend API request reports a CSRF error:** use the CSRF guidance below
  for local development only. Do not disable CSRF protection on a shared,
  staging, or production site.

## Configuration notes

- The Frappe hook `get_website_user_home_page` routes Guests to the app's
  `www/index.html`; signed-in users are not redirected.
- The frontend is served from `/` in production and its built assets are
  published under `/assets/demo_erpnext/frontend/`.
- The website template injects a session CSRF token for frontend requests.
  Keep this behavior enabled in deployed environments.
- The existing frontend starter documentation mentions setting
  `"ignore_csrf": 1` in a site's `site_config.json` if the Vite development
  server produces CSRF errors. That setting disables CSRF checks: use it only
  on an isolated local development site, and remove it afterward. Never use it
  in production.
- The app metadata declares the license as `mit`; its Python package metadata
  and version are maintained in `pyproject.toml`.

## Contributing

1. Create a branch for your change.
2. Install the frontend dependencies from `frontend/` with `npm ci`.
3. From the repository root, install and run the repository hooks:

   ```bash
   pre-commit install
   pre-commit run --all-files
   ```

4. For frontend changes, run `npm run build` from `frontend/`.
5. For server-side changes, run the app tests from the bench:

   ```bash
   bench --site your-site run-tests --app demo_erpnext
   ```

Keep changes focused, include tests for new behavior where appropriate, and
describe any setup or configuration changes in your pull request.

The repository's CI workflow installs the app on a test site and runs its
server tests. The linter workflow runs the configured pre-commit checks and
Semgrep rules on pull requests, and checks Python dependencies with
`pip-audit`.

## License

The app declares the **MIT License** in `demo_erpnext/hooks.py`. See the
[MIT License](https://opensource.org/license/mit/) for the license terms.
