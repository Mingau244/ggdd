## What this project is

A **bullet-hell MMO roguelike**: Godot 4.7 mono C# client (`client/`) + SpacetimeDB Rust server module (`server/spacetimedb/src/`). Repo root is the parent of `docs/`'s parent — all paths below are relative to repo root. The client is **fully compositional** (entity/component architecture modeled on `code_examples/comedot`); the server mirrors that compositionally: one table per concern = a component, ids like `profile_id`/`enemy_id` join a logical entity's rows across tables, and "archetype helpers" bundle the rows of one entity.

The docs you're writing live in `docs/Technical Design Docs/Description/` and are the narrative, learn-from-scratch companion to the terse maintainer references (`CLAUDE.md` at repo root, `client/CLAUDE.md`, `server/CLAUDE.md`). The docs folder is an Obsidian vault with the `sync-embeds` and `block-link-plus` community plugins installed — the conventions below depend on both. The vault also has the local `vscode-editor` fork installed, whose "Generate flowcharts from code" feature produces the `.canvas` flowcharts under `flowcharts/` — these docs link to them (see §"Flowchart integration").

## Hard rules for every session

1. **The code is the only source of truth.** `CLAUDE.md` files are accurate jump-off points (use their module maps to find files fast), but verify every claim against the actual code before writing it. Where they disagree, code wins.
2. **Never read `client/Scripts/module_bindings/`** — generated SpacetimeDB bindings.
3. **Never read `flowcharts/Subflowcharts/` wholesale.** Use the doc's main canvas in `flowcharts/Mains/` plus at most a handful of targeted subflowchart canvases. Link canvases by **file path** — canvas node ids are re-rolled randomly on every regeneration, so never deep-link to a node.
4. **Document what the code does, not what it's planned to do.** Flag TODOs/stubs/known bugs explicitly in the doc's "Known gaps" section and inline where relevant (e.g. "`LobbyGui` is scene navigation only — all profile reducer calls live in `LobbyComponent`").
5. **Don't restate the CLAUDE.md files.** Link to them; these docs are the narrative complement.
6. **Audience**: intermediate programmer, brand new to Godot, C#, Rust, and SpacetimeDB. Explain unfamiliar concepts (reducers, views, subscriptions, node trees, scenes, `.tscn` files) briefly inline or with an analogy. Don't explain general programming ideas (loops, classes, dictionaries).
7. **Style**: dense causal prose — "X does Y *because* Z". One step = one cause-and-effect beat. Every mechanism claim should name the file/function that implements it.
8. Each session updates the **Status** column of the table in `01 Roadmap.md` before finishing.

## Doc lineup & session order

Docs are written **in session order** (numeric). Each session writes its system doc(s) **and** appends its numbered steps to `00 End-to-End Timeline Flowchart.md` in the same session, so the sync-embeds in the system doc resolve. Sessions 2+ also produce their **flowchart main canvas** in the same session (see §"Flowchart integration").

