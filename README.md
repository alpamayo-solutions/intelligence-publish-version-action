# Alpamayo Intelligence Publish Version Action

Public GitHub Action for publishing component versions (and SBOM/artifacts) to
Alpamayo Intelligence.

The action publishes a version to an Intelligence **component feed** and uploads
local artifact files (such as CycloneDX SBOMs) as version artifacts. It is a thin
wrapper around the Intelligence publish + artifact-upload API; repository-specific
behavior belongs in the caller workflow.

## Usage

```yaml
- name: Publish version to Alpamayo Intelligence
  uses: alpamayo-solutions/intelligence-publish-version-action@v2
  with:
    api-url: https://intelligence.alpamayo-solutions.com
    token: ${{ steps.intelligence-token.outputs.publisher-token }}  # <token_key>.<secret>
    project-id: 01KMTEFDGRY87RPNHPBCN5FAVN
    feed: 01KSZK7J7HNYY6ZNJ901XDMBW3
    version: ${{ steps.meta.outputs.version }}
    external-ref: ${{ github.ref_name }}
    commit-sha: ${{ github.sha }}
    artifact-files: sboms/*.cdx.json
```

## Authentication

Authenticate with a **publisher token** created via
`alp admin versioning publisher-token-create` (format `<token_key>.<secret>`),
passed as `token` and sent as a Bearer credential. The publisher is resolved from
the token, and must be allowlisted for the target `feed`. Store the token in a
secret source (a GitHub Actions secret or a preceding `op read` step) — do not use
platform-admin credentials.

## Inputs

Required:

| Input | Description |
| --- | --- |
| `api-url` | Base URL of the Intelligence API. |
| `token` | Publisher token `<token_key>.<secret>`. |
| `project-id` | Project ID that owns the component (used to upload artifacts). |
| `feed` | VersionFeed ID to publish to (the publisher must be allowlisted for it). |
| `version` | Version label to publish. |

Version metadata (optional):

| Input | Default | Description |
| --- | --- | --- |
| `origin-external-id` | (version) | Stable external identity for the version. |
| `source-kind` | `git` | `git`, `object`, or `observed`. |
| `external-ref` | | Source reference, such as a tag or branch. |
| `commit-sha` | | Resolved immutable commit SHA. |
| `object-key` | | Object-storage key for object-kind primary content. |

Artifacts (optional):

| Input | Default | Description |
| --- | --- | --- |
| `artifact-kind` | `sbom` | Kind recorded for uploaded artifact files. |
| `artifact-file` | | One local artifact file to upload. |
| `artifact-files` | | Newline-separated local paths or glob patterns to upload. |
| `artifact-label` | | Human-readable artifact label. |

## Migrating from v1

v1 posted to the removed resource-scoped `tracked-version` routes and authenticated
with a Keycloak service account. v2 publishes to `/admin/versioning/publish/` with a
publisher token and uploads artifacts to
`/projects/<project-id>/versions/<version-id>/artifacts/upload/`. Replace
`client-id`/`client-secret`/`resource-id` with `token` + `feed`.
