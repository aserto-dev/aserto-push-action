# aserto-push-action

## Aserto Push 

GitHub action to publish policies to the Policy Registry

## Inputs

### `bundle`

**Required** The file path to the policy bundle file. 

Default `build/bundle.tar.gz`

### `release_id`

**Required** The The GitHub release ID. 

No default value

### `policy_registry`

**Required** The policy registry service endpoint. 

Default `https://bundler.prod.aserto.com/v1`

### `push_key`

The composite key containing the registry API key, tenant and policy IDs, needed to publish the policy bundle to the policy registry.

**Required** Unless all three individual overrride values are provided!

No default value

### `tenant_id`

Aserto tenant ID.

No default value

### `policy_id`

Aserto policy ID.

No default value

### `policy_registry_key`

API key for authenticating with the policy registry.

No default value

### `verbose`

Verbose logging of execution of action [true | false]. 

Default `false`

## Outputs

None defined


## Example

```
name: build-release

on:
  workflow_dispatch:
  push:
    tags:
    - '*'

jobs:
  release_policy:
    runs-on: ubuntu-latest
    name: build
    steps:
    
    - uses: actions/checkout@v2

    - name: Build Policy 
      id: aserto-build
      uses: aserto-dev/aserto-build-action@v2
      with:
        source_path: src
        target_path: build
        target_file: bundle.tar.gz
        revision: "$GITHUB_SHA"
        verbose: true

    - name: Release Policy
      id: release
      uses: xresloader/upload-to-github-release@master
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      with:
        file: "build/bundle.tar.gz"
        tags: true
        draft: false
        verbose: true

    - name: Push Policy 
      id: aserto-push
      uses: aserto-dev/aserto-push-action@v2
      env:
        ASERTO_PUSH_KEY: ${{ secrets.ASERTO_PUSH_KEY }}
      with:
        bundle: build/bundle.tar.gz
        release_id: ${{ steps.release.outputs.release_id }}
        verbose: true

```