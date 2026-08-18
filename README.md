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
