# Zombies Build Kit

Zombies Build Kit is a Minecraft Java gameplay and map-authoring project. This repository coordinates the shared components through Git submodules pinned to compatible commits.

In development for Minecraft Java 26.2. Shared base datapack and resource packs, structure templates, and the VR companion mod are available for integration testing. The server app hosts a prepared world for multiplayer. The maps component contains editable Nacht and template worlds; playable releases still require testing.

## Components

| Folder | Repository | Responsibility |
| --- | --- | --- |
| `datapacks/` | [zbk_datapacks](https://github.com/Stews-Creations/zbk_datapacks) | shared gameplay, bundled structure templates, map add-ons, and the developer template |
| `resourcepacks/` | [zbk_resourcepacks](https://github.com/Stews-Creations/zbk_resourcepacks) | base and map assets with optional Vivecraft overlays |
| `vr_mod/` | [zbk_vr_mod](https://github.com/Stews-Creations/zbk_vr_mod) | Optional Vivecraft companion mod |
| `server/` | [zbk_server](https://github.com/Stews-Creations/zbk_server) | Desktop app that hosts a world as a multiplayer server |
| `maps/` | [zbk_maps](https://github.com/Stews-Creations/zbk_maps) | Editable worlds, dependency manifests, local links, and portable builds |

The base pack includes reusable gameplay and authoring systems without map selection, map-exclusive events, or sound-pack selection. Nacht and Der Eisendrache are separate optional datapacks and resource packs. The `zbk_template` datapack provides a developer reference for the event hooks. Blockbench authoring projects are excluded; required generated runtime models and animations remain included. All packs start at version `1.0.0`.

The [maps repository](https://github.com/Stews-Creations/zbk_maps) is separate from the shared components. It tracks editable Nacht and template worlds and builds portable ZIPs from tested dependency revisions. See its [local setup and build guide](maps/README.md).

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

The [architecture overview](ARCHITECTURE.md) defines component ownership, dependencies, and the shared gameplay boundary. Submodule commits record a source combination; they do not install Minecraft packs or certify a playable release. The base pack includes its reusable structure templates in the datapack. See the [VR mod guide](https://github.com/Stews-Creations/zbk_vr_mod#build) for building and installing the optional client companion. Download the base datapack and template ZIPs from the [datapack releases](https://github.com/Stews-Creations/zbk_datapacks/releases), and download the matching base resource pack and optional Vivecraft overlay from the [resource-pack releases](https://github.com/Stews-Creations/zbk_resourcepacks/releases). Map-specific resource packs stay in source and will be bundled into finished world downloads.

## Build your own map add-on

The base pack owns the game state, rounds, shared enemies, weapons, and reusable systems. Your datapack owns your map's quests, events, and presentation. The base pack announces changes through vanilla function tags (`#zbk:event/*`); your functions listen and react. To change shared state, call the owning core function directly, such as `zbk:game/start/request` or `zbk:map_elements/power/management/on`. Use complete lifecycle operations instead of editing round state directly, and follow each function's execution-context requirements.

For example, the base pack runs `function #zbk:event/round/round_start`. Your datapack can subscribe with `data/zbk/tags/function/event/round/round_start.json`:

```json
{
  "replace": false,
  "values": ["my_map:events/round_start"]
}
```

Put your response in `data/my_map/function/events/round_start.mcfunction`. Notifications such as round start/end, power activation, enemy kills, and player downs/revives report what happened. Request hooks such as `before_game_start` let you block an action or defer a start for an intro through core request functions. Ordinary notifications cannot cancel base pack behavior.

1. Copy the [datapack template](datapacks/zbk_template/README.md), rename its `zbk_template` identifiers to your own namespace, and retain the shared `zbk` event tag paths. Register your map with the base pack compatibility revision `30000` and keep runtime inactive until the base pack accepts it. The template includes these lifecycle checks.
2. Add only your map's behavior and assets. Use matching names for your datapack and resource pack, such as `my_map`, with assets under `assets/my_map/`. Refer to those assets by namespaced IDs. A resource pack supplies presentation; it does not register a map or run event handlers.
3. Test the base pack alone and the base pack plus your add-on. Use the [integration guide](datapacks/README.md#base-pack-integration) for event context, public calls, blocking, and deferred starts. The base pack also controls Panzer round scheduling; without Panzer spawner markers, automatic Panzer spawns are disabled.

### Pack selection and order

Install `zombies_build_kit` and **one map provider** in the world's `datapacks/` folder. Choose `zbk_nacht_der_untoten`, `zbk_der_eisendrache`, or your own map. Do not ship `zbk_template` alongside a real map: it also registers as a provider. Multiple map providers leave map runtime inactive. Additional event-only add-ons can coexist if they do not register another provider.

Map registration does not depend on datapack load order: the base pack collects registrations on the tick after load. Keep event tags at `"replace": false` so their listeners combine, and never rely on the order of independent listeners. Higher-priority duplicate function files can replace lower-priority files, so give your implementations unique names instead of overriding the base pack files.

Resource packs use visual priority: **higher in the Resource Packs screen wins** for conflicting assets. Enable the matching map pack above the base pack. For the supplied packs, use this top-to-bottom order:

| World | Resource packs, highest priority first |
| --- | --- |
| Base pack only | `zombies_build_kit_vivecraft_overlay` (optional), `zombies_build_kit` |
| Nacht | `zombies_build_kit_vivecraft_overlay` (optional), `zbk_nacht_der_untoten`, `zombies_build_kit` |
| DE | `zbk_der_eisendrache_vivecraft_overlay` (optional), `zombies_build_kit_vivecraft_overlay` (optional), `zbk_der_eisendrache`, `zombies_build_kit` |

For your own map, put its resource pack above the base pack and any corresponding Vivecraft overlay above the assets it overrides. These dependencies are installation requirements; Minecraft does not download the base pack automatically.

### Bundle a finished world

Keep the source packs separate, then assemble a tested combination for each world:

1. Put the base datapack ZIP and the chosen map datapack ZIP in `<world>/datapacks/`. Each ZIP must have `pack.mcmeta` at its root. Keeping two datapacks inside one world preserves the event hooks; they do not need to be merged into one datapack. The base pack supplies its structure templates automatically; no separate structure installation is required.
2. For an official ZBK map release, build one world resource pack as `<world>/resourcepacks/resources.zip` so players need only the world download. Start with the base pack assets, apply the map's assets above them, and keep one compatible root `pack.mcmeta`. Resolve shared JSON files such as sound catalogs and font definitions deliberately; blind folder copying can discard entries. Preserve licenses, notices, and credit. Keep Vivecraft transforms as an optional higher-priority client overlay unless the bundle specifically targets VR.
3. Test that exact world and resource combination before publishing. A multiplayer server must distribute or configure its resource pack separately; the server does not send its resource-pack folder to clients automatically. The [server app](server/README.md) does this for a selected world. See the [finished-map repository](maps/README.md) for distribution guidance.

The [maps builder](maps/README.md#build-portable-worlds) assembles Nacht and template worlds automatically from their dependency manifests, including separate datapacks with bundled structures and one merged resource pack. Resource files are overlaid in manifest order; duplicate JSON files are replaced whole rather than merged by key. Author complete overrides when paths overlap. These workflow artifacts are development builds and require gameplay testing before release. Der Eisendrache is not currently included in the world build manifests.

## Contributions and feedback

We are not currently accepting outside development help or pull/merge requests while we establish the project. Please hold off on submitting changes for inclusion.

You are welcome to fork the repository and [open issues](https://github.com/Stews-Creations/zombies-build-kit/issues) to report bugs, suggest features, ask questions, or share feedback. We will update this section when we are ready to accept contributions.

## License and credit

Free noncommercial use, modification, and sharing are allowed with credit to
[MiniStew](https://www.youtube.com/@MiniStew). Monetized videos and streams are
allowed under the [media permission](LICENSES/MEDIA_PERMISSION.md). Selling covered ZBK
content or maps containing it, or charging for server access, is not covered
by that permission. See [licensing and attribution](LICENSES/LICENSE.md) for the code
and asset licenses, their scope, and redistribution requirements.
