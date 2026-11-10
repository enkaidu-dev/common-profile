---
name: ruby-with-enkaidu
description: >-
  Run and manage Ruby through Enkaidu's `shell_command` tool. Use whenever you need
  to execute Ruby: quick `ruby -e` checks, reading or parsing files, JSON/YAML work,
  spawning a short-lived subprocess, or running a `.rb` script. Trigger this even when
  the user does not explicitly say "Ruby" but wants to run, test, or diagnose a
  snippet, a calculation, or some data wrangling.
---

# Using Ruby with Enkaidu

This skill captures what has been observed about using Ruby inside an Enkaidu session,
especially the constraints of the `shell_command` tool and the sandbox (cordon) mode.
It is a living document — add to it as we learn more.

## What is Enkaidu?
1. Enkaidu is an agentic assistant for use with local and self-hosted AI models.
2. Enkaidu is intended to run locally and securely.
3. Enkaidu does not support running arbitrary custom scripts as part of skills.

## The one hard constraint: the allowlist
The `shell_command` tool only permits a short list of command binaries.
To run Ruby, start the command with `ruby`. Other
binaries may be rejected unless explicitly listed as allowed or approved by the shell command's
input description.

The gate is **not a real shell**. It checks the command string roughly as `binary + args`
and does not interpret pipes, redirects, or command chaining the way bash would, so those
tokens get rejected rather than executed. Concretely:

| Pattern | Result | Why |
|---|---|---|
| `ruby -v`, `ruby -e 'puts 1'` | OK | Bare allowed binary |
| multi-line in `-e` via newlines | OK | Newlines separate statements |
| `ruby -e 'a; b'` | REJECTED | `;` splits the cmd; 2nd fragment is not an allowed binary |
| `… \| …`, `… & …`, `..` in a path | REJECTED | Forbidden tokens / path-traversal guard |
| anything containing `clean` or `wipe` | BLOCKED | Listed in `restricted_terms` |

The recurring culprit is **`;`**. Even inside a single-quoted `-e` string, a semicolon
splits the command and the trailing fragment looks like a bare command, so the gate rejects
it. The fix every time: **use newlines, not semicolons.**

## Running code

For anything small (fewer than ~5 lines), run it inline. Newlines separate statements:

```
ruby -e '
require "json"
puts JSON.pretty_generate({ a: 1, b: [1, 2, 3] })
'
```

A single statement is just:

```
ruby -e 'puts File.read("README.md").lines.size'
```

## When and where to create temp scripts
For longer work, cramming it all into `-e` gets fragile (escaping, the `;` trap, hard to
re-run). Prefer a real file:

1. Write the Ruby to a file in **`./tmp/scripts`** using the `write_text_file` tool, e.g.
   `./tmp/scripts/parse_config.rb`. `write_text_file` creates parent directories as needed.
2. Run it with a single allowed binary, args only: `ruby ./tmp/scripts/parse_config.rb`.
   This passes the gate cleanly and is easy to read, re-run, and tweak.

Additional rules:
- **Do not delete the scripts you create.** They are kept so the kinds of Ruby scripts in
   use can be reviewed later.
- **Use `./tmp/scripts`, not `/tmp`** — `/tmp` is not writable from inside the sandbox.
- Use the tools for reading, writing and editing files


## Spawning a subprocess from Ruby
Ruby remains a normal interpreter, so it can exec child processes:

```
ruby -e 'puts `cat README.md`'
```

The allowlist only governs the *top-level binary*, not what Ruby does internally. Use this
responsibly for a quick external call; mind that the cordon is ReadWrite, and prefer
`read_text_file` for ordinary file reads.

## Environment notes (observed — may change)
- Ruby present: `ruby 4.0.6` on `arm64` (Darwin 25). This is a preview/leading-edge Ruby,
  so APIs may differ from a stable release — confirm behavior rather than assuming
  stable-Ruby semantics.
- Ruby is managed by the cordon (`using_ruby.ruby_path: $RUBY_ROOT/bin/ruby`).
- `ruby` runs with cwd at the repo root, so use paths relative to that.
- `cordon.mode: commands` with `skip_confirm_with_cordon: true` means allowed `ruby`
  commands run without a confirmation prompt.
- These notes are version-specific; revisit if the Ruby version or sandbox policy changes.

## Gotchas / checklist
- No shell operators in the command string: avoid `;`, `&`, `|`, `&&`, `>`, `..`.
- Multi-statement → newlines in the `-e` string, or write a `.rb` file and run it.
- Only `ruby` is a good fit here as the starting binary (other binaries get rejected).
- For work beyond a few lines, prefer `write_text_file` into `./tmp/.scratch` then
   `ruby <path>` over a large `-e` blob.
- If a command is rejected, read the error: "Only the following commands are allowed: …"
   means wrong binary or a forbidden token — strip `;`/`&`/`|`/`..` or a restricted term.

## Maintenance note
This skill is meant to be updated over time as the allowlist, Ruby version, or cordon mode
changes. Log non-trivial changes under `CHANGELOG.md` in this folder with a date and the
observed behavior, so the next instance can trust and extend it instead of re-deriving.
Keep the two testing skills (crystal-testing, crystal-spectator-testing) separate — those
handle Crystal, not Ruby.
