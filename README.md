# meros-co/applause-publish-action

Publish a plugin release to Applause from GitHub Actions.

**Full guide:** <https://applause.example/publishing> — claiming a scope, adding
this workflow as a trusted publisher, writing `applause.yaml`, and what happens
after you publish.

```yaml
# .github/workflows/release.yml
on:
  release:
    types: [published]

permissions:
  contents: read
  id-token: write   # trusted publishing: no secret to store

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: meros-co/applause-publish-action@v1
        with:
          package: vellum-audio/driftwood
```

One-time set-up in the Applause publisher portal: add this repository and
`release.yml` as a **trusted publisher** for your scope. The action then proves
who it is with the OIDC token GitHub issues each run — there is no long-lived
secret to leak. (Other CI systems can use a publish token instead: pass it as
`token`.)

## `applause.yaml`

An ordinary Applause manifest at the root of your repository, except that file
entries may leave out `sha256` and `size`, and URLs may use `{version}` and
`{tag}`:

```yaml
files:
  - url: https://github.com/vellum-audio/driftwood/releases/download/{tag}/driftwood-{version}-win-x64.zip
    type: archive
    architectures: [x64]
    systems: [{ type: win }]
    contains: [clap, vst3]
```

The action downloads every file, computes its hash and size from the real bytes,
fills in `date` and — if you left `changes` out — the release notes, validates
the result, and submits it. A declared hash or size that
does not match the asset fails the run.

Applause then checks each file and publishes the version once the checks pass.
With `wait: true` (the default) the job ends when the version is published, and
fails with the reason if a check fails.

## Inputs

| Input | Default | |
|---|---|---|
| `package` | — | `org/package` |
| `manifest` | `applause.yaml` | Template path |
| `version` | tag without `v` | Must be strict semver |
| `registry` | `https://applause.example` | The Applause site |
| `token` | — | Publish token; omit for trusted publishing |
| `wait` | `true` | Wait for publish or failure, for up to 15 minutes |
| `dry-run` | `false` | Resolve, validate, print; do not submit |
