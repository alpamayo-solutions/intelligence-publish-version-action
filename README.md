# Alpamayo Intelligence Publish Version Action

Public GitHub Action for publishing repository release/build versions to
Alpamayo Intelligence.

The action records a version for an Intelligence project resource and can attach
local artifact files, such as CycloneDX SBOMs, as version artifacts. It is a
thin wrapper around the Intelligence ingest API; repository-specific behavior
belongs in the caller workflow.

## Usage

```yaml
- name: Publish version to Alpamayo Intelligence
  uses: alpamayo-solutions/intelligence-publish-version-action@v1
  with:
    api-url: https://intelligence.alpamayo-solutions.com
    client-id: intelligence-version-publisher
    client-secret: ${{ secrets.INTELLIGENCE_VERSION_PUBLISHER_CLIENT_SECRET }}
    project-id: 01KMTEFDGRY87RPNHPBCN5FAVN
    resource-id: 01KMWCTX9WQYA63M53PQ0TX5ZA
    version: ${{ steps.meta.outputs.version }}
    source-ref-kind: git_branch
    source-ref: ${{ github.ref_name }}
    resolved-ref: ${{ github.sha }}
    artifact-files: sboms/*.cdx.json
```

## Inputs

Required inputs:

| Input | Description |
| --- | --- |
| `api-url` | Base URL of the Intelligence API. |
| `project-id` | Intelligence project ID that owns the resource. |
| `resource-id` | Intelligence project resource receiving the version. |
| `version` | Version label to publish. |

Authentication inputs:

| Input | Description |
| --- | --- |
| `token` | Existing Intelligence access token. |
| `client-id` | Keycloak service-account client ID used when `token` is not provided. |
| `client-secret` | Keycloak service-account client secret used when `token` is not provided. |
| `token-url` | Explicit Keycloak token URL. |
| `keycloak-url` | Keycloak base URL used with `realm` to build a token URL. |
| `realm` | Keycloak realm, default `intelligence`. |

Version metadata inputs:

| Input | Default | Description |
| --- | --- | --- |
| `source-ref-kind` | `git_tag` | Source reference kind, such as `git_tag`, `git_branch`, `git_commit`, or `external`. |
| `source-ref` | | Source reference that produced the version. |
| `resolved-ref` | | Immutable resolved reference, usually a full commit SHA. |

Artifact inputs:

| Input | Default | Description |
| --- | --- | --- |
| `artifact-kind` | `sbom` | Artifact kind recorded in Intelligence. |
| `artifact-file` | | One local artifact file to upload. |
| `artifact-files` | | Newline-separated local artifact paths or glob patterns to upload. |
| `artifact-url` | | External artifact URL to record instead of uploading a local file. |
| `artifact-label` | | Human-readable artifact label. Local uploads default to the file basename. |

## Requirements

The calling workflow needs:

- A registered Intelligence project resource for the repository or deployable.
- A scoped service account allowed to write tracked versions for that resource.
- A secret source for the service-account credential, such as GitHub Actions
  secrets or a preceding `op read` step.

Do not use platform-admin service accounts for CI version publishing.
