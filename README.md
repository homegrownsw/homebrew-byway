# Byway CLI

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
about it — in the terminal and [over MCP](#asking-an-llm-about-your-location-history--mcp)
for LLM assistants.

```sh
byway login          # enrol this machine
byway update         # sync, and say what's new
byway visits         # today's visits
byway where          # where you are now
byway timeline 2026-07-20
```

`byway help` lists the rest. Full documentation lives with the product.

**macOS only** (Apple Silicon and Intel — the published binary is universal).

## Asking an LLM about your location history — MCP

`byway mcp` serves a [Model Context Protocol](https://modelcontextprotocol.io)
tool server over stdio, giving an assistant like Claude structured access to your
history. **The queries run against the local mirror on your machine** — no
bearer token is plumbed anywhere and nothing about a question leaves the
computer.

### Claude Code

```sh
claude mcp add byway -- byway mcp
```

### Claude Desktop, and other MCP clients

Add it to the client's MCP server config — `byway` needs no arguments and no
credentials, because it reuses the keyring `byway login` already put in your
Keychain:

```json
{
  "mcpServers": {
    "byway": { "command": "byway", "args": ["mcp"] }
  }
}
```

`byway login` must have run at least once first; the server reads the same local
database as the rest of the CLI.

### What it does while it runs

It pulls once on start and then follows your account for the life of the
session, so a long conversation reflects what your phone has synced in the
meantime, and derived data is rebuilt after any pull that absorbed something.
Useful flags:

| flag | effect |
|---|---|
| `--no-sync` | serve the existing mirror as-is; never touch the network |
| `--no-recompute` | pull, but don't rebuild derived tables afterwards |
| `--skip-geocode` | skip the place-reconcile stage of a rebuild (offline) |

### The tools

Nineteen, over visits, travel, places and trips:

| | |
|---|---|
| **Finding visits** | `search_visits`, `last_visit`, `first_visit`, `timeline`, `visits_near` |
| **Places** | `top_places`, `top_categories`, `places_visited`, `time_at_place` |
| **Travel** | `distance_traveled`, `journeys`, `country_timeline`, `trips` |
| **Statistics** | `get_summary`, `activity_over_time`, `list_years` |
| **Raw points** | `points_near`, `count_points`, `data_extent` |

So an assistant can answer things like *"when did I last go to that café in
Edinburgh?"*, *"how much time did I spend abroad in 2025?"*, *"which grocery
shop do I use most?"* or *"show me everywhere I stopped within a mile of this
postcode"* — with real answers from your own data rather than guesses.

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

## Installing the archive by hand

`brew install` is the supported path and needs nothing extra. If you'd rather
take the `.zip` from the Releases page, macOS quarantines anything a browser
downloads and will refuse to run it — *"Apple could not verify 'byway' is free of
malware"*. Clear the attribute:

```sh
unzip byway-<version>-macos-universal.zip
xattr -d com.apple.quarantine byway
```

That is not a way around a missing signature. Every published binary **is**
signed with a Developer ID certificate and **is** notarized by Apple; the
notarization ticket simply cannot be *stapled* to it, because a ticket attaches
only to a bundle, `.dmg` or `.pkg` and `byway` is a bare executable. Homebrew
fetches with `curl`, which never sets the quarantine attribute, which is why the
recommended path is unaffected.

## Verifying a copy

```sh
codesign --verify --strict --verbose=2 "$(brew --prefix)/bin/byway"
codesign -dvv "$(brew --prefix)/bin/byway" 2>&1 | grep -E 'Authority|TeamIdentifier'
```

You should see `Authority=Developer ID Application: Homegrown Software Ltd` and
`TeamIdentifier=2G974K78BH`.

Don't use `spctl --assess --type exec` here: it evaluates application bundles and
answers *"rejected (the code is valid but does not seem to be an app)"* for any
command-line tool, signed or not. It is not telling you anything is wrong.

## Licence

The `byway` binary is proprietary software, © Homegrown Software. It is
redistributable through this tap; the source is not published. The formula in
this repository exists to install it.
