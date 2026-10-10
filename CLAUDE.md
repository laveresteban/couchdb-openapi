# CLAUDE.md — couchdb-openapi

OpenAPI 3.0.3 description of the CouchDB 3.x HTTP API (`openapi.yaml`).
Source of truth for every SDK. Current version: **0.5.0**.

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

## Fixes

1. **`putAttachment` only accepts `application/octet-stream`.** CouchDB stores
   the request's `Content-Type` as the attachment's `content_type`, so
   generated clients that follow the spec save every file as octet-stream.
   Python and Node work around it with header overrides. Use `*/*` (or a
   wildcard media type) for the request body and the GET response.
2. **`ReplicationRequest.source`/`target` are object-only**, but the
   description says "Database URL, or an object". CouchDB accepts a plain URL
   string. Make them `oneOf: [string, object]`, or fix the description.
3. **`Db` path pattern `^[a-z][a-z0-9_$()+/-]*$` rejects system databases**
   (`_users`, `_replicator`, `_global_changes`). Python's generator doesn't
   enforce it today, but stricter generators will. Allow a leading `_` for
   those names or drop the pattern.
4. **`feed=continuous` documents an `application/json` `ChangesResult`
   response**, but the body is newline-delimited JSON. Generated clients wait
   for the full body and then fail to parse it. Every SDK already bypasses the
   generated call for continuous. Document it as `application/x-ndjson` (or a
   separate operation) so generated code doesn't pretend to support it.
5. **Missing error responses.** No operation declares `403`, and several miss
   `400`/`401` (e.g. `getDocument`, `putDocument`, `postView`). Add a shared
   set so typed errors are consistent across SDKs.
6. **`ViewQuery` is missing common fields**: `key`, `inclusive_end`,
   `startkey_docid`/`endkey_docid`, `stable`, `update_seq`, `conflicts`,
   `attachments`. `key` especially, since users reach for it first.
7. **`DesignDocument` only models `views`/`options`.** Add `filters`,
   `validate_doc_update`, `updates`, `autoupdate`, and per-view `options`
   (partitioned design docs need `options.partitioned`).
8. **`ReplicationResult` is thin.** Add `no_changes`, `replication_id_version`
   and type the `history` entries (docs_read, docs_written, doc_write_failures, ...).
9. `oasdiff/oasdiff-action/breaking@main` is unpinned. Pin a version tag.
10. Paths `/{db}/{docid}/{attname}` and `/{db}/_design/{ddoc}` /
    `/{db}/_local/{docid}` overlap structurally. Some routers and mock
    servers pick the wrong one. Document that `docid` never starts with `_`
    here, or move attachments of design docs to their own path.

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
