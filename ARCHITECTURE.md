# Workspace architecture

The parent repository coordinates independently versioned components. Implementation belongs in the component that owns its behavior; integration documentation and cross-component release checks belong here. The shared gameplay datapack, base resource pack, optional Vivecraft overlay, and VR companion are separate components; structure templates belong to the base datapack. Manager remains a foundation.

## Ownership

| Component | Owns | Boundary |
| --- | --- | --- |
| `datapacks/` | shared gameplay, optional map behavior, persistent configuration, authoring functions, structure templates and their provenance, validation | Must run without a map add-on |
| `resourcepacks/` | Base pack and map models, textures, sounds, fonts, HUD, and optional VR presentation overrides | Matches base datapack resource identifiers |
| `vr_mod/` | Optional client-side Fabric/Vivecraft integration | Does not own server gameplay |
| `manager/` | Desktop installation, world preparation, and artifact packaging | Consumes component outputs and does not become their source of truth |
| `maps/` | Editable map worlds, dependency manifests, local links, and portable world builds | Optional consumer of tested component revisions; generated archives are build artifacts |

Each component is a Git submodule with independent history. The parent records exact commits. A component may be developed independently, but changes to shared identifiers or installation contracts require integration validation before updating the parent.

## Runtime dependencies

The base datapack and base resource pack form a compatible pair. The base pack includes shared round, dog, teleporter, game, menu, and character voice audio; optional map packs retain location cues, radio content, Easter eggs, and other map-specific presentation. The VR mod and VR overlay are optional client additions. Manager is an optional authoring and distribution tool. The base pack bundles all shared structure templates under its `data/` directory. All shared templates use `data/zbk/structure/<category>/<template>.nbt` and the `zbk:<category>/<template>` identifier; Pack-a-Punch uses the `pack_a_punch/` category. Build Kit placement, animation, clearing, and reset load them directly from the datapack; no external structure component or world installation is required. Map builds exclude the former world-installed base pack template directory to prevent it from overriding the bundled version.

Submodules manage source versions only. Map add-ons register through the [base pack event hooks](datapacks/README.md#base-pack-integration). Install one map provider alongside the base pack; conflicting or incompatible providers remain inactive. The base pack emits vanilla `#zbk:event/*` function tags, and add-ons call the owning `zbk:` core functions directly. Lifecycle entry points retain readiness checks, request handling, and event dispatch; callers must preserve the documented context and arguments. Nacht, Der Eisendrache, and the developer template are separate packs; no numeric map or sound selector is required. The base datapack and base resource pack both install as `zombies_build_kit`. The base pack gameplay functions, dialogs, and shared function IDs use `zbk:`; the base pack resource assets use `zbk:` for item models, fonts, textures, and sounds. The optional `zombies_build_kit_vivecraft_overlay` overrides `zbk:` held-weapon models and loads above the base pack.

The maps component tracks editable worlds and manifests with tested component revisions. Its local setup links those worlds and their source dependencies into Minecraft. GitHub Actions builds portable world ZIPs from the selected revisions. Region files use Git LFS; generated ZIPs, player records, and local links stay outside Git. Maps remain optional consumers of the base pack.

## Shared gameplay boundary

Retain reusable gameplay and authoring systems after checking their function and asset dependencies. Keep map-exclusive Nacht and Der Eisendrache quests, set pieces, items, audio, UI, structures, and configuration in their respective optional packs. Shared folders and generated namespaces must be audited as well as map-specific folders.

Persistent markers and their configuration are authoritative. Runtime entities should be reconstructed through their owning module's initialization. Root lifecycle functions orchestrate responsibilities; generated model files contain presentation output rather than handwritten gameplay.

Blockbench projects and authoring workspaces are excluded. Exported models and animation functions required by the base pack are runtime inputs and may be tracked. Document their provenance and any external regeneration requirements. Keep portable, maintained generators and validators with the component that owns their output.

## Packaging contract

Package explicit runtime roots rather than the entire development checkout. Exports must exclude repository metadata, local settings, development plans, assistant instructions/caches, and Blockbench sources. The base pack exports also exclude map-specific content; finished-map releases include only the selected map and its documented dependencies. Git ignore rules alone do not filter filesystem-based packaging. Every distributed component must include its LICENSES/ directory, including LICENSE.md, NOTICE, and MEDIA_PERMISSION.md, together with applicable third-party notices. Custom-map downloads containing ZBK material must retain the corresponding notices and terms.

Required runtime binaries such as PNG, OGG, and NBT files remain versioned. Preserve build inputs such as lockfiles and the Gradle wrapper. Build output, downloaded dependencies, local worlds, test servers, and exported archives do not belong in source commits.

## Documentation and validation

Each component owns documentation for its interfaces, requirements, and validation. Keep component links usable in standalone clones. The parent [development workflow](docs/development.md) explains coordination and release ordering.

Before migration is accepted, validate datapack commands and references, parse JSON, verify resource and structure dependencies, and run isolated load/reset/gameplay checks. Verify Manager packaging and real-client presentation separately. A successful Git checkout is not a substitute for gameplay or VR testing.
