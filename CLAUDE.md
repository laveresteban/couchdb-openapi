# CLAUDE.md — couchdb-openapi

OpenAPI 3.0.3 description of the CouchDB 3.x HTTP API (`openapi.yaml`).
Source of truth for every SDK. Current version: **0.6.0**.

Pipeline: tag `vX.Y.Z` here → `release.yml` sends `spec-released` →
couchdb-sdk-generator regenerates SDKs and opens PRs. See the generator's
CLAUDE.md for the cross-repo picture.

## Commands

```sh
npx @stoplight/spectral-cli@6 lint openapi.yaml --fail-severity=warn
npx @redocly/cli@1 lint openapi.yaml
```

CI also runs an `oasdiff` breaking-change check against the PR base.

## Rules

- Bump `info.version` in every change (semver; oasdiff breaking → major).
  Generated SDK versions come straight from this field.
- Every new endpoint/param gets a scenario in
  `couchdb-sdk-generator/conformance/specs`.
- Don't use the patterns that break generators (see generator README):
  enum inside `additionalProperties`, untyped `{}` where null is meaningful,
  open `Document` for Kotlin.

## Fixes still open

Done in 0.6.0: system db names allowed by the `Db` pattern, `401`/`403`
(and other missing 4xx) responses, more `ViewQuery` fields (`key`, ...),
`DesignDocument` filters/updates/validation, typed `ReplicationResult`
history, continuous `_changes` documented as NDJSON, `oasdiff` action pinned.

1. **`putAttachment` only accepts `application/octet-stream`.** CouchDB stores
   the request's `Content-Type` as the attachment's `content_type`. Python and
   Node send the real type with a header override. Switching the spec to
   `*/*` is a breaking change per oasdiff, so leave it for 1.0.
2. **`ReplicationRequest.source`/`target` are object-only.** CouchDB also
   accepts a plain URL string. `oneOf: [string, object]` generates awkward
   wrapper types in Python and TypeScript, so it's left as object-only for now.
3. **Continuous `_changes` is still typed as one `ChangesResult`.** It's
   documented as NDJSON, but generated clients still can't stream it.
4. Paths `/{db}/{docid}/{attname}` and `/{db}/_design/{ddoc}` /
   `/{db}/_local/{docid}` overlap structurally. Some routers and mock
   servers pick the wrong one.

## Features to add

Ordered by what the SDKs need first:

1. `GET /{db}/_all_docs` and `GET .../_view/{view}` (only POST exists).
2. `HEAD /{db}/{docid}` (cheap existence/ETag check; SDK `has()` does a full GET today).
3. `_design_docs`, `_local_docs`, `POST /_dbs_info`.
4. `_purge`, `_explain`, `_compact`, `_view_cleanup`.
5. `_scheduler/jobs`, `_scheduler/docs`, `_active_tasks`, `_db_updates`.
6. `_users` helpers (create user doc in `_users`), JWT / proxy auth schemes.
7. `open_revs` on GET document (multipart/mixed or `Accept: application/json`).
8. `_bulk_get` multipart response and `_bulk_docs` `417` on validation failure.
9. ETag / `If-None-Match` support on doc and attachment GET.
