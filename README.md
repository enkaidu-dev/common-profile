# Common Profile for Enkaidu

> ☝️ COMPATIBILITY - Requires Enkaidu 0.9.10 or better

> ⚠️ WORK IN PROGRESS, some namespaces are more "in progress" than orders. Stables ones are identified below.

This is an [Enkaidu](https://enkaidu.dev/) "profile" with multiple "purpose-driven" namespaces containint custom prompts, system prompts, and macros. Each namespace provides a consistent way of engaging with it.

- "Enter" via a forked session
- "Init" after entering if using the namespace for the first-time in current project
- "Compact" current session
- "Leave" forked session you _entered_ earlier, which performs namespace-specific closing operations before running compacting and exiting the session.

Additionally, as a convenience, you can "Launch" into a separate session, automatically invoking "Enter" as well.

<!-- TOC -->
**Table of contents**
- [USAGE](#usage)
  - [Clone and symlink](#clone-and-symlink)
  - [Git submodule](#git-submodule)
  - [Download](#download)
- [STABLE](#stable)
  - [Global macros](#global-macros)
  - [Coding Agents](#coding-agents)
- [EXPERIMENTAL](#experimental)
  - [Skills (EXPERIMENTAL)](#skills-experimental)
- [Development](#development)
  - [Naming](#naming)
  - [Conventions](#conventions)
- [Contributions, by invitation!](#contributions-by-invitation)
<!-- /TOC -->

## USAGE

An Enkaidu profile for a project lives in the `.enkaidu` folder. This repo contains the files that go **inside** the profile folder.

### Clone and symlink

If you have your own `.enkaidu/` folder with other content, or if you want to share across projects but not commit / include in your codebase / repo, you can do this:

1. Clone the repo somewhere outside your project.
```sh
git clone https://github.com/enkaidu-dev/common-profile.git
```
2. From your project's repo, use `ln -s` to symlink the entire repo as `.enkaidu`, OR create an `.enkaidu/` folder and then symlink in the sub-folders, OR ... you get the idea. ;-)
```sh
# For example
ln -s ../common-profile .enkaidu
```
3. Add `.enkaidu` to your `.gitignore` file

### Git submodule

To use it entirely in your project's Git repo, pull it in as a submodule and map it to `.enkaidu` like so:

```sh
git submodule add https://github.com/enkaidu-dev/common-profile .enkaidu
```

And run the following to pick up updates to the common profile:

```sh
# Pull latest commit from the remote that is tracked by the submodule
git submodule update --remote .enkaidu
```

> 🧐 Tip: You can put the submodule under a different folder and then use symlinks as described in the section above.

### Download

[Download a ZIP file](https://github.com/enkaidu-dev/common-profile/archive/refs/heads/main.zip) of the entire repo and
- Uncompress the archive into a folder, likely one called `common-profile` by default;
- Rename the folder to `.enkaidu`
- Move it into the project where you're using Enkaidu

> 🧐 Tip: You can keep this folder in your home or `Documents` folder and then create an alias (macOS) or short-cut (Windows) or symlink (Linux-likes) to the folder in any and all project folders.

## STABLE

### Global macros

The profile defines four globally usable macros: **task**, **task_json**, **analyze**, and **brief**. Each macro follows a uniform three‑step workflow: start a fresh session, run a user‑provided query, and then clean up the session while preserving the outline. This design allows the macros to be invoked from any namespace while keeping the execution context isolated and deterministic.

| Action                              | Example                                              | Include history? | Description                                                                                                                                                                                                                                         |
|-------------------------------------|------------------------------------------------------|------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Run a task with a full prompt       | `!task "Your prompt"`                                | Yes              | Executes the given prompt in a clean session and returns the session outline.                                                                                                                                                                       |
| Run a task and return JSON response | `!task_json "Your prompt" schema=path/to/schema.yml` | Yes              | Executes the prompt in a clean session and ensures the response conforms to the provided JSON response schema file that conforms to the following structure: `{ "name" : <string>, "description": <string>, "strict": <bool>, "schema": <object> }` |
| Analyze a file (step‑by‑step)       | `!analyze path/to/file.ext`                          | No               | Runs a detailed analysis of the specified file, identifying main points and summarizing them.                                                                                                                                                       |
| Briefly summarize a file            | `!brief path/to/file.ext`                            | No               | Generates a concise bullet‑point summary of the specified file.                                                                                                                                                                                     |


### Coding Agents

Currently we have the following coding agent name spaces.

- `codex.` for macros / prompts based on those from [Codex](https://github.com/openai/codex).
- `dev.` for macros / prompts that we've discovered and evolved while using Enkaidu with smaller local models.

> 👆 If you are running models with <= 32K of tokens in the context, use the `dev.` macros.

Feel free to try both and decide which works for you. You might even want to consider doing some work with one and then switching to the other.

> 👆 While you can mix macros from one namespace after using `.enter` from another, though I suggest sticking to namespace-specific macros after entering a namespace-specific session until you `.leave` that session.

Each namespace (referred to as `NS` below) defines the following actions as macros.

<!-- markdown-table-prettify-ignore-start -->
| Action                                                                                                                                                                                                                                                                                                                                                                                                                                                                        | Example           |
|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------|
| Use `!<NS>.enter` to start a session with the system prompt and configuration for the namespace.                                                                                                                                                                                                                                                                                                                                                                       | `!dev.enter` or `!codex.enter`   |
| Use `!<NS>.init` to initialize your project with an `AGENTS.md` file. This will update one if it already exists.                                                                                                                                                                                                                                                                                                                                                       | `!simple.init`  or `!codex.init`   |
| Use `!<NS>.compact` to create a compact context checkpoint with a hand-off summary. When your session has become long or when you're done with a particular goal or objective, this command can help to reduce the size of the context and keep enough information so that you can start your next thing. **This will replace your current session context with the compact version that is generated in a nested session.**                                           | `!simple.compact` or `!codex.compact` |
| Use `!<NS>.leave` to leave the session started by `!*.enter`. This will perform a compaction using `!*.compact` and then take that context checkpoint / hand-off summary to the parent session. **The parent session _will not be reset_**. You can _enter_ and _leave_ multiple times and collect and gather the checkpoints for the sessions.<br>*Combine this with `/session save ...` and `/session load ...` to persist checkpoints over time if that is useful.* | `!simple.leave` or `!codex.leave`   |
<!-- markdown-table-prettify-ignore-end -->

### Skills

The goal of these macros is to help understand and use (and create) skills without building the notion of skills into Enkaidu.

A *skill* is any folder under `./.enkaidu/skills/` — at any depth — that directly contains a `SKILL.md` file with YAML front-matter (name, description, etc.). Folders without a `SKILL.md` are just organizational folders used to group skills, not skills themselves. A skill's name is the name of the folder directly containing its `SKILL.md`. For example:

```
.enkaidu/skills/
  +-- about-skills/SKILL.md
  +-- cloudops/
      +-- ops/SKILL.md
  +-- project/
      +-- do-this/SKILL.md
      +-- do-that/SKILL.md
```

Here `about-skills`, `ops`, `do-this`, and `do-that` are skills, while `cloudops` and `project` are just organizational folders.

Two skills are pre-defined as they are used by the skill creator macro:
- `about-skills`
- `skill-creator`

The following macros related to skills are available to play with.

| Action                  | Example                  | Include history? | Description                                                                                                                                                                                                                                                            |
|-------------------------|--------------------------|------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Initialize skills       | `!skill.init`            | Yes              | Prepares the session to work with skills: lists the available skills (via `!skill.list`) and loads the `about-skills` skill so its guidance is in context.                                                                                                            |
| List available skills   | `!skill.list`            | No               | Recursively finds every `SKILL.md` under `./.enkaidu/skills` (at any depth) and outputs a JSON array where each element contains the full path to that `SKILL.md` plus the front‑matter fields (name, description, etc.).                                              |
| Use a skill by its name | `!skill.use SKILLNAME`   | N/a              | Loads the named skill into the current session. Searches recursively under `./.enkaidu/skills` for a folder named `SKILLNAME` containing a `SKILL.md`, stops and reports if none is found, otherwise outputs a concise summary of the skill and next‑step instructions if applicable. |
| Create your own skill   | `!skill.create "Prompt"` | Yes              | Initiates a 2‑level nested session to create a new skill from the quoted prompt. It loads `about-skills` and `skill-creator`, researches the topic (with Web content retrieval preferred as markdown), proposes candidate names, and after confirmation automatically creates the skill folder and files, then returns to the original session. |
| Review an existing skill | `!skill.review "Prompt"` | Yes              | Initiates a 2‑level nested session to review and update an existing skill based on the quoted prompt. It stops immediately if the described skill doesn't exist; otherwise it loads the skill and researches the topic (with Web content retrieval preferred as markdown) to improve it.                                            |

## Development

An [Enkaidu profile](https://enkaidu.dev/docs/using_enkaidu/profiles/) can have prompts, system prompts, and macros in single respectively named YAML files, or as many YAML files within respectively named folders.

This repository contains folders:

| Folder            | Description                                              |
|-------------------|----------------------------------------------------------|
| `macros/`         | Contains macros in functionally names YAML files         |
| `prompts/`        | Contains prompts in functionally names YAML files        |
| `system_prompts/` | Contains system prompts in functionally names YAML files |

### Naming

Enkaidu doesn't support namespaces syntactically. Instead we use a period `'.'` in the name of prompts, system prompts and macros to separate the namespace from their functional names, like so: `<NAMESPACE>.<FUNCTION>`

E.g. `codex.init` vs `simple.init`

### Conventions

1. When macros, prompts, and system_prompts are related, there should be a file with the same name in each folder so that it's clear the capabilities are related.

2. Within such files, consider a unique prefix for all those entities.

See the `simple.yml` files under each folder to get a sense of the naming and name-spacing.

## Contributions, by invitation!

*With apologies*, at this time contributions are *by invitation only* and limited to people I know and see often.

These are early days for Enkaidu and I am busy with family and work.

At this time I want to work on this at a manageable pace.
