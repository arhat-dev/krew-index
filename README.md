# krew-index

[![CI](https://github.com/arhat-dev/krew-index/workflows/CI/badge.svg)](https://github.com/arhat-dev/krew-index/actions?query=workflow%3ACI)

__NOTE:__ Your SHOULD NEVER edit yaml files in plugins dir manually.

## Workflow

### Add a new plugin `kubectl-FOO`

- Create a matrix file at [templates/matrix-kubectl-FOO.yaml](./templates)
  - With all kernel/arch pairs defined for the plugin
- Create a plugin config at [templates/kubectl-FOO.yaml](./templates)
  - Add a `workflow:run` task for the plugin (NOTE: there should be a rendering suffix to the workflow definition)
    - with name `index-kubectl-FOO`
    - with env values
      - `APP=kubectl-FOO`
      - `VERSION=v{MAJOR}.{MINOR}.{FIX}`
      - `SHORT_DESCRIPTION` (mandatory field for a plugin manifest)
      - `DESCRIPTION`
      - `CAVEATES`
    - with jobs to produce `build/archive/${APP}.${KERNEL}.${ARCH}.tar.gz`
    - with hooks blew

      ```yaml
      hooks:
        before:
        - shell: mkdir -p build/manifests

        after:matrix:success:
        - shell@file?str|template?str: templates/shell/write-blob-entry.tmpl
          env:
          - name: IN_ARCHIVE_FILES
            value@transform?str:
              value@template?str: |-
                # your list of files to be extracted from archive
                - from: something # in archive path
                  to: . # where it should be extracted to, relative path to plugin installation dir
              ops:
              # this template operation is required by templates/shell/write-blob-entry.tmpl
              - template: |-
                  {{ VALUE | indent 2 }}

        after:success:
        - shell@file?str|template?str: templates/shell/write-manifest.tmpl
      ```

- Include the new plugin in the [github workflows build matrix](./.github/workflows/ci.yaml)

## LICENSE

```txt
Copyright 2021 The arhat.dev Authors.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
