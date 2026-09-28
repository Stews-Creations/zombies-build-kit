# Core workspace architecture

The parent repository coordinates independently versioned components. Implementation belongs in the component that owns its behavior; integration documentation and cross-component release checks belong here. The shared gameplay datapack, core resource pack, optional Vivecraft overlay, structure templates, and VR companion are separate components. Manager remains a foundation.

## Ownership

| Component | Owns | Boundary |
| --- | --- | --- |
| `datapacks/` | Core gameplay, optional map behavior, persistent configuration, authoring functions, validation | Must run without a map add-on |
| `resourcepacks/` | Core and map models, textures, sounds, fonts, HUD, and optional VR presentation overrides | Matches core datapack resource identifiers |
| `vr_mod/` | Optional client-side Fabric/Vivecraft integration | Does not own server gameplay |
| `manager/` | Desktop installation, world preparation, and artifact packaging | Consumes component outputs and does not become their source of truth |
| `structures/` | Reusable structure templates and their provenance | Shared placement, door animation, clearing, and reset functions consume world-installed templates |
| `maps/` | Finished-map catalog, installation guides, credits, and release downloads | Optional consumer of versioned core components; playable world archives are release assets |

Each component is a Git submodule with independent history. The parent records exact commits. A component may be developed independently, but changes to shared identifiers or installation contracts require integration validation before updating the parent.

## Runtime dependencies

The core datapack and base resource pack form a compatible pair. Core includes shared round, dog, teleporter, game, menu, and character voice audio; optional map packs retain location cues, radio content, Easter eggs, and other map-specific presentation. The VR mod and VR overlay are optional client additions. Manager is an optional authoring and distribution tool. External structure templates support both Build Kit placement and subsequent shared runtime operations, so the complete set stays installed in the world. Datapack-owned Pack-a-Punch templates remain a separate namespace and are not duplicated into the external bundle.

Submodules manage source versions only. Map add-ons register through the [Core event API](datapacks/docs/API.md). Install one map provider alongside Core; conflicting or incompatible providers remain inactive. Core emits vanilla `#zbk:event/*` function tags, and add-ons call the public `zbk:api/*` functions. Nacht, Der Eisendrache, and the developer template are separate packs; no numeric map or sound selector is required. The core datapack and base resource pack both install as `zombies_build_kit`. Core gameplay functions, dialogs, and other datapack IDs retain `zombies:`; Core resource assets use `zbk:` for item models, fonts, textures, and sounds. The optional `zombies_build_kit_vivecraft_overlay` overrides `zbk:` held-weapon models and loads above the base pack.

The maps component documents each finished map's tested dependency versions and installation requirements. Its submodule pin selects catalog documentation; it does not download playable worlds or make those maps dependencies of the core. Editable worlds and packaged downloads stay outside Git, and finished worlds are distributed through that repository's releases.

## Core migration boundary

Retain reusable gameplay and authoring systems after checking their function and asset dependencies. Keep map-exclusive Nacht and Der Eisendrache quests, set pieces, items, audio, UI, structures, and configuration in their respective optional packs. Shared folders and generated namespaces must be audited as well as map-specific folders.

Persistent markers and their configuration are authoritative. Runtime entities should be reconstructed through their owning module's initialization. Root lifecycle functions orchestrate responsibilities; generated model files contain presentation output rather than handwritten gameplay.

Blockbench projects and authoring workspaces are excluded. Exported models and animation functions required by core are runtime inputs and may be tracked. Document their provenance and any external regeneration requirements. Keep portable, maintained generators and validators with the component that owns their output.

## Packaging contract

Package explicit runtime roots rather than the entire development checkout. Exports must exclude repository metadata, local settings, development plans, assistant instructions/caches, and Blockbench sources. Core exports also exclude map-specific content; finished-map releases include only the selected map and its documented dependencies. Git ignore rules alone do not filter filesystem-based packaging. Every distributed component must include its LICENSE.md, LICENSES/, NOTICE, and MEDIA_PERMISSION.md, together with applicable third-party notices. Custom-map downloads containing ZBK material must retain the corresponding notices and terms.

Required runtime binaries such as PNG, OGG, and NBT files remain versioned. Preserve build inputs such as lockfiles and the Gradle wrapper. Build output, downloaded dependencies, local worlds, test servers, and exported archives do not belong in source commits.

## Documentation and validation

Each component owns documentation for its interfaces, requirements, and validation. Keep component links usable in standalone clones. The parent [development workflow](docs/development.md) explains coordination and release ordering.

Before migration is accepted, validate datapack commands and references, parse JSON, verify resource and structure dependencies, and run isolated load/reset/gameplay checks. Verify Manager packaging and real-client presentation separately. A successful Git checkout is not a substitute for gameplay or VR testing.