| Session | Writes | Appends to 00 (prefixes) | Code to read |
| ------- | ------ | ------------------------ | ------------ |
| 1 | `00 End-to-End Timeline Flowchart.md` scaffold (headers only), `01 Roadmap.md` | — | — |
| 2 | `02 Architecture & Sync Model.md` | `boot`, `conn` | `client/sstdbsdk/DatabaseConnector.cs`, `client/sstdbsdk/TableSubscriber.cs`, `client/sstdbsdk/TableBinderComponent.cs`, `client/project.godot` (autoloads), `server/spacetimedb/src/lib.rs`, `server/spacetimedb/src/main/lifecycle.rs` |
| 3 | `03 Entity & Component Framework.md` | — (atemporal; composition map) | `client/Scripts/Components/` framework files (`Component.cs`, `IComponent.cs`, `AreaComponent.cs`, `Node2DComponent.cs`, `Entities/IEntity.cs`, `Entity.cs`, `EntityRegistry.cs`), every entity `.tscn` (`main`, `local_player`, `non_local_player`, `default_enemy`, `drop`, `bullet_manager`, `world_3d`), server `*/tables.rs` files, archetype helpers in `player/methods.rs` + `enemy/methods.rs` |
| 4 | `04 Player System.md` (lobby → profile → join → leave, scaffolding) | `lobby`, `join` | `server/.../player/reducers.rs` (`set_username`, `create_profile`, `delete_profile`, `join_world`, `leave_world`), `player/methods.rs` (`try_scaffold_profile`/`teardown_profile`, `recompute_stats`), `main/lifecycle.rs`; client `LobbyGui.cs`, `LobbyComponent.cs`, `profile_panel.tscn` |
| 5 | `05 Movement & Position Sync.md` | `move` | `server/.../player/reducers.rs` (`report_movement`), `world/wrap.rs`, `world/hex.rs`; client `PositionSyncComponent.cs`, `InterpolationComponent.cs`, `TorusMath.cs`, `RemotePlayer.cs`, `LocalPlayer.cs` |
| 6 | `06 World & Terrain.md` | `terrain` | `server/.../world/` (all of it: `tables.rs` MapConfig/lap vectors, `hex.rs`, `wrap.rs`, `aoi.rs`, `prng.rs`, `noise.rs`, `terrain/` pipeline, `reducers.rs` building, `views.rs`); client `Components/Terrain/` stack (`TerrainComponent`, `TileComponent`, four layer components), `HexGridOverlayComponent.cs`, `HexGridOverlay3DComponent.cs` |
| 7 | `07 Enemy System.md` | `enemy` | `server/.../enemy/` (def/instance tables, `tick_enemy_spawn`/`tick_enemy_behavior`, `methods.rs` aggro/phases/movement/archetype helpers); client `Enemy.cs`, `default_enemy.tscn`, `BulletSpawnerComponent.cs`, `bullet_manager.tscn` |
| 8 | `08 Combat & Damage Math.md` | `combat` | `server/.../combat/mod.rs`, `player/reducers.rs` (`report_hit`), `enemy/reducers.rs` (`report_enemy_hit`); client `CombatComponent.cs`, `DamageComponent.cs`, `DamageReceivingComponent.cs`, `hit_zone.tscn`, `HealthComponent.cs`, `BulletHitRouterComponent.cs` |
| 9 | `09 Items, Inventory & Enchantments.md` | `equip`, `loot` | `server/.../item/` (tables, admin reducers, seeds, views), `player/` inventory reducers (`swap_slots`, `use_item`, `drop_item`, `pickup_drop`, `apply_enchantment`, `remove_enchantment`, `tick_active_consumable_effects`) + `recompute_stats`; client `InventoryComponent.cs`, `SlotComponent.cs`, `ItemSidebarComponent.cs`, `StatsSidebarComponent.cs`, `LocalPlayerInventoryComponent.cs`, `PickupComponent.cs`, `inventory_panel.tscn`, `enchantment_row.tscn` |
| 10 | `10 Camera & Presentation.md` | `camera` | client `CameraRigComponent.cs`, `Camera2DPresenterComponent.cs`, `World3DComponent.cs`, phantom-camera wiring in `main.tscn` / `local_player.tscn` / `world_3d.tscn`, `CharacterModel3D.cs` |
| 11 | `11 Admin, Debug & World Generation.md` | `admin`, `end` | `server/.../main/admin.rs`, `main/debug.rs`, `main/seeds.rs`, `main/global.rs`, `world/reducers.rs` + `terrain/mod.rs` (`internal_generate_world_proc`/`internal_generate_world_manual`), `main/lifecycle.rs` (`init`, `client_disconnected` teardown); client `DebugOverlay.cs` |

Aspirational systems described in the Game Design Docs but absent from the code (guild territory PvP, base-building beyond `place_building`/`remove_building`, biomes as gameplay, most of enchantment breadth, the slash/bullet-despawn protocol in `server/spacetimedb/src/plan.md`) stay **out** of these docs; where a doc touches such an edge it says so explicitly.

> **Renumbering note**: the previous draft numbered both the roadmap and the architecture doc "01". The lineup above resolves the collision: `01` is the roadmap, system docs run `02`–`11`. Old cross-references like `[[07 Combat & Damage Math]]` are now `[[08 Combat & Damage Math]]` — no written docs exist yet, so nothing else needs fixing.

## System doc schema (docs 02–11)

Every system doc has exactly these sections, in order:

