# Common Profile for Enkaidu

This is intended to be a generally useful [Enkaidu](https://enkaidu.dev/) profile with prompts, system prompts, and macros.

> ⚠️ WORK IN PROGRESS. EXPECT CHANGES.

An Enkaidu profile for a project lives in the `.enkaidu` folder. This repo contains the files that go **inside** the profile folder.

<!-- TOC -->
- [Installing](#installing)
  - [Download](#download)
  - [Git sub-module](#git-sub-module)
- [Documentation](#documentation)
  - [Namespaces](#namespaces)
  - [Actions](#actions)
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

### Namespaces

Enter `/macro ls` to see available macros

The following namespaces should be apparent:
- `codex.` for resources based on those from [Codex](https://github.com/openai/codex).
- `simple.` for resources that we've discovered and evolved while using Enkaidu with smaller local models.

If you are running models with <= 24K of tokens in the context, use the `simple.` resources.

Otherwise, try both and decide which works for you. You might even want to consider doing some work with one and then switching to the other.

> 👆 While you can do so, you're better off sticking to namespace-specific commands after entering a namespace-specific session. For example, if you use `!codex.enter`, don't use `simple.*` commands until after using `!codex.leave`

### Actions

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
