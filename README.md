# Zombies Build Kit

Zombies Build Kit is a Minecraft Java gameplay and map-authoring project. This repository coordinates the core components through Git submodules pinned to compatible commits.

In development for Minecraft Java 26.2. No playable release is available yet.

## Components

| Folder | Repository | Responsibility |
| --- | --- | --- |
| `datapacks/` | [zbk_datapacks](https://github.com/Stews-Creations/zbk_datapacks) | Core gameplay and reusable Build Kit functions |
| `resourcepacks/` | [zbk_resourcepacks](https://github.com/Stews-Creations/zbk_resourcepacks) | Shared runtime assets and optional Vivecraft overlay |
| `vr_mod/` | [zbk_vr_mod](https://github.com/Stews-Creations/zbk_vr_mod) | Optional Vivecraft companion mod |
| `manager/` | [zbk_manager](https://github.com/Stews-Creations/zbk_manager) | Desktop authoring, installation, and export application |
| `structures/` | [zbk_structures](https://github.com/Stews-Creations/zbk_structures) | Reusable building templates |

The initial implementation includes only reusable core systems. Nacht and Der Eisendrache-specific content and Blockbench authoring projects are outside its scope. Required exported runtime models and animations remain eligible for migration after their dependencies are checked.

## Get the workspace

Install Git with access to GitHub, then clone all components:

```powershell
git clone --recurse-submodules https://github.com/Stews-Creations/zombies-build-kit.git
cd zombies-build-kit
git submodule status --recursive
```

For an existing clean checkout:

```powershell
git pull --ff-only
git submodule update --init --recursive
```

These commands select the component commits recorded by the parent repository. Commit or preserve local component work before updating. See the [development workflow](docs/development.md) for editing and publishing changes.

## Architecture and releases

The [architecture overview](ARCHITECTURE.md) defines component ownership, dependencies, and the core-only boundary. Submodule commits record a source combination; they do not install Minecraft packs or certify a playable release. Installation and release instructions will be added with the implementations.