1. **Assumed knowledge** — wikilinks to the prerequisite docs (e.g. every doc assumes `[[02 Architecture & Sync Model]]`; docs 04–11 assume `[[03 Entity & Component Framework]]`).
2. **The 30-second version** — a short paragraph: what this system is and how it works, no detail.
3. **Flowcharts** — wikilinks to the generated `.canvas` flowcharts for this system: the doc's own main canvas in `flowcharts/Mains/` plus 1–3 deep-dive subflowchart canvases under `flowcharts/Subflowcharts/`, all linked by path (never by node id).
4. **System flowchart** — the doc's own numbered steps, rendered as sync-embed blocks transcluded from `00 End-to-End Timeline Flowchart.md` (see mechanical conventions). Numbering restarts at `01.` within each doc regardless of the anchor's own number in 00.
   - **Exception — doc 03** is atemporal (framework structure, not a runtime sequence). Its fourth section is instead a **composition map**: for each logical entity (local player, remote player, enemy, drop, bullet manager, game manager), the client scene root + its component children (with their `TableBinderComponent` children as wired in the `.tscn`), and the server tables joined by its entity id + the archetype helper that bundles them.
5. **Main body** — the full documentation, covering everything in the doc's lineup row above. Where content duplicates a timeline step, sync-embed it from 00 instead of re-typing; free prose is for everything the timeline doesn't say (why, invariants, edge cases, formulas).
6. **Known gaps / stubs** — explicit list of TODOs, stubs, unwired UI, and known bugs in this system's code (see the verified gap lists in §"Background" below). If none, omit the section.
7. **Where to go next** — 1–3 sentences pointing to the next doc(s) to read.

## `01 Roadmap.md` (session 1)

Contains: what these docs are and who they're for; the conventions section (copy §"Mechanical conventions" below into it, adapted as observer-facing guidance); the reading-order/status table (doc #, title, status, one-line coverage summary — mirror the lineup table above, statuses all start blank and get filled in by later sessions); a link to the global `flowcharts/main.canvas` as the visual overview; and the note about aspirational systems staying out. Status values: blank / `done`.

## `00 End-to-End Timeline Flowchart.md` conventions

Session 1 writes the scaffold with these exact headers (content gets appended by the sessions per the lineup table):

```
# Boot & Connection          ← prefixes: boot, conn
# Lobby & Profiles           ← prefix: lobby
# Joining the World          ← prefix: join
# Movement & Position Sync   ← prefix: move
# Terrain & World Streaming  ← prefix: terrain
# Enemies & AI               ← prefix: enemy
# Combat & Damage            ← prefix: combat
# Inventory, Items & Enchantments  ← prefixes: equip, loot
# Camera & Presentation      ← prefix: camera
# Admin & Debug              ← prefix: admin
# Disconnect & Teardown      ← prefix: end
```

Step format within each section:

- Numbered prose steps starting from 1 within each section.
- Each step ends with a `^prefix-N` block anchor. Each prefix restarts its own count at 1 (e.g. `^boot-1`, `^boot-2`, `^conn-1`…).
- Steps cross-refer to other steps by anchor name ("the subscription opened in join-1"), never by bare number.
- Where a step's details belong to another doc, say so and link it ("the damage formula is [[08 Combat & Damage Math|08's]] concern").

## Flowchart integration (the `vscode-editor` fork's canvas flowcharts)

The plugin's "Generate flowcharts from code" feature (right-click a folder in Obsidian's file explorer, or drive it headlessly via `plugin.flowchart.*` eval API) generates a deterministic tree of `.canvas` files under `flowcharts/Subflowcharts/<client|server>_subfolder/…` at three granularity levels (`_symbol`, `_codefile`, `_subfolder`). "Generate main flowchart from subflowchart metadata" composes `flowcharts/main.canvas` from every subflowchart tagged `includeInMainFlowchart` in its canvas frontmatter, adding aggregated cross-file edges. Key mechanics:

- **Stable across regeneration**: canvas file **paths** and file-node `file`+`subpath` identity. **Not stable**: node ids (random each run) and layout. Hence hard rule 3: link by path only.
- Include flags live in each canvas's own `metadata.frontmatter.tags` and survive code regeneration — but there is only **one** flag dimension, so curating flags for a second main canvas clobbers the first set. Plan for re-curation (see below) until the plugin supports named main sets.

### Per-session flowchart steps (sessions 2–11, before writing the doc)

