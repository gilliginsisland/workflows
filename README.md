# Shared workflows

Reusable GitHub Actions workflows for tag releases. No build system or dependency
installation is needed: the ZIP release workflow archives a folder directly.

## ZIP release

Add `.github/workflows/release.yml` to the repository you want to release:

```yaml
name: Release

on:
  push:
    tags:
      - 'v*'

permissions:
  contents: write

jobs:
  release:
    uses: gilliginsisland/workflows/.github/workflows/zip-release.yml@main
    with:
      folder: package
      asset-name: my-project-${{ github.ref_name }}.zip
```

Pushing `v1.2.3` creates a release titled `v1.2.3`, generates release notes,
and attaches `my-project-v1.2.3.zip`. The archive contains the folder's
contents at its root, without an enclosing folder.

### Pre-releases

The tag controls the release type; no workflow setting needs to change:

| Tag | Release type |
| --- | --- |
| `v1.2.3-pre` | Pre-release; not marked as latest stable |
| `v1.2.3` | Stable release |

The full tag is used for the release title and any filename that references
`github.ref_name`, so the example above produces `my-project-v1.2.3-pre.zip`
for a pre-release. The suffix must be exactly `-pre` at the end of the tag.

### Default archive name

Omit `asset-name` to use the folder's basename with `.zip` appended:

```yaml
jobs:
  release:
    uses: gilliginsisland/workflows/.github/workflows/zip-release.yml@main
    with:
      folder: path/to/package
```

The attached asset is `package.zip`. Its filename does not include the tag
unless you include it explicitly in `asset-name`. A trailing slash on `folder`
is supported. Use `folder: .` to package the repository root without Git metadata or `.github` workflow directories.

### Release without assets

Omit `folder` (and the entire `with` block) to create only the release:

```yaml
jobs:
  release:
    uses: gilliginsisland/workflows/.github/workflows/zip-release.yml@main
```

Keep the same tag trigger and `contents: write` permission shown in the full
example. With no folder, checkout and packaging are skipped, and `asset-name`
is ignored. GitHub still provides its standard source-code ZIP/tar downloads;
the workflow uploads no custom assets.

### Inputs and behavior

| Input | Required | Default |
| --- | --- | --- |
| `folder` | No | Empty: release without custom assets |
| `asset-name` | No | `<folder-basename>.zip` when a folder is provided |

- Runs on tag-push events in the calling repository.
- Uses the caller's repository and pushed tag; the tag is also the release title.
- Uses the caller's automatic `GITHUB_TOKEN`; no separate token secret is needed.
- Includes files inside the selected folder, including hidden files, except `.git` metadata
  and `.github` directories (at the root or nested).
- Does not change source files or validate a version stored in project metadata.
- Tags ending in `-pre` create pre-releases and do not become the latest stable
  release. Other tags create normal releases.
- Creates a published release. An existing release for the tag causes failure;
  existing releases/assets are not replaced.
- `@main` follows the current shared workflow. Pin a commit SHA when you want
  a fixed implementation.
