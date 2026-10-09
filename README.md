# couchdb-openapi

OpenAPI 3.0 description of the Apache CouchDB 3.x HTTP API. This repo is the
**source of truth** for every generated CouchDB SDK.

```
couchdb-openapi  --(tag vX.Y.Z → repository_dispatch)-->  couchdb-sdk-generator  --(PR)-->  couchdb-python
```

## Workflow

1. Edit `openapi.yaml`, open a PR. CI runs Spectral + Redocly lint and an
   `oasdiff` breaking-change check against `main`.
2. Merge, then tag: `git tag v0.2.0 && git push --tags`.
3. `release.yml` sends a `spec-released` event to `couchdb-sdk-generator`,
   which regenerates SDKs and opens PRs in each SDK repo.

Versioning: semver. A breaking spec change (per oasdiff) requires a major bump.

## Lint locally

```sh
npx @stoplight/spectral-cli@6 lint openapi.yaml
```

## Secrets

- `SDK_BOT_TOKEN`: fine-grained PAT (or GitHub App token) with `contents:write`
  on `couchdb-sdk-generator`.

## Coverage

Server, `_session` auth, databases, documents, `_bulk_docs`, `_all_docs`,
Mango `_find`/`_index`, design documents and views, attachments, `_changes`
(normal/longpoll), `_replicate`, `_security`, and partitioned queries.

Not yet covered: continuous/eventsource `_changes` (streaming), `_scheduler`,
`_node`/config, `_purge`, `_revs_diff`, `_explain`, search.

Every change here should come with a scenario in
`couchdb-sdk-generator/conformance/specs` so all SDKs are tested against it.
See the generator README for spec patterns that generate broken code.
