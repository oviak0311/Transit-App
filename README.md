# EasyTransit
AI-powered smart transit navigation for Markham, Ontario

## Rider profile and language

`/app` starts with three-step onboarding, with optional personalization, before
mounting the map. A returning rider with a completed remembered profile skips
it. The app menu opens the complete editor with draft/save/cancel semantics.
The landing page keeps **Plan my journey** primary and QR phone handoff secondary.

- UI locales: English (`en-CA`), French (`fr-CA`), Simplified Chinese (`zh-Hans`).
  Language detection happens after mount; a valid stored locale wins.
  Traditional Chinese is not silently mapped to Simplified.
- Locale is independent of the Markham, Toronto, Montreal and Vancouver planning
  presets. Centroids are search bias/empty-map centres, not a rider's location
  or guaranteed coverage. Estimated times use the labelled planning-area time
  zone, even for existing trips. Fares remain explicitly **CAD**. Distance units
  and clock formats are selectable.
- Profile schema v1 uses only `transitiq.rider-profile`, separately from journeys.
  It tracks `schemaVersion`, `updatedAt`, completion and typed preferences.
  Validation safely rejects invalid, corrupt and unknown-version data.
- **Remember on this device is off by default.** There is no automatic first-run
  write. Without opt-in, even completion lives only in memory until reload.
  Opt-out removes a previously remembered profile. Storage denial/quota errors
  show a localized warning and leave the session usable. Confirmed reset deletes
  only the profile key and returns to onboarding. If browser storage blocks
  deletion, the warning explains how to clear site data when available.
- No account, analytics, diagnoses, demographics, address history, trip endpoints
  or geolocation are stored in the profile. Location is requested only by the
  location button. Local profiles are visible to anyone using the same browser;
  this is not encrypted account storage.

### Applied now vs future model inputs

Available modes and priority apply to routing. The existing request wiring also
sends `max_walking_meters` for a walking requirement, `preferred_walking_meters`
for a walking preference, and `requires_step_free` for access needs. These
walking/access edits replan an existing journey. This preserves API compatibility;
it does not change backend or model behavior. Quiet streets and avoid hills
are also requested; these cycling preferences depend on Valhalla.

A request is not a guarantee of verified access or satisfied requirements.
The backend may fall back to showing all routes with warnings, including routes
that exceed recorded limits. Onboarding and results prominently advise checking
provider details and confirming access with operators, including for a zero-metre
walking limit. Original provider warnings remain visibly labelled English.

Cycling confidence/bike type, rain/cold/dark comfort and routine
purpose/arrival/days are local future-model foundation fields only, not sent to
routing, verified or enforced. Locale, units, appearance, region and clock
format are presentation/search settings. Changing them or future-model fields
does not replan an existing journey.

Provider directions, explanations and warnings remain original English under a
labelled `lang="en"` boundary; the main recommendation uses structured route data.
Source research and police/legal content retain labelled English boundaries.
Official service, stop and headsign names are not translated. API failures show
localized app errors rather than raw provider messages.

### Verification (no new dependencies)

```sh
npm test
npx --no-install tsc --noEmit --incremental false
npm run lint
```

Node tests transpile pure TypeScript using the existing TypeScript dev dependency.
The checked-in smoke test uses an already-running server and externally available
Playwright module/browser:

```sh
PLAYWRIGHT_MODULE=/absolute/path/to/playwright \
PLAYWRIGHT_EXECUTABLE_PATH=/absolute/path/to/chromium \
SMOKE_BASE_URL=http://localhost:3000 npm run test:browser
```

It mocks every transit API, tile and external HTTP request. Coverage includes
onboarding, optional fields, persistence/opt-out/reset/corruption, blocked storage,
no automatic location, manual search after location denial, routing payloads,
walking/access versus display-only request dependencies, English boundaries,
short CJK/IME place queries, coordinate resolution and 320px/desktop layouts.
It does not test provider coverage or routing correctness.
# Transit-App
