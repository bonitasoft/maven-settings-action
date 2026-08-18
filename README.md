# Maven Settings Action

Composite GitHub Action to set up Maven settings with Bonitasoft Artifactory repository.
Intended for internal usage only.

## Input

| Name                   | Description                             |
| ---------------------- | --------------------------------------- |
| `keeper-secret-config` | The Keeper Secret Manager configuration |

## Usage

```yaml
jobs:
  maven-build:
    runs-on: ubuntu-latest
    steps:
      - name: Setup Maven Settings
        uses: bonitasoft/maven-settings-action@TAGNAME
        with:
          keeper-secret-config: ${{ secrets.KSM_CONFIG }}
```

## Avoid overriding settings.xml

Place this action after any other step that writes `~/.m2/settings.xml` — such as
`actions/setup-java` or an `actions/cache` on `~/.m2` — and before the Maven command,
otherwise the generated `settings.xml` gets overridden.

```yaml
jobs:
  maven-build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v7

      - name: Setup Java
        uses: actions/setup-java@v5
        with:
          distribution: temurin
          java-version: 21

      - name: Cache Maven
        uses: actions/cache@v6
        with:
          # ~/.m2/repository, never ~/.m2, which would also cache settings.xml
          path: ~/.m2/repository
          key: ${{ runner.os }}-maven

      - name: Setup Maven Settings
        uses: bonitasoft/maven-settings-action@TAGNAME
        with:
          keeper-secret-config: ${{ secrets.KSM_CONFIG }}

      - name: Build
        run: mvn -B deploy
```

## Generated settings.xml

[`settings.xml`](settings.xml) is copied verbatim to `~/.m2/settings.xml`, overwriting any
existing file.

It contains **no credential**: servers reference `${env.*}` expressions that Maven resolves
when it reads the file. The action exports those variables to the job environment via
Keeper, so Maven must run **in the same job** as this action — the variables are not
available to other jobs, nor inside a container started manually with `docker run`.

`GITHUB_ACTOR` and `GITHUB_TOKEN` back the `github` server. `GITHUB_ACTOR` is already set
in every job by GitHub Actions, but `GITHUB_TOKEN` is not exported automatically — set it
yourself if your build resolves from or publishes to GitHub Packages.

A `401` from Artifactory usually means the variables did not reach the Maven process,
leaving the `${env.*}` expressions unresolved: check that Maven runs in the same job as
this action. It can also be a stale credential (e.g. a rotated JFROG token in Keeper).
Or, on a `401` from `maven.pkg.github.com`, a missing `GITHUB_TOKEN` in the job.

The `gpg.keyname` property is set to the fingerprint of the imported GPG key; see the
linked [`settings.xml`](settings.xml) for the servers, repositories and plugin
repositories it defines.
