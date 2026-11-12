# Maintainer guide (one maintainer allows each listing)

No bot and no CI. One maintainer handles each PR:

1. `craft-ext catalogue check --repo . --only <id>` (online). It must pass with no errors. It checks
   schemas, vendor and namespace, OSI licence, public source and matching licence, tag → commit,
   every asset's size and SHA-256, and that the archive manifest equals the record. It also flags paid
   wording.
2. Quick look (the parts automation cannot prove):
   - Is the source plausibly what was built? Glance at the repo at that tag.
   - Is it free to use, with no paywall, trial, licence key or "pro" upsell?
   - Do the permissions make sense for what it says it does? For native plugins, take a closer look:
     they have full access to the user's computer.
   - Does the vendor impersonate someone?
3. Merge, then publish:
   ```sh
   craft-ext catalogue build --repo .
   craft-ext catalogue sign --repo . --key ~/.config/craft/catalogue.key   # key never committed
   git add dist && git commit -m "Publish catalogue serial N" && git push
   ```
   Re-run build + sign at least every 3 weeks: signed metadata expires after 30 days.

## Reports and the kill list

- Reports arrive as issues from the **Report plugin** form. Any report labelled `security` or
  `malware`, or three reports from different accounts, gets the `needs-review` label.
- After triage, add a `[[revocation]]` to `revocations.toml`, then build + sign:
  - `advisory`: a warning in apps;
  - `block-new`: no new installs, enables, updates or rollbacks to matching releases;
  - `disable`: the kill list. Matching releases stop in every app on the next refresh (at start and
    every 6 hours), including local installs with the same id or digest.
- Account takeover: `block-new` (or `disable` for shipped malware) on the vendor, then a note in the
  issue, then a recovery release is checked like a first listing.

## Keys

`craft-ext catalogue keygen --out <file>` creates a key and prints the public half for
`craft_ext_core::catalogue::TRUSTED_KEYS`. Apps hold two key slots so a replacement can ship before the
old key is retired. If a key leaks, ship an app update with only the new key, then re-sign.
