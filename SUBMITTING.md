# Submitting a plugin

## 1. Register as a vendor (once)

Add `vendors/<your-github-login>.toml`:

```toml
login = "ink-studio"
github_id = 12345678        # https://api.github.com/users/<login> → "id"
name = "Ink Studio"
tier = "community"          # maintainers change this to "verified" or "official"
url = "https://ink.example"
```

To become **verified**, add `verified_how = "github-org-domain ink.example"` (your GitHub organisation
shows a verified domain) or `verified_how = "dns-txt ink.example"` with a TXT record
`craft-plugins-vendor=<login>` on that domain. A maintainer checks it and sets `tier = "verified"`.

## 2. Build and release

```sh
craft-ext new effect my-effect --id io.github.<you>.my-effect
cd my-effect && craft-ext build && craft-ext check --store && craft-ext test
craft-ext pack --release-record --tag v1.0.0 --commit <sha> \
  --url https://github.com/<you>/my-effect/releases/download/v1.0.0/io.github.<you>.my-effect-1.0.0.craftplugin
```

Upload the `.craftplugin` file to a GitHub Release with that tag. Never replace an uploaded asset: a
new file needs a new version.

## 3. Open a pull request

Add `plugins/<id>/listing.toml` (first time only):

```toml
id = "io.github.you.my-effect"
vendor = "you"
repo = "https://github.com/you/my-effect"
repo_id = 87654321          # https://api.github.com/repos/<owner>/<repo> → "id"
tags = ["stylize"]          # at most 10
summary = "One line people see in search"          # optional, at most 140 characters
categories = ["effects"]    # optional, at most 2 from the per-app list in craft-extensions docs
homepage = "https://you.example/my-effect"         # optional
icon = "media/icon.png"     # optional: square PNG in plugins/<id>/media/, at most 256 KiB
screenshots = ["media/before-after.png"]           # optional: up to 6 PNGs, each at most 2 MiB
signer_workflow = "you/my-effect/.github/workflows/release.yml"   # optional: earns "Built from source"
```

Add the generated `plugins/<id>/releases/<version>.toml`, with an optional `changelog` (at most 4 KiB).

Before opening the pull request, run both checks yourself:

```sh
craft-ext catalogue check --only <id>
craft-ext catalogue check-submission --repo . --id <id>
```

`check-submission` runs every automated check and prints one verdict:

- **Schema and limits.**
- **Lookalike names** of listed plugins.
- **Wider permissions** than your previous release.
- **Source repository and an immutable GitHub release.**
- **Build attestation**, when you name a `signer_workflow` and publish with
  `actions/attest-build-provenance`.

A check that cannot run fails; it is never skipped. Wider permissions and lookalike names mark the
submission `needs-review`. Everything else is merged without manual review. Your release appears in
apps after the next signed build.

Maintainers curate `featured.toml` (`ids = [...]`, at most 24). Featuring is never requested in a
pull request.