1. **Regenerate** flowcharts for both code roots (`client/` and `server/spacetimedb/`) so the canvases reflect the code as of this session. Delete any stale canvases the run reports (stale detection is report-only; stale canvases can still carry `includeInMainFlowchart` and pollute mains).
2. **Curate**: flag the subflowchart canvases covering this doc's system with `includeInMainFlowchart` (bulk "Include subset … Depth N" menu, or the eval API).
3. **Generate the doc's main** via "Generate main flowchart from subflowchart metadata", then rename `flowcharts/main.canvas` → `flowcharts/Mains/<doc-slug>.canvas` (e.g. `flowcharts/Mains/08-combat.canvas`). Moving it out of `Subflowcharts/` keeps later mains from nesting it.
4. Repeat 2–3 for any earlier docs whose mains need refreshing after the regeneration in step 1 (flags must be re-curated per main — this is the known finicky part).
5. **Finish with the global main**: re-flag the top-level set (typically the two `<root>_subfolder.canvas` files) and generate `flowcharts/main.canvas`; `01 Roadmap.md` links to it.
6. Fill in the doc's **Flowcharts** section (schema §3) with the links.

## Mechanical conventions (exact syntax — follow precisely)

**Wikilinks to code** (block-link-plus line-search syntax):

```
[[filename##text on the target line|display text]]
```

- Bare filename, no path — the plugin searches the vault (e.g. `[[lifecycle.rs##pub fn init|init]]`, `[[GameManager.cs##public override void _Ready|_Ready]]`).
- Search text must be unique within that file — use the function/struct signature line.
- Every timeline step and every flowchart entry links to code wherever possible.

**Wikilinks to canvas flowcharts** are ordinary wikilinks including the folder path, e.g. `[[flowcharts/Mains/08-combat.canvas|Combat flowchart]]`.

**Sync-embed blocks** (sync-embeds plugin) — for any content duplicated between docs, transclude from the single source (the timeline doc) instead of re-typing:

````
```sync
![[00 End-to-End Timeline Flowchart#^move-2{seamless:true,title:false,marker:01.}]]
```
````

- `marker:NN.` — the step number *as it should render in the embedding doc*, zero-padded to two digits. System-doc flowcharts number from `01.` regardless of the source step's number in 00.

**Section references between docs** use Obsidian wikilinks: `[[04 Player System]]`, or with display text `[[08 Combat & Damage Math|08]]`.

## Per-session verification checklist (run before marking a doc done)

1. Every `^anchor` referenced by the session's sync-embeds exists in `00 End-to-End Timeline Flowchart.md`.
2. Spot-check every `[[file##text|…]]` code link: the search text appears (uniquely) in that code file.
3. Every class, table, reducer, view, and function named in the doc exists in the code today — grep for it. (The pre-refactor client had classes like `TerrainManager`, `CameraController2D`, `CameraRig`, `LocalPlayerCombat`, `LocalPlayerInventory` — these **no longer exist**; their successors are the `*Component` classes in `client/Scripts/Components/`. Never cite the old names except to say they were replaced.)
4. **Cite the live wiring site.** Binder/signal wiring lives inline in `main.tscn`, `local_player.tscn`, `default_enemy.tscn`, `non_local_player.tscn`, and `world_3d.tscn`. Seven standalone component scenes — `catalog_component.tscn`, `entity_spawner_component.tscn`, `terrain_component.tscn`, `subscription_component.tscn`, `camera_rig_component.tscn`, `camera_2d_presenter_component.tscn`, `hex_grid_overlay_component.tscn` (plus `debug_overlay.tscn`) — are **unreferenced duplicates** of what `main.tscn` declares inline. Never cite them as the wiring site; mention them only as drift hazards.
5. The doc's **Flowcharts** section links resolve, and every linked canvas was regenerated this session (paths stable, contents fresh).
6. `01 Roadmap.md` status table updated for the doc(s) written this session.

## Background: what changed since the wiped docs (so you don't resurrect stale claims)

