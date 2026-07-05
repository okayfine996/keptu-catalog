# Official document-template catalog

`catalog/v1/catalog.json` is the source of truth for the official template
catalog shown in the app's type picker ("官方模板" section). The app downloads
it as a static file — there is no backend.

## Contract

- Wire format: `RemoteCatalog` / `RemoteDocTemplate` / `RemoteFieldSpec`
  (`Keptu/Core/Networking/RemoteDocTemplate.swift`).
- Templates are **not localized**: `displayName` and every field `label` are
  written in the document's own language (居民身份证, Hong Kong Identity Card, …).
- `category` must be a `DocCategory` raw value; `kind` must be a
  `DocField.Kind` raw value (`text` / `number` / `expiry` / `issueDate` /
  `name` / `issuedLocation` / `issuedBy`). Unknown categories are dropped by
  the client, so a typo here silently hides a template — the fixture tests
  catch this before it ships.
- `regions`: ISO alpha-2 codes where the template surfaces; `null` = worldwide.
- Schema is only ever changed additively with optional keys; older clients
  must keep decoding newer catalogs.

## Publishing

The app fetches `{host}/v1/catalog.json` from the hosts in
`TemplateCatalogService.defaultHosts` (jsDelivr over a public GitHub repo,
with a CN-friendly mirror and raw.githubusercontent.com as fallbacks).

1. Edit `catalog/v1/catalog.json` and **bump `version`** (clients only apply
   a catalog whose version is greater than their cached one).
2. Run `./init.sh` — `OfficialCatalogFixtureTests` validates the exact bytes
   that will be published.
3. Copy `catalog/v1/` into the public catalog repo referenced by
   `defaultHosts` and push to `main`.

jsDelivr caches branch refs for up to ~12 hours; catalog updates tolerate that
staleness. If stronger mainland-China availability is ever needed, prepend an
阿里云 OSS (or any static-host) URL to `defaultHosts` — no other code change
is required.
