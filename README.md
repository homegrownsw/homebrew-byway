# Byway — Homebrew tap

The official Homebrew tap for the **[Byway](https://byway.place)** desktop CLI.

```sh
brew install homegrownsw/byway/byway
```

That taps this repository and installs `byway` in one step. If you'd rather tap
first:

```sh
brew tap homegrownsw/byway
brew install byway
```

Upgrade with `brew upgrade byway`, and remove the tap with
`brew untap homegrownsw/byway`.

## What `byway` is

`byway` is a read-only desktop client for a **Byway** account. It enrols as an
end-to-end-encrypted device, mirrors your decrypted location history into a
local encrypted SQLite database, keeps it fresh, and answers everyday questions
about it — in the terminal and over MCP for LLM clients.

```sh
byway login          # enrol this machine
byway update         # sync, and say what's new
byway visits         # today's visits
byway where          # where you are now
byway timeline 2026-07-20
```

`byway help` lists the rest. Full documentation lives with the product.

**macOS only for now** (Apple Silicon and Intel — the published binary is
universal). Linux is a tracked follow-up.

## What's in this repository

- **`Formula/byway.rb`** — the Homebrew formula.
- **[Releases](../../releases)** — the signed, notarized `byway` binaries the
  formula installs, one per version, tagged `byway-v<version>`.

Both are written by the release tooling in Byway's own repository, which is
private. `Formula/byway.rb` is **generated — don't edit it by hand**; the next
release overwrites it. Bug reports and questions belong with the product, not
here: this repository is a distribution channel, and a pull request against the
formula would be replaced by the next release.

### Why the binaries are released here rather than alongside the source

Byway's source repository is private, and GitHub Releases inherit their
repository's visibility — so a release cut there could only be downloaded by
someone with source access, which `brew install` cannot assume. Publishing the
archives here keeps the source private and the binaries public.

## Versions

Year-based: `2026.1`, `2026.2`, …, then `2027.1`. `byway version` reports the
version you have installed.

## Verifying a download

Every published binary is signed with a Developer ID certificate and notarized
by Apple, so Gatekeeper accepts it. To check a copy yourself:

```sh
codesign --verify --strict --verbose=2 "$(brew --prefix)/bin/byway"
spctl --assess --type exec -vv "$(brew --prefix)/bin/byway"
```

The notarization ticket is not stapled to the binary, because a ticket can only
be attached to a bundle, `.dmg` or `.pkg` and `byway` is a bare executable.
Gatekeeper checks it online instead. `spctl` therefore needs a network
connection to give a verdict.

## Licence

The `byway` binary is proprietary software, © Homegrown Software. It is
redistributable through this tap; the source is not published. The formula in
this repository exists to install it.