- **Client**: refactored to a fully compositional entity/component architecture. `GameManager` is now thin glue plus a static facade delegating to child components (`TableSubscriber`, `CatalogComponent`, `EntitySpawnerComponent`, `LobbyComponent`) **and to the `DatabaseConnector` autoload** — connection ownership moved out of any scene-tree component entirely (see below). Camera static singletons are gone — components are reached via `GameManager.GetComponent<T>()`. `TerrainManager.cs` was replaced by the `Components/Terrain/` stack (`TerrainComponent` + pooled `TileComponent`s + four layer components). Old camera/player-combat/inventory classes became `CameraRigComponent`, `Camera2DPresenterComponent`, `World3DComponent`, `CombatComponent`, `InventoryComponent`, `SlotComponent`, `ItemSidebarComponent`, `StatsSidebarComponent`.
- **A later refactor moved connection/subscription ownership into a first-party `client/sstdbsdk/` folder** (sibling of `client/addons/`, which really is third-party-only: blastbullets2d, open_godot_mcp, phantom_camera). `sstdbsdk` is own code and currently **untracked in git** — it should be committed; don't treat it as vendored or generated. It has three pieces: `DatabaseConnector.cs`, an **autoload singleton** (`project.godot`, `DatabaseConnector.Instance`) owning the `DbConnection` and per-frame `FrameTick()`; `TableSubscriber.cs`, a `Component` child of `Main` owning the subscription waves (`BaseTables`/`LobbyTables`/`GameTables`) and the `MapConfig` lap-vector mirror; and `TableBinderComponent.cs`, a reflection-based `[Tool] Component` that binds one subscribed table and re-exposes its row events as `RowInserted`/`RowUpdated`/`RowDeleted` Godot signals — now the standard way almost every component consumes table rows, wired per-instance in `.tscn` files rather than hand-hooked in code. Docs 02 and 03 need to cover this; the pattern produced these `LocalPlayer` children: `PositionSyncComponent` (note: **not** `LocalPositionSyncComponent` — it handles both inbound position rows and the outbound `ReportMovement` timer), `LocalPlayerDataComponent`, `LocalPlayerInventoryComponent`, `LocalPlayerProfileComponent`.
- **Server**: new `combat/` module owns the whole damage pipeline; `report_hit`/`report_enemy_hit` just resolve base damage and delegate. New `Enchantment` table + `apply_enchantment`/`remove_enchantment` reducers + enchantment UI in `ItemSidebarComponent`. `world/`'s old `methods.rs` split into focused files (`prng`/`noise`/`hex`/`wrap`/`aoi`) plus a composable `terrain/` generation pipeline. Archetype helpers (`try_scaffold_profile`/`teardown_profile`, `spawn_enemy_archetype`/`despawn_enemy_archetype`) bundle each entity's rows. `main/global.rs` centralizes tunable constants. Every `views.rs` is now implemented (18 views, no empty placeholders). Lifecycle hooks are named `client_connected`/`client_disconnected` (not `identity_connected`).

## Known gaps to document (verified against the code — distribute to the relevant docs' "Known gaps" sections)

**Server:**
- `src/plan.md`'s slash/bullet-despawn protocol is entirely unimplemented: no `BulletDespawnEvent` table, no `slash_bullet` reducer, `report_hit` still takes the old single `SequenceStepRef` arg. `combat/mod.rs` comments reference these as "upcoming callers".
- `LootDrop.expires_at` is written at insert time but never enforced — no expiry sweeper exists.
- `nearby_remote_players_profiles` is **not** AOI-filtered despite the name — it semijoins profiles of *all* logged-in players.
- `PlayerDamageOutcome`/`EnemyDamageOutcome` enums in `combat/mod.rs` are `#[allow(dead_code)]`; both callers discard them.
- `client_connected` mishandles the already-`LoggedInPlayer` case (deletes/reinserts instead of rejecting) — flagged as a bug in the code itself.
- `PlayerPosition`/`PlayerChunk` rows persist while logged out ("ghosts"); enemy sim guards against them via `logged_in_player` checks rather than cleaning up.
- Stale comment: `world/hex.rs` references `hex_grid_overlay.gd`; the real client implementation is C# (`HexGridOverlay*Component.cs`).

**Client:**
- Leftover debug `GD.Print`s in `TableBinderComponent.Bind/HandleInsert`, `TerrainComponent._Ready/OnTileRowInserted`, and `EntitySpawnerComponent.OnDropInsert`.
- The `3D` CanvasLayer in `main.tscn` is `visible = false` — the 3D backdrop (`World3DComponent`) is currently disabled at runtime, and the hidden `HexGridOverlayComponent` is still subscribed via its binder.
- `local_player.tscn` has a disabled legacy `Camera` (Camera2D) child superseded by the phantom-camera setup.
- `LobbyGui` ServerList/Settings panels are show/hide only — no server-switching or settings logic behind them.
- `SlotComponent.cs`'s docstring says it's declared 24 times; `inventory_panel.tscn` actually has 29.
- The seven unreferenced duplicate component scenes listed in checklist item 4.
