# Official document-template catalog

`catalog/v1/catalog.json` is the source of truth for the official template
catalog shown in the app's type picker ("官方模板" section). The app downloads
it as a static file — there is no backend.

## Contract

- Wire format: `RemoteCatalog` / `RemoteDocTemplate` / `RemotePageSpec` /
  `RemoteFieldSpec` (`Keptu/Core/Networking/RemoteDocTemplate.swift`).
- Templates are **not localized**: `displayName`, every page `label`, and every
  field `label` are written in the document's own language (居民身份证,
  Hong Kong Identity Card, 正面 / 背面, …).
- **Pages (multi-page documents).** A document is an ordered list of pages,
  each a physical side/page with its own image and fields. A template describes
  those pages with either:
  - `pages`: an array of `{ "label": "正面", "fields": [ … ] }` — preferred for
    two-sided or multi-page documents (ID cards, driver's licences, bank cards);
    or
  - `fields`: a flat field array — shorthand for a single-page document, treated
    by the client as one unlabeled page (`resolvedPages`).

  Provide exactly one of the two per template. Every page must carry at least
  one field, and a template with more than one page must label each page. Field
  labels stay unique enough across pages that anchors (`name` / `number` /
  `expiry`) appear at most once per document.
- `category` must be a `DocCategory` raw value; `kind` must be a
  `DocField.Kind` raw value (`text` / `number` / `expiry` / `issueDate` /
  `name` / `issuedLocation` / `issuedBy`). Unknown categories are dropped by
  the client, so a typo here silently hides a template — the fixture tests
  catch this before it ships.
- `regions`: ISO alpha-2 codes where the template surfaces; `null` = worldwide.
- Schema is only ever changed additively with optional keys; older clients
  must keep decoding newer catalogs (the `pages` key was added in `version` 2
  alongside the existing `fields` shorthand).

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
