# Let Me Get In

**A normalized accessibility referential, exposed as a public API and a web app.**

[![Search a place](https://raw.githubusercontent.com/Let-Me-Get-In/.github/main/profile/assets/app-search.jpg)](https://app.letmegetin.org)

| A place, with its completeness score | The public API contract |
|---|---|
| [![Place detail](https://raw.githubusercontent.com/Let-Me-Get-In/.github/main/profile/assets/app-place.jpg)](https://app.letmegetin.org) | [![API documentation](https://raw.githubusercontent.com/Let-Me-Get-In/.github/main/profile/assets/api-docs.jpg)](https://api.letmegetin.org/docs) |

[**app.letmegetin.org**](https://app.letmegetin.org) · [**api.letmegetin.org/docs**](https://api.letmegetin.org/docs) · [**source**](https://github.com/Let-Me-Get-In/letmegetin)

---

## Why

Jaccede was the historical open database of accessible places in France. When it
went away, its data went with it. A public alternative exists today and covers many
places, but an accessibility referential that is **normalized, versioned and
queryable** is still missing — the kind of thing a third party can build on, not
just browse.

Let Me Get In rebuilds that referential from the sources that survived, and exposes
it as a read-first API.

## What is in the referential

| | |
|---|---|
| Categories | 158 |
| Criteria | 182 |
| Filters | 52 |
| Official labels | 15 |

Plus the full category-1 criteria grid. Everything is committed as JSON in
`data/referential/` and loaded by a seed command. The original raw sources are
private and **not** committed; they are only needed to regenerate the JSON.

Accessibility is recorded per handicap family — motor, hearing, vision, cognitive —
and each place carries a **completeness score**, because "unknown" and "not
accessible" are not the same answer.

## Where the data comes from

Let Me Get In does not invent accessibility data. Places come from open datasets,
each with its own licence and attribution requirements.

| Source | Licence | Role |
|---|---|---|
| [AccesLibre](https://www.data.gouv.fr/fr/datasets/acceslibre/) (`data.gouv.fr`) | Licence Ouverte / Etalab 2.0 | Primary seed, France-first |
| [OpenStreetMap](https://www.openstreetmap.org/copyright) | ODbL | Additive, opt-in |
| [Overture Maps](https://overturemaps.org) | CDLA-Permissive | Additive |
| Google Places | — | **Never used**: its terms forbid storing the data |

Imports are idempotent per `(source, source_id)`. Duplicates are linked *softly*:
rows within 100 m are compared by normalized name; a close match joins a canonical
place and is logged, a doubtful one is flagged for review. Nothing is merged
destructively, and no source overwrites another.

## What this adds

AccesLibre tells you what is recorded about a place. Let Me Get In adds the layer a
developer needs in order to **build on** those records:

**1. A typed referential, not per-source free text.** 158 categories, 182 criteria,
52 filters and 15 official labels, organized in a tree. One vocabulary, whatever
dataset the place came from — so "entrance width" is a criterion with a type and a
unit, not a sentence in a CSV column.

**2. Accessibility per handicap family, with a completeness score.** Motor, hearing,
vision, cognitive. Each place says how much of its record is actually filled in,
because *"unknown" and "not accessible" are not the same answer* — and most
directories conflate them.

**3. A versioned contract.** The path is versioned (`/api/v1`), the referential
carries a `referential_version`, and grids answer `304` through an `ETag` when
nothing changed. You can build a client that knows when its cached copy is stale.

**4. Multi-source reconciliation.** The same place can arrive from AccesLibre, OSM
and Overture; it resolves to one canonical entry with a per-source trace, instead of
three rows a consumer has to deduplicate itself.

**5. Query by what you need, not by what exists.** Search returns places filtered by
handicap and category, and each result exposes which criteria are known — so a
client can hide a place it cannot vouch for, rather than showing it as accessible.

The referential is open data. Attribution is required, and the ODbL share-alike
applies to any derived database that includes OpenStreetMap content.

## Stack

| Layer | |
|---|---|
| API | FastAPI, SQLAlchemy 2 async, Pydantic v2 |
| Data | PostgreSQL + PostGIS, Redis (referential cache, rate limits) |
| Web | Next.js (App Router), TypeScript, Tailwind, MapLibre |
| Operations | OpenTofu + Ansible, one Hetzner VPS, Compose stacks, Sentry |
| Quality | 193 tests, 85% coverage floor, E2E and axe accessibility checks against a deployed environment |

## The API

Reads are open. An optional `X-API-Key` header gets you a dedicated rate limit;
writes require a bearer token.

```bash
curl "https://api.letmegetin.org/api/v1/places?lat=48.85&lng=2.35"
```

| | |
|---|---|
| Search and details | `/api/v1/places`, `/api/v1/places/{slug}` |
| Referential | `/api/v1/categories` and the criteria tree |
| Reviews and contributions | read endpoints are public, writes are authenticated |
| Contract | [`/openapi.json`](https://api.letmegetin.org/openapi.json), Swagger at `/docs` |

The admin CLI is currently the only path that writes to the referential itself.

## Repositories

| Repository | |
|---|---|
| [`letmegetin`](https://github.com/Let-Me-Get-In/letmegetin) | API, web app, referential data — public |
| `letmegetin-config` | OpenTofu + Ansible, the VPS and its Compose stacks — private |
| `letmegetin-e2e` | End-to-end and accessibility checks against a deployed environment — private |

## Status

Working, deployed, and honest about what is missing:

- **The community layer is not built.** Spots are read from the referential; the
  contribution workflow that would let visitors add and correct places is not
  there yet.
- **One host.** Development and production run as two Compose stacks on a single
  VPS, provisioned by OpenTofu and Ansible.
- **No licence yet.** The code is public, but no licence has been chosen, so
  default copyright applies. Ask before reusing it.

---

Built and maintained by [@SBillion](https://github.com/SBillion).
