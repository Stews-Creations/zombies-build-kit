# Zombies Build Kit

Zombies Build Kit is a Minecraft Java gameplay and map-authoring project. This repository coordinates the core components through Git submodules pinned to compatible commits.

In development for Minecraft Java 26.2. Shared core datapack and resource packs, structure templates, and the VR companion mod are available for integration testing. Manager is not migrated, and no finished map is included.

## Components

| Folder | Repository | Responsibility |
| --- | --- | --- |
| `datapacks/` | [zbk_datapacks](https://github.com/Stews-Creations/zbk_datapacks) | Core gameplay, map add-ons, and the developer template |
| `resourcepacks/` | [zbk_resourcepacks](https://github.com/Stews-Creations/zbk_resourcepacks) | Core and map assets with optional Vivecraft overlays |
| `vr_mod/` | [zbk_vr_mod](https://github.com/Stews-Creations/zbk_vr_mod) | Optional Vivecraft companion mod |
| `manager/` | [zbk_manager](https://github.com/Stews-Creations/zbk_manager) | Desktop authoring, installation, and export application |
| `structures/` | [zbk_structures](https://github.com/Stews-Creations/zbk_structures) | Reusable building templates |
| `maps/` | [zbk_maps](https://github.com/Stews-Creations/zbk_maps) | Finished-map catalog, installation guides, and release downloads |

The core includes reusable gameplay and authoring systems without map selection, map-exclusive events, or sound-pack selection. Nacht and Der Eisendrache are separate optional datapacks and resource packs. The `zbk_template` datapack provides a developer reference for the event API. Blockbench authoring projects are excluded; required generated runtime models and animations remain included. All packs start at version `1.0.0`.

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

The [architecture overview](ARCHITECTURE.md) defines component ownership, dependencies, and the core-only boundary. Submodule commits record a source combination; they do not install Minecraft packs or certify a playable release. See the [structures installation guide](https://github.com/Stews-Creations/zbk_structures#install-in-a-custom-map) for custom-world templates and the [VR mod guide](https://github.com/Stews-Creations/zbk_vr_mod#build) for building and installing the optional client companion. Build the core datapack ZIP using the [datapack guide](datapacks/README.md), and install the matching `zombies_build_kit` resource pack using the [resource-pack guide](resourcepacks/README.md). The optional Vivecraft overlay goes above the base resource pack.

## Build your own map add-on

Core owns the game state, rounds, shared enemies, weapons, and reusable systems. Your datapack owns your map's quests, events, and presentation. Core announces changes through vanilla function tags (`#zbk:event/*`); your functions listen and react. To intentionally change shared state, call the public `zbk:api/*` functions. Do not call private `zombies:` functions or edit Core's round state directly.

For example, Core runs `function #zbk:event/round_start`. Your datapack can subscribe with `data/zbk/tags/function/event/round_start.json`:

```json
{
  "replace": false,
  "values": ["my_map:events/round_start"]
}
```

Put your response in `data/my_map/function/events/round_start.mcfunction`. Notifications such as round start/end, power activation, enemy kills, and player downs/revives report what happened. Request hooks such as `before_game_start` let you block an action or defer a start for an intro through the API. Ordinary notifications cannot cancel Core behavior.

1. Copy the [datapack template](datapacks/zbk_template/README.md), rename its `zbk_template` identifiers to your own namespace, and retain the shared `zbk` event tag paths. Register your map with Core API `10000` (version `1.0.0`) and keep runtime inactive until Core accepts it. The template includes these lifecycle checks.
2. Add only your map's behavior and assets. Use matching names for your datapack and resource pack, such as `my_map`, with assets under `assets/my_map/`. Refer to those assets by namespaced IDs. A resource pack supplies presentation; it does not register a map or run event handlers.
3. Test Core alone and Core plus your add-on. Use the [API reference](datapacks/docs/API.md) for event context, public calls, blocking, and deferred starts. Core also controls Panzer round scheduling; without Panzer spawner markers, automatic Panzer spawns are disabled.

### Pack selection and order

Install `zombies_build_kit` and **one map provider** in the world's `datapacks/` folder. Choose `zbk_nacht_der_untoten`, `zbk_der_eisendrache`, or your own map. Do not ship `zbk_template` alongside a real map: it also registers as a provider. Multiple map providers leave map runtime inactive. Additional event-only add-ons can coexist if they do not register another provider.

Map registration does not depend on datapack load order: Core collects registrations on the tick after load. Keep event tags at `"replace": false` so their listeners combine, and never rely on the order of independent listeners. Higher-priority duplicate function files can replace lower-priority files, so give your implementations unique names instead of overriding Core files.

Resource packs use visual priority: **higher in the Resource Packs screen wins** for conflicting assets. Enable the matching map pack above Core. For the supplied packs, use this top-to-bottom order:

| World | Resource packs, highest priority first |
| --- | --- |
| Core only | `zombies_build_kit_vivecraft_overlay` (optional), `zombies_build_kit` |
| Nacht | `zombies_build_kit_vivecraft_overlay` (optional), `zbk_nacht_der_untoten`, `zombies_build_kit` |
| DE | `zbk_der_eisendrache_vivecraft_overlay` (optional), `zombies_build_kit_vivecraft_overlay` (optional), `zbk_der_eisendrache`, `zombies_build_kit` |

For your own map, put its resource pack above Core and any corresponding Vivecraft overlay above the assets it overrides. These dependencies are installation requirements; Minecraft does not download Core automatically.

### Bundle a finished world

Keep the source packs separate, then assemble a tested combination for each world:

1. Put the Core datapack ZIP and the chosen map datapack ZIP in `<world>/datapacks/`. Each ZIP must have `pack.mcmeta` at its root. Keeping two datapacks inside one world preserves the event API; they do not need to be merged into one datapack. Install the required world structures using the [structures guide](structures/README.md).
2. Either distribute the matching resource packs with the order above, or build one world resource pack as `<world>/resources.zip`. For a combined pack, start with Core assets, apply the map's assets above them, and keep one compatible root `pack.mcmeta`. Resolve shared JSON files such as sound catalogs and font definitions deliberately; blind folder copying can discard entries. Preserve licenses, notices, and credit. Keep Vivecraft transforms as an optional higher-priority client overlay unless the bundle specifically targets VR.
3. Test that exact world and resource combination before publishing. A multiplayer server must distribute or configure its resource pack separately; the server does not send its resource-pack folder to clients automatically. See the [finished-map repository](maps/README.md) for distribution guidance.

This is the intended assembly approach for our Nacht and DE worlds too. The current packaging tools export separate packs; automatic world bundling and resource-pack merging are not implemented yet.

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
