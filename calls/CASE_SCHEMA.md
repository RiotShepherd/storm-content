# STORM v2 case file schema

One JSON file per 999 call. The file is the ground truth: the AI caller roleplays from it and never invents beyond it, the scripted fallback matches against it, and QA scores the player's Call Card against it. Nothing the caller model says can change the truth.

## Top level

- `id`: stable kebab-case id, matches the filename.
- `category`: grouping key (domestic, burglary, roads, retail, asb, missing, civil, ...).
- `title`: shown in the supervisor QA review only, never to the player mid-call.

## caller

- `name`, `age`, `phone`.
- `persona`: one line of voice direction, e.g. "frightened, whispering, close to tears". This drives the AI caller's whole register, write it like a stage direction.
- `callerIs`: `victim` | `witness` | `third_party` | `involved`.
- `openingLine`: what they say the moment the call connects, before any question.
- `dontKnowLines`: in-character lines for questions about `unknownTopics`.

## locationTypes (the normal way)

Real place classes this call can spawn at: `train_station`, `bus_station`, `tram_stop`, `supermarket`, `shopping_centre`, `retail_park`, `pub`, `high_street`, `pedestrian_street`, `car_park`, `park`, `school`, `university`, `stadium`, `office`, `industrial_estate`, `residential_road`, `house`, `flats`, `a_road`, `b_road`, `motorway`, `roundabout`, `bridge`, `level_crossing`, `track`. The engine resolves a REAL place live from OpenStreetMap (cached per category, offline-safe) each time the call spawns, so a Fight on Train lands at a real railway station and a domestic on a real residential street (house numbers are generated). Use `{{address}}`, `{{town}}`, `{{place}}` and `{{postcode}}` placeholders in `openingLine`, `summaryTruth` and fact `value`/`beat` strings; they are substituted at spawn.

## location (optional fixed override)

- `line`, `town`, `postcode`, `lat`, `lng`: pins the case to one exact place; normally omitted in favour of locationTypes.
- `callerKnowsPostcode`: false means QA does not expect the postcode on the card.

## incident

- `type`: the correct incident type string.
- `summaryTruth`: the full ground-truth narrative. This becomes the AI caller's system prompt core; every fact the caller can ever reveal must be in here.
- `correctGrade`: 1-5 (national grading model).
- `correctThrive`: the six booleans a competent supervisor would set.

## facts[]

Each fact is one piece of information the caller can release:

- `key`: stable id.
- `label`: how the QA debrief names it.
- `value`: the ground truth string. QA checks the player's card for this; the fact guard corrects near-miss mangling in AI output.
- `askHints[]`: lowercase substrings that mean the player asked for this (scripted matching and "did you ask" fairness in QA).
- `beat`: the caller's in-character line releasing the fact. Write it in the persona's voice.
- `spontaneous`: true if a rattled caller would blurt it unprompted.
- `critical`: true for facts that must survive verbatim (names, addresses, registrations); weighs more in QA and is fact-guarded.
- `cardField` (optional): which Call Card field this belongs in (`callerName`, `addressLine`, ...).

## unknownTopics[]

Topic substrings the caller genuinely cannot answer; triggers `dontKnowLines`.

## Authoring rules

- Grade 5 cases (no policing purpose) are as valuable as Grade 1s; the game teaches grading by contrast.
- Never use M dashes.
- Keep beats spoken and short; nobody on a 999 call talks in paragraphs.
- UK realities only: real road naming styles, plausible Sussex geography, UK registrations (two letters, two digits, three letters).
