# storm-content

Live content for STORM (UK 999 dispatch simulation). The game fetches this repo on start and every hour, validates every file, and swaps the new content in; a broken file only knocks out that one call.

## Layout

- `calls/active/` spawns naturally, always. Sub-folders per category (`rtc/`, `fire/`, `medical/`, `police/`, `rescue/`, `hazmat/`) are just for tidiness, the game reads recursively.
- `calls/events/` spawns only while the file's date window is open (`from` / `until`, or `weightBy.month`). Rotate by changing the dates.
- `calls/retired/` known to the game (catalogue, old saves) but never spawns. Retire a call by moving its file here.
- `templates/call-template.json` the annotated template. Copy it, fill it in, delete or keep the `_` notes.
- `manifest.json` bump `version` after any change so players pick it up.

## Naming

One call per file. File name = the `id` inside it: lowercase, underscores, no spaces, category first: `rtc_multi_vehicle.json`, `fire_chimney.json`, `medical_cardiac_arrest.json`, `police_domestic_dispute.json`, `rescue_person_in_water.json`. `id` must be unique across all three folders; it is what saves and scoring use, so a call keeps its identity when moved between folders.

## How the game reads this repo

STORM v0.0.4 and later fetch `manifest.json` from the `main` branch on start and once an hour, list every `calls/**/*.json` (via the GitHub tree listing, or an optional `files` array in `manifest.json` if you ever want to pin the list), download and validate each file on its own, and keep an offline copy under `content/remote/` next to the game. In the game, Settings > Live call content shows every file with the reason it was skipped if it failed validation, a "Check for new calls now" button, and a "Ring now" button per call for testing a new file straight away.

What the game currently uses from a file: `id`, `name`, `category`, `enabled`, `weight`, `grade`, `onSceneMinutes`, `reward`, `locationTypes` (unknown types are ignored, a file with none left is skipped), `units` and `patients` (summed into on-scene requirements per service), `records`, `caller.openings` and `dialogue.<promptId>.answers`. Everything else in the template (weightBy, desks, duplicateCallers, escalations, majorIncident, transport...) is ignored for now and safe to fill in ahead of time.

## The MissionChief UK derived set (manifest version 4)

`calls/active/` now holds 540 land based calls generated from the MissionChief UK mission list (every land mission: fire, medical, police, RTC, rescue, hazmat; coastguard, ocean/coastal rescue, airfield, mountain rescue and planned policing tasks are left out). Each file's `notes` quotes the source mission (vehicles, personnel, patients, prisoners, escalation, POI) and its `tags` end with `mc_<id>`, so a file can always be traced back. Mechanics come from the source (units, patients, grade from the triage code, weight from the station prerequisites, reward = credits / 10); the caller dialogue, hazards and scene reports were written per call.

Unit keys used in `units` across the set: `irv, traffic_car, arv, dog_unit, cell_van, mounted_unit, eod_unit, drone_unit, single_pump, dual_pump, aerial, rescue_unit, water_carrier, basu, hazmat_unit, iccu, bulk_foam_unit, welfare_unit, fire_officer, otl, ambulance, rrv, hart, hems`. The game maps each to the service it counts against on scene; unknown keys are ignored, so new keys can be used ahead of the game catching up.

The game reads this repo on start and once an hour; Settings > Live call content lists every file and the reason any file was skipped. `rescue_vessel_capsized` now uses `coastline` / `harbour` (the only water location types the game has).

## Vehicles (reference)

`vehicles/uk-fleet.json` is the game's built-in UK fleet (every land vehicle type from the MissionChief UK list that real UK services run, plus the desk-only air units), with crew ranges, real purchase prices in pounds, the training each crew member holds and a note on which services use it; `vehicles/training.json` is the training catalogue; `templates/vehicle-template.json` documents the fields. The game does not read these from the repo (its list is `content/vehicles.json` next to the game, seeded from the same data); they are here so call files and the game agree on unit keys.


## Layout (since the v0.0.8 rebuild)

- `calls/active/` - live case files in the new format (police service, one JSON per 999 call). Listed in `calls/manifest.json`, synced by the game on boot.
- `calls/retired/` - the legacy pre-rebuild library (fire, medical, hazmat, rescue, rtc, and the original police files the active set was converted from). Kept for reference and future conversion; also serves any old v0.0.7 installs via the root `manifest.json`.
- `calls/events/` - seasonal packs (empty for now).
- `templates/call-template.json` + `calls/CASE_SCHEMA.md` - how to author a new case.
- `retired/vehicles/` - the old vehicle and training reference data. Vehicles, crews and training are built into the game itself since the rebuild, so this data is retired rather than consumed.
