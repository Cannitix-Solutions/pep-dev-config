# pep-dev-config

The **public, secret-free** S9 endpoint-discovery document for PeptiTrack **TEST** builds.

A canonical TEST APK does not bake an API URL. At launch it reads `dev-config.json`
from this repo and uses the `api_base_url` it finds, so the backend can move without
rebuilding or reinstalling the app.

## This file contains no secrets

Only a public backend hostname and metadata. Never add a key, token or connection
string here — the document is world-readable by design.

## Fields

| field | meaning |
|---|---|
| `schema_version` | Must be `1`. Any other value is rejected by the app. |
| `environment` | Must be `test`. Any other value is rejected. |
| `api_base_url` | The backend. Must be https, on the app's host allowlist, and contain `/api/v1`. |
| `min_build` | Builds older than this are refused and shown an update screen. |
| `issued_at` | Informational only. |

## The host allowlist is enforced in the app

`api_base_url` is validated against a compiled-in allowlist, so editing this file
cannot point testers at an arbitrary server. Currently allowed:

- the shared Azure dev backend, by **exact** hostname match
- any `*.trycloudflare.com` quick tunnel, for local development

Adding a new backend host requires a code change in `mobile/lib/core/config/dev_endpoint_discovery.dart`
**and** a new APK — not just an edit here.

## Trust model

The security anchor is the integrity of this document: TLS plus write access to this
repo, combined with the in-app allowlist. **Keep write access restricted and the
default branch protected.** This mechanism is for dev/test only and is never enabled
in a build shipped to real end users.

## Changing the backend

Edit `dev-config.json` on `main`. Testers pick it up on next launch — the app reads
it through the GitHub Contents API, which returns the current file immediately
(unlike the `raw.githubusercontent.com` CDN, which can serve a stale copy for minutes).
