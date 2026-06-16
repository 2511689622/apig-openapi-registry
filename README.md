# APIG OpenAPI Registry

This repository is the central registry for Huawei Cloud APIG imports.

It stores reviewed registry copies of service OpenAPI files and APIG import target configuration. Tooling lives in a separate repository:

- Registry repo: `2511689622/apig-openapi-registry`
- Tooling repo: `2511689622/apig-registry-tooling`

## Repository layout

```text
.
├── catalog.yaml
├── services/
│   └── software-package-server/
│       └── communities/
│           └── openeuler/
│               ├── openapi.yaml
│               └── apig.yaml
└── .github/
    └── workflows/
        ├── ci.yml
        ├── validate.yml
        ├── sync-openapi.yml
        └── import-apig.yml
```

## YAML ownership

### Service repository

The service repository owns the source OpenAPI file, for example:

```text
software-package-server/docs/openapi.openeuler.yaml
```

Service owners maintain API paths, schemas, auth, backend definitions, and Huawei APIG OpenAPI extensions.

### Registry repository

This repository stores:

```text
services/<service>/communities/<community>/openapi.yaml
services/<service>/communities/<community>/apig.yaml
```

- `openapi.yaml` is a reviewed registry copy synced from the service repository. Do not edit it manually except for emergency fixes.
- `apig.yaml` is maintained in this registry repository and describes the Huawei Cloud APIG import target.

## catalog.yaml

`catalog.yaml` links source files in service repositories to reviewed registry copies in this repository.

```yaml
apiVersion: infra.example.com/v1
kind: ApigCatalog
metadata:
  name: infra-apig-catalog

services:
  - name: software-package-server
    owner: package-team
    enabled: true
    communities:
      - name: openeuler
        owner: openeuler-infra-team
        enabled: true
        source:
          repo: https://github.com/opensourceways/software-package-server
          # Production OpenAPI source branch for this community.
          branch: main
          openapi: docs/openapi.openeuler.yaml
        registry:
          openapi: services/software-package-server/communities/openeuler/openapi.yaml
          apig: services/software-package-server/communities/openeuler/apig.yaml
```

This registry imports production APIs only. Different communities can use different production source branches through `source.branch`.

## Workflow

### 1. Source OpenAPI changes in the service repository

The service repository sends `repository_dispatch` to this registry repository when its source OpenAPI file changes.

Payload example:

```json
{
  "service": "software-package-server",
  "community": "openeuler",
  "repo": "opensourceways/software-package-server",
  "ref": "<commit-sha>",
  "openapi": "docs/openapi.openeuler.yaml"
}
```

### 2. sync-openapi.yml creates a registry PR

`.github/workflows/sync-openapi.yml` checks out:

1. this registry repository,
2. `2511689622/apig-registry-tooling`,
3. the source service repository at the provided ref.

It copies the source OpenAPI into the registry path, validates Huawei APIG OpenAPI format, and creates a PR. It does not import APIG.

### 3. Humans review and merge

Review the registry PR diff before merge. This is the audit boundary.

### 4. import-apig.yml imports after merge

Run `.github/workflows/import-apig.yml` manually with:

```text
service=software-package-server
community=openeuler
dry_run=true|false
```

The import workflow reads only registry files from this repository and uses scripts from `2511689622/apig-registry-tooling`.

## Required GitHub configuration

### Secrets

```text
HUAWEICLOUD_SDK_AK
HUAWEICLOUD_SDK_SK
```

If the tooling repository is private:

```text
TOOLING_REPO_TOKEN
```

If the source service repository is private:

```text
SOURCE_REPO_TOKEN
```

### Variables

```text
HUAWEICLOUD_PROJECT_ID
APIG_INSTANCE_ID
OPENEULER_SOFTWARE_PACKAGE_SERVER_APIG_GROUP_ID
```
