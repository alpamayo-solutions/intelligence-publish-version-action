# Alpamayo Intelligence Publish Version Action

Canonical, reusable CI step for publishing component versions (and SBOM/artifacts) to
Alpamayo Intelligence. This repository is the **single source of truth** for both the
GitHub Actions composite action and the Azure DevOps template — consume it from any
project (internal or external); do not copy the publish logic into your own repo.

It publishes a version to an Intelligence **component feed** and uploads local artifact
files (such as CycloneDX SBOMs) as version artifacts via the Intelligence publish +
artifact-upload API.

## GitHub Actions

```yaml
- name: Publish version to Alpamayo Intelligence
  uses: alpamayo-solutions/intelligence-publish-version-action@v2
  with:
    api-url: https://intelligence.alpamayo-solutions.com
    token: ${{ secrets.ALP_PUBLISHER_TOKEN }}   # <token_key>.<secret>
    project-id: <project-id>
    feed: <feed-id>
    version: ${{ steps.meta.outputs.version }}
    external-ref: ${{ github.ref_name }}
    commit-sha: ${{ github.sha }}
    artifact-files: sboms/*.cdx.json
```

## Azure DevOps

Reference this repository as a `repositories` resource and extend the template:

```yaml
resources:
  repositories:
    - repository: alp-publish
      type: github
      name: alpamayo-solutions/intelligence-publish-version-action
      ref: refs/tags/v2
      endpoint: <your-github-service-connection>

steps:
  - template: azure-devops/publish-version.yml@alp-publish
    parameters:
      apiUrl: https://intelligence.alpamayo-solutions.com
      token: $(ALP_PUBLISHER_TOKEN)            # <token_key>.<secret>
      projectId: <project-id>
      feed: <feed-id>
      version: $(version)
      externalRef: $(Build.SourceBranchName)
      commitSha: $(Build.SourceVersion)
      artifactFiles: sboms/*.cdx.json
```

## Authentication

Authenticate with a **publisher token** created via
`alp admin versioning publisher-token-create` (format `<token_key>.<secret>`),
passed as `token`/`token` and sent as a Bearer credential. The publisher is resolved from
the token and must be allowlisted for the target `feed`. Store the token in a secret source
(a CI secret or a preceding `op read` step) — do not use platform-admin credentials.

## Inputs

GitHub uses kebab-case input names; Azure uses camelCase parameters. Required: `api-url`/`apiUrl`,
`token`, `project-id`/`projectId`, `feed`, `version`.

Optional version metadata: `origin-external-id`/`originExternalId` (defaults to the version),
`source-kind`/`sourceKind` (`git`|`object`|`observed`, default `git`), `external-ref`/`externalRef`,
`commit-sha`/`commitSha`, `object-key`/`objectKey`.

Optional artifacts: `artifact-kind`/`artifactKind` (default `sbom`), `artifact-file`/`artifactFile`
(one path), `artifact-files`/`artifactFiles` (newline-separated paths or globs),
`artifact-label`/`artifactLabel`.

## Migrating from v1

v1 POSTed to the removed resource-scoped `tracked-version` routes and authenticated with a
Keycloak service account. v2 publishes to `/admin/versioning/publish/` with a publisher
token and uploads artifacts to `/projects/<project-id>/versions/<version-id>/artifacts/upload/`.
Replace `client-id`/`client-secret`/`resource-id` with `token` + `feed`.
