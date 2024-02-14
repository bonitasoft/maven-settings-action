# maven-settings-action
Composite Github Action to setup Maven settings with Bonitasoft Artifactory repository.
Intend for internal usage only.

## Input

| Name          | Description                                             |
| ------------- |---------------------------------------------------------|
| `KSM_CONFIG`  | The Keeper Secret Manager configuration                 |

## Example Workflow File

```yaml
name: Build
on: [pull_request]

jobs:
    maven-build:
        runs-on: ubuntu-latest
        steps:
          - name: Setup Maven Settings
            uses: bonitasoft/maven-settings-action@TAGNAME
            with:
              KSM_CONFIG: ${{ secrets.KSM_CONFIG }}
```
