# Common Profile for Enkaidu

This is intended to be a generally useful [Enkaidu](https://enkaidu.dev/) profile with prompts, system prompts, and macros.

> ⚠️ WORK IN PROGRESS. EXPECT CHANGES.

An Enkaidu profile for a project lives in the `.enkaidu` folder. This repo contains the files that go **inside** the profile folder.

<!-- TOC -->
- [Installing](#installing)
  - [Download](#download)
  - [Git sub-module](#git-sub-module)
- [Documentation](#documentation)
  - [Personal Memory (EXPERIMENTAL)](#personal-memory-experimental)
  - [Coding Agents](#coding-agents)
- [Structure](#structure)
- [Conventions](#conventions)
- [Contributions, by invitation!](#contributions-by-invitation)
<!-- /TOC -->

## Installing

### Download

[Download a ZIP file](https://github.com/enkaidu-dev/common-profile/archive/refs/heads/main.zip) of the entire repo and
- Uncompress the archive into a folder, likely one called `common-profile` by default;
- Rename the folder to `.enkaidu`
- Move it into the project where you're using Enkaidu

> 🧐 Tip: You can keep this folder in your home or `Documents` folder and then create an alias (macOS) or short-cut (Windows) or symlink (Linux-likes) to the folder in any and all project folders.

### Git sub-module

To use it entirely in your project's Git repo, pull it in as a submodule and map it to `.enkaidu` like so:

```sh
git submodule add https://github.com/enkaidu-dev/common-profile .enkaidu
```

And run the following to pick up updates to the common profile:

```sh
# Pull latest commit from the remote that is tracked by the submodule
git submodule update --remote .enkaidu
```

## Documentation

Enkaidu doesn't have actual namespaces. Instead we use a period to separate the namespace from the name of macros, prompts and system prompts.

### Personal Memory (EXPERIMENTAL)

> ⚠️ Requires Enkaidu 0.8.7 or later, since YAML files live in sub-folders under `prompts/`, `macros/` and `system_prompts/`.

The `personal.` namespace defines actions for maintaining a persistent memory by maintaining this folder structure in your workspace:

```
.personal/
├── INDEX.md - Navigation hub
├── MEMORY.md - High-level synthesis
├── AGENTS.md - Instructions for agents
├── CONVENTIONS.md - Formatting rules
├── memories/
│   └── YYYY-MM-DD-memories.md - Daily chronological records
└── knowledge/
    └── subject-name/
        ├── ABOUT.md - Main knowledge document
        ├── CHANGELOG.md - Evolution of understanding
        ├── _meta.json - Metadata
        └── references/ - Supporting materials
```

There are a couple of ways to use these macros.

#### Lifecycle macros

#### `!personal.enter`

Start a nested session with the system prompt and configuration for personal memory system.

#### `!personal.init`

Use this to initialize the personal memory system. Only needed once. If `.personal/` exists with various files, you don't need to run this macro.

#### `!personal.compact`

This will attempt to update memories and knowledge and then compact the current session, replacing your current session context with the compact version that is generated in a nested session.

#### `!personal.leave`

This will leave the session started by `!personal.enter`. It performs compaction (using `!personal.compact`) and then return the parent session _without resetting the parent session's memory_.

This means you can _enter_ and _leave_ multiple times and collect and gather the checkpoints for the sessions.

#### Separate session

Instead of entering and leaving in your working session, you could just spin up a separate session to use for personal updates.

Run `!personal.launch` to start a new session called `personal`. You can then use `/session goto personal` to switch to it.

### Coding Agents

Currently we have the following coding agent name spaces.

- `codex.` for resources based on those from [Codex](https://github.com/openai/codex).
- `simple.` for resources that we've discovered and evolved while using Enkaidu with smaller local models.

If you are running models with <= 24K of tokens in the context, use the `simple.` resources.

Otherwise, try both and decide which works for you. You might even want to consider doing some work with one and then switching to the other.

> 👆 While you can do so, you're better off sticking to namespace-specific commands after entering a namespace-specific session. For example, if you use `!codex.enter`, don't use `simple.*` commands until after using `!codex.leave`

Each namespace defined the following actions as macros.

#### `*.enter`

Use `!<NAMESPACE>.enter` to start a session with the system prompt and configuration for the namespace.

E.g. `!simple.enter`

#### `*.init`

Use `!<NAMESPACE>.init` to initialize your project with an `AGENTS.md` file. This will update one if it already exists.

E.g. `!simple.init`

#### `*.compact`

Use `!<NAMESPACE>.compact` to create a compact context checkpoint with a hand-off summary. When your session has become long or when you're done with a particular goal or objective, this command can help to reduce the size of the context and keep enough information so that you can start your next thing.

This will replace your current session context with the compact version that is generated in a nested session.

E.g. `!simple.compact`

#### `*.leave`

Use `!<NAMESPACE>.compact` to leave the session started by `!*.enter`. This will perform a compaction using `!*.compact` and then take that context checkpoint / hand-off summary to the parent session. The parent session _will not be reset_.

This means you can _enter_ and _leave_ multiple times and collect and gather the checkpoints for the sessions.

Combine this with `/session save ...` and `/session load ...` to persist checkpoints over time if that is useful.

E.g. `!simple.leave`


## Structure

An [Enkaidu profile](https://enkaidu.dev/docs/using_enkaidu/profiles/) can have prompts, system prompts, and macros in single respectively named YAML files, or as many YAML files within respectively named folders.

This repository contains folders:

Folder            | Description
------------------|---------------------------------------------------------
`macros/`         | Contains macros in functionally names YAML files
`prompts/`        | Contains prompts in functionally names YAML files
`system_prompts/` | Contains system prompts in functionally names YAML files

## Conventions

1. When macros, prompts, and system_prompts are related, there should be a file with the same name in each folder so that it's clear the capabilities are related.

2. Within such files, consider a unique prefix for all those entities.

See the `simple.yml` files under each folder to get a sense of the naming and name-spacing.

## Contributions, by invitation!

*With apologies*, at this time contributions are *by invitation only* and limited to people I know and see often.

These are early days for Enkaidu and I am busy with family and work.

At this time I want to work on this at a manageable pace.
