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
tags = ["stylize"]
```

and the generated `plugins/<id>/releases/<version>.toml`. Run `craft-ext catalogue check --only <id>`
yourself first. One maintainer reviews the checklist output and merges. Your release appears in apps
after the next signed build.
