# Core workspace architecture

The parent repository coordinates independently versioned components. Implementation belongs in the component that owns its behavior; integration documentation and cross-component release checks belong here. Reusable structure templates have been migrated; the gameplay, resource-pack, and application components remain foundations.

## Ownership

| Component | Owns | Boundary |
| --- | --- | --- |
| `datapacks/` | Server gameplay, persistent configuration, reusable authoring functions, gameplay validation | Must run without a map add-on |
| `resourcepacks/` | Core models, textures, sounds, fonts, HUD, and optional VR presentation overrides | Matches core datapack resource identifiers |
| `vr_mod/` | Optional client-side Fabric/Vivecraft integration | Does not own server gameplay |
| `manager/` | Desktop installation, world preparation, and artifact packaging | Consumes component outputs and does not become their source of truth |
| `structures/` | Reusable structure templates and their provenance | Shared placement, door animation, clearing, and reset functions consume world-installed templates |

Each component is a Git submodule with independent history. The parent records exact commits. A component may be developed independently, but changes to shared identifiers or installation contracts require integration validation before updating the parent.

## Runtime dependencies

The core datapack and base resource pack form a compatible pair. The VR mod and VR overlay are optional client additions. Manager is an optional authoring and distribution tool. External structure templates support both Build Kit placement and subsequent shared runtime operations, so the complete set stays installed in the world. Datapack-owned Pack-a-Punch templates remain a separate namespace and are not duplicated into the external bundle.

Submodules manage source versions only. Future map add-ons will need an explicit gameplay interface and compatible core version. No add-on registration, dependency detection, or map selection system is implemented by this foundation.

## Core migration boundary

Retain reusable gameplay and authoring systems after checking their function and asset dependencies. Exclude map-exclusive Nacht and Der Eisendrache quests, set pieces, items, audio, UI, structures, and configuration. Shared folders and generated namespaces must be audited as well as map-specific folders.

Persistent markers and their configuration are authoritative. Runtime entities should be reconstructed through their owning module's initialization. Root lifecycle functions orchestrate responsibilities; generated model files contain presentation output rather than handwritten gameplay.

Blockbench projects and authoring workspaces are excluded. Exported models and animation functions required by core are runtime inputs and may be tracked. Document their provenance and any external regeneration requirements. Keep portable, maintained generators and validators with the component that owns their output.

## Packaging contract

Package explicit runtime roots rather than the entire development checkout. Exports must exclude repository metadata, local settings, development plans, assistant instructions/caches, Blockbench sources, and map-specific content. Git ignore rules alone do not filter filesystem-based packaging.

Required runtime binaries such as PNG, OGG, and NBT files remain versioned. Preserve build inputs such as lockfiles and the Gradle wrapper. Build output, downloaded dependencies, local worlds, test servers, and exported archives do not belong in source commits.

## Documentation and validation

Each component owns documentation for its interfaces, requirements, and validation. Keep component links usable in standalone clones. The parent [development workflow](docs/development.md) explains coordination and release ordering.

Before migration is accepted, validate datapack commands and references, parse JSON, verify resource and structure dependencies, and run isolated load/reset/gameplay checks. Verify Manager packaging and real-client presentation separately. A successful Git checkout is not a substitute for gameplay or VR testing.
