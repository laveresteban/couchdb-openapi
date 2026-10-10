# CLAUDE.md — couchdb-openapi

OpenAPI 3.0.3 description of the CouchDB 3.x HTTP API (`openapi.yaml`).
Source of truth for every SDK. Current version: **0.7.0**.

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

Done in 0.6.0/0.7.0: system db names in the `Db` pattern, 4xx responses,
`ViewQuery`/`DesignDocument`/`ReplicationResult` fields, attachments accept
any content type (`*/*` added next to `application/octet-stream`, so not
breaking), replication `source`/`target` take a URL string or an object,
attachment paths can't be mistaken for `_design`/`_local` (`AttDocId`
pattern), `oasdiff` pinned.

1. **Continuous `_changes` is typed as one `ChangesResult`.** CouchDB serves
   it as `application/json` even though the body is NDJSON, and one operation
   can't have two response shapes for the same media type. It's documented on
   the operation; SDKs read the raw stream.
2. **`ReplicationEndpoint` is untyped.** `oneOf: [string, object]` is the
   accurate schema, but typescript-fetch 7.10 generates broken code for it
   (imports a `./string` model).
3. **JWT (`bearerAuth`) has no conformance scenario**: it needs
   `jwt_authentication` and keys configured server-wide.

## Features to add

- `_bulk_get` multipart response (streams attachments instead of base64).
- `open_revs` on GET document, as its own path or with `Accept: application/json`.
- `_node/{node}/_config`, `_membership`, `_cluster_setup`, search indexes.
- Proxy authentication headers (`X-Auth-CouchDB-*`).
