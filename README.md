# Zombies Build Kit

Zombies Build Kit is a Minecraft Java gameplay and map-authoring project. This repository coordinates the core components through Git submodules pinned to compatible commits.

In development for Minecraft Java 26.2. Structure templates and the VR companion mod are available; core gameplay, resource packs, and Manager are not ready yet. There is no complete playable release.

## Components

| Folder | Repository | Responsibility |
| --- | --- | --- |
| `datapacks/` | [zbk_datapacks](https://github.com/Stews-Creations/zbk_datapacks) | Core gameplay and reusable Build Kit functions |
| `resourcepacks/` | [zbk_resourcepacks](https://github.com/Stews-Creations/zbk_resourcepacks) | Shared runtime assets and optional Vivecraft overlay |
| `vr_mod/` | [zbk_vr_mod](https://github.com/Stews-Creations/zbk_vr_mod) | Optional Vivecraft companion mod |
| `manager/` | [zbk_manager](https://github.com/Stews-Creations/zbk_manager) | Desktop authoring, installation, and export application |
| `structures/` | [zbk_structures](https://github.com/Stews-Creations/zbk_structures) | Reusable building templates |
| `maps/` | [zbk_maps](https://github.com/Stews-Creations/zbk_maps) | Finished-map catalog, installation guides, and release downloads |

The initial implementation includes only reusable core systems. Nacht and Der Eisendrache-specific content and Blockbench authoring projects are outside its scope. Required exported runtime models and animations remain eligible for migration after their dependencies are checked.

The [finished-map repository](https://github.com/Stews-Creations/zbk_maps) is separate from the core components. No finished maps are published there yet; future playable world downloads will be release assets, with compatibility and installation details for each map.

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

The [architecture overview](ARCHITECTURE.md) defines component ownership, dependencies, and the core-only boundary. Submodule commits record a source combination; they do not install Minecraft packs or certify a playable release. See the [structures installation guide](https://github.com/Stews-Creations/zbk_structures#install-in-a-custom-map) for custom-world templates and the [VR mod guide](https://github.com/Stews-Creations/zbk_vr_mod#build) for building and installing the optional client companion. Other installation and release instructions will be added with their implementations.

## Contributions and feedback

We are not currently accepting outside development help or pull/merge requests while we establish the core project. Please hold off on submitting changes for inclusion.

You are welcome to fork the repository and [open issues](https://github.com/Stews-Creations/zombies-build-kit/issues) to report bugs, suggest features, ask questions, or share feedback. We will update this section when we are ready to accept contributions.

## License and credit

Free noncommercial use, modification, and sharing are allowed with credit to
[MiniStew](https://www.youtube.com/@MiniStew). Monetized videos and streams are
allowed under the [media permission](MEDIA_PERMISSION.md). Selling covered ZBK
content or maps containing it, or charging for server access, is not covered
by that permission. See [licensing and attribution](LICENSE.md) for the code
and asset licenses, their scope, and redistribution requirements.
