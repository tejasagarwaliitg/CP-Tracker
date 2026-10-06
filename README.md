# CP Tracker

A dark, head-to-head Codeforces tracker for two teammates. It compares weekly weighted scores, shows solved problems and rating progress, and lets each person star problems for later. Handles and stars stay in this browser by default; shared synchronization requires Firebase configuration.

## Run locally

No build step or package installation is needed. From the repository root, run:

```sh
python3 -m http.server 8137
```

Then open <http://localhost:8137>.

## Free hosting with Cloudflare Pages

1. Push this repository to your GitHub account. The repository can remain private.
2. In Cloudflare, open **Workers & Pages**, choose **Create application → Pages → Connect to Git**, and authorize the GitHub repository.
3. Select **None** for the framework preset, leave the build command empty, and set the build output directory to `/` (the repository root). Deploy.

Cloudflare Pages redeploys after pushes to the production branch and provides preview deployments for other branches. This app has no server-side build or backend requirement when used in local-storage mode.

## Configuration

`index.html` contains the app and its configuration. Codeforces data is fetched from the public API. With `FIREBASE_CONFIG` left empty, data is stored in the current browser only. Configure Firebase and appropriate Firestore security rules before expecting data to sync between devices or users. Never put private credentials or service-account keys in this static app.

See [AGENTS.md](AGENTS.md) for contribution and validation guidance.
