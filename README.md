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
