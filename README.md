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

Server (`_up`, `_uuids`, `_all_dbs`, `_dbs_info`, `_db_updates`,
`_active_tasks`), auth (basic, `_session` cookie, JWT bearer), databases,
documents (`HEAD`, `If-None-Match`/`ETag`), `_bulk_docs`, `_all_docs` (GET and
POST), `_design_docs`, `_local_docs`, Mango `_find`/`_index`/`_explain`, design
documents and views (GET and POST), attachments (any content type), `_changes`
(normal, longpoll and continuous; `style=all_docs`; `_doc_ids`/`_selector`
filters), replication primitives (`_revs_diff`, `_bulk_get`, `_local` docs,
`new_edits:false`, `_revisions`), `_replicate` (URL or endpoint object),
`_scheduler/jobs` and `/docs`, `_security`, `_compact`, `_view_cleanup`,
`_purge`, and partitioned queries.

Not covered: eventsource `_changes`, `_node`/config, search, proxy auth,
multipart `_bulk_get`, and `open_revs` on GET document (it changes the
response shape; use `_bulk_get`).

Every change here should come with a scenario in
`couchdb-sdk-generator/conformance/specs` so all SDKs are tested against it.
See the generator README for spec patterns that generate broken code.

## License

Apache License 2.0. See [LICENSE](LICENSE) and [NOTICE](NOTICE).
