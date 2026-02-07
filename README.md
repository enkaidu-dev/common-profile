# Common Profile for Enkaidu

This is intended to be a generally useful [Enkaidu](https://enkaidu.dev/) profile with prompts, system prompts, and macros.

An Enkaidu profile for a project lives in the `.enkaidu` folder. This repo contains the files that go **inside** the profile folder.

## Usage

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
