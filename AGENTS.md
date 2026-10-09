# Repository Guidelines

## Project Structure & Module Organization

CP Tracker is a static, single-page competitive programming dashboard. `index.html` contains all markup, inline CSS, and vanilla JavaScript, including Codeforces API calls, canvas charts, and persistence. Keep related changes within its existing configuration, storage, synchronization, and rendering sections.

`UI-REVIEW.md` records UI findings; check the current code before treating an item as unresolved. `.planning/ui-reviews/` holds review artifacts and ignores screenshot files. There are no separate source, test, or asset directories, package manifest, or build pipeline. Firebase compatibility SDKs load from a CDN.

## Build, Test, and Development Commands

- `python3 -m http.server 8137`: serve the repository locally; open `http://localhost:8137`.
- `git diff --check`: check tracked changes for whitespace errors before committing.
- `git diff -- index.html`: review application changes.

No compilation, dependency installation, automated test command, or lint command is configured. Network access is required for Firebase SDK loading and live Codeforces requests.

## Coding Style & Naming Conventions

Follow nearby formatting: JavaScript generally uses two-space indentation, semicolons, and single-quoted strings; CSS uses compact rules. Use `camelCase` for functions and variables and uppercase names for configuration constants such as `USERS` and `FIREBASE_CONFIG`. Reuse CSS variables in `:root` for theme colors. Keep DOM IDs and event bindings consistent. Escape external text with `esc()` before inserting it into HTML. Avoid unrelated formatting changes or new tooling without a clear need.

## Testing Guidelines

Validation is currently manual; no testing framework or coverage threshold exists. Check configured member identities, synchronization, user tabs, search, starred filtering, period selection, charts, and comparison views. Reload to verify local persistence. Exercise invalid configured handles, empty histories, API failures, duplicate accepted submissions, and narrow screens. For date logic, check UTC boundaries and streak gaps. Test Firebase synchronization across two sessions when changing that path.

## Commit & Pull Request Guidelines

The repository has no commits yet, so no historical convention exists. Use concise imperative subjects, for example `Fix starred count after toggling`. Keep commits focused. PRs should explain the change, link relevant issues or review findings, list manual checks, and include screenshots for visible changes.

## Security & Configuration

Member handles are fixed in `USERS`; legacy saved handles must not override them. An empty `FIREBASE_CONFIG` selects localStorage mode. Firebase mode uses anonymous authentication and shared `marks` data; verify Firestore rules before enabling shared use. Never commit service-account credentials or private tokens.
