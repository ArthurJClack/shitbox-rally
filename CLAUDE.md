# Shitbox Rally — notes for Claude

Rally game: old cars, heavy physics sim, procedurally generated stages, PS1/PS2 look (inspired by TXR).
Godot 4.7, GDScript, Jolt physics. Main scene `scenes/menu.tscn`. Full feature docs are in `README.md` — read it first.

## Layout
- `src/autoload/` — `Settings` (user://settings.cfg), `Retro` (PS1/PS2 material swap + post pass), `InputManager` (bindings, steering curve, profiles), `Game`.
- `src/car/` — `car.gd` (RigidBody3D), `wheel.gd` (raycast suspension, tyre model, surface/feature lookup), `drivetrain.gd`, `engine_sound.gd`, `car_sfx.gd` (tyres, brakes, stones, water, particles), `car_camera.gd` (views + g-force effects).
- `src/track/` — `track_generator.gd` (centre line, sections, terrain, props), `road_detail.gd` (width profile, high-res road mesh, potholes/patches/paint/cracks/tar snakes/ice/puddles/fords/ponds, wheel feature queries, spills, pace-note calls), `scenery.gd` (fields, buildings, fences, far terrain), `ground_map.gd` (deformable loose layer, thrown stones that bounce off cars), `pace_notes.gd`, `track_params.gd`.
- `src/data/surfaces.gd` — all surface definitions. `IDS` order is stored in GroundMap cells: only append.
- `src/render/` — `psx_post.gdshader`, `water.gdshader` (MGS2/3-style water).
- `src/ui/` — menus, HUD, `loading_screen.gd` (tips/jokes + biome screenshots).
- `assets/models/props/<id>/` — vegetation/rock variants (OBJ + PNG + tscn); `TrackGenerator.prop_variants()` picks up to 5 per stage.
- `assets/audio/` — engine loops (Engine Simulator), tyre recordings, synthesised brake/ding/water sounds (`tools/sfx/synth_sfx.py`).
- `src/tools/` — `smoke_test.tscn` (every car drives a stage), `generate_placeholders.gd`, `capture_loading_screens.tscn`.

## Conventions / gotchas
- The project treats "type inferred from Variant" as an error: values from Dictionaries / untyped vars need explicit types (`var x: float = d.x`, not `var x := d.x`).
- A new `class_name` isn't seen until the editor rescans; preferring `preload("res://...")` for new scripts (e.g. `road_detail.gd`, `loading_screen.gd`) avoids "Could not find type" errors.
- `--script` mode doesn't load autoloads; test with scenes (`godot --headless --path . res://src/tools/smoke_test.tscn --fixed-fps 120`).
- Effect WAVs should import with `compress/mode=0` (PCM) so `car_sfx.gd` can loudness-normalise them.
- Road features live in road coordinates (s along the centre line, x lateral). Rates per surface are the `WEAR` table in `road_detail.gd`. Real road edges: `gen.detail.edge(i, side)`; `half_width` is nominal, `max_half_width` the widest.
- Materials: plain `StandardMaterial3D`s are auto-converted by `Retro`; alpha-scissor is supported, alpha-blend / shader materials are left alone (add `retro_skip` meta to opt out).

## Recent work (Oct 2026, done in Claude Cowork)
Vegetation/rock packs replaced placeholders; relaxed steering centre (Settings → Gameplay → Steering); loading screen with tips/jokes and generated-landscape shots; lower-pitched tyre scrub; brake / handbrake / underbody-ding / water sounds; particles and thrown stones collide with the car; 14 surfaces (old/washed/broken tarmac, packed/hard-packed gravel, packed snow…); road detail (width variation, potholes, patches, lane separations, paint, cracks, tar snakes, ice, puddles, water splashes, washboard, cattle grids, grass strips, spills) with pace-note calls; MGS-style water (puddles, fords, ponds); camera g-force effects (Settings → Display).

## Open items
- Fill in licences for the vegetation/rock packs in `assets/models/props/CREDITS.txt`.
- Old placeholder `tree_pine/tree_broadleaf/tree_dead/bush/rock.tscn` in `assets/models/props/` are unused and can be deleted.
- Stage generation is ~3–4 s for 3 km on a slow machine; terrain + scenery are the biggest costs.
- Not yet a git repo — see README / ask to set up a private GitHub repo.
