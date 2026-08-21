# STORM v2 case file schema

One JSON file per 999 call. The file is the ground truth: the AI caller roleplays from it and never invents beyond it, the scripted fallback matches against it, and QA scores the player's Call Card against it. Nothing the caller model says can change the truth.

## Top level

- `id`: stable kebab-case id, matches the filename.
- `category`: grouping key (domestic, burglary, roads, retail, asb, missing, civil, ...).
- `title`: shown in the supervisor QA review only, never to the player mid-call.

## people (generated identities)

Declare specs, get fresh people every spawn: `"people": {"caller": {"gender": "any", "ageRange": [19, 78]}, "suspect": {"gender": "male"}, "patient": {"gender": "any", "ageRange": [13, 15], "familyOf": "caller"}}`. Gender `any` rolls per spawn; `familyOf` shares a surname (a missing child takes the calling parent's). Generated people bring matched pronoun and kinship tokens for use anywhere in text:

- Caller: `{{callerName}}`, `{{callerFirst}}`, `{{callerAge}}`, `{{c_he}}`, `{{c_him}}`, `{{c_his}}`, `{{c_man}}`, `{{c_mother}}` (mother/father), `{{c_mum}}` (mum/dad)
- Suspect: `{{suspectName}}`, `{{suspectFirst}}`, `{{suspectAge}}`, `{{suspectDesc}}` (gender-consistent generated description), `{{s_he}}`, `{{s_him}}`, `{{s_his}}`, `{{s_man}}`, `{{s_male}}`
- Patient: `{{patientName}}`, `{{patientFirst}}`, `{{patientAge}}`, `{{p_he}}`, `{{p_him}}`, `{{p_his}}`, `{{p_boy}}` (boy/girl), `{{p_son}}` (son/daughter)
- Vehicle: `{{vehicleReg}}` (current-format UK plate), `{{vehicleDesc}}` (colour + make)
- Capitalised variants work automatically: `{{s_He}}`, `{{p_His}}`, `{{c_Mother}}`.

Caller phone numbers are generated in the Ofcom drama ranges (+44 7700 900xxx mobile, +44 1632 960xxx landline): real UK format, never a real person's number. Only pin `caller.name`/`caller.phone` in the file when a case genuinely needs one fixed person; if the prose uses gendered pronouns for a role, fix that role's gender to match or tokenise the pronouns.

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

## deployment

What the job needs on scene before it can be closed. Mirrored from this
call's MissionChief UK counterpart, so the resourcing decision a player
makes here matches the one that game asks for.

- `mcId`: the MissionChief UK mission id this mirrors. Keep it; it is how
  a requirement set is traced back to its source.
- `vehicles[]`: each line is `{ class, count }`, optionally `chance`
  (0..1) for requirements that only appear on some spawns. The roll
  happens once, when the incident is created.
- `personnel[]` (optional): `{ kind, count }` headcounts counted across
  every crew on scene, on top of the vehicle lines.
- `prisonersMax` (optional): most prisoners this job can produce. They
  need cell capacity present on scene to leave for custody.
- `durationMin`: working time on scene once the whole set is assembled.

Vehicle classes: `patrol` (IRV/JRU), `patrol_or_arv`, `arv`, `traffic`
(Traffic Car / Armed Traffic Car), `dsu` (Dog Support Unit / MDC),
`psu` (PSU Carrier), `mounted`, `eiu` (rail), `helicopter` (NPAS),
`heli_or_drone`.

Personnel kinds: `armed` (AFOs, and they must arrive in an armed
vehicle), `po1` and `po2` (Public Order Levels 1 and 2), `sergeant`,
`inspector`, `officers` (a plain headcount).

An incident resolves only when every line is satisfied by units actually
on scene and held there for `durationMin`. Losing cover resets the
timer. Anything a call needs must be reachable by the force fleet; the
coverage tool in the game server proves this for every active call.

```json
"deployment": {
  "mcId": 92,
  "vehicles": [
    { "class": "patrol", "count": 6 },
    { "class": "dsu", "count": 1 }
  ],
  "personnel": [{ "kind": "armed", "count": 4 }],
  "prisonersMax": 1,
  "durationMin": 25
}
```

## Authoring rules

- Grade 5 cases (no policing purpose) are as valuable as Grade 1s; the game teaches grading by contrast.
- Never use M dashes.
- Keep beats spoken and short; nobody on a 999 call talks in paragraphs.
- UK realities only: real road naming styles, plausible Sussex geography, UK registrations (two letters, two digits, three letters).
