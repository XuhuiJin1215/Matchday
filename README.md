# Matchday

A static site that turns one CSV per match into a browsable log of every game you've been to.
No build step, no dependencies — drop it in a GitHub repo, turn on Pages, done.

```
index.html          the whole site
data/               one CSV per match + index.csv (the manifest)
emblems/            M001a.png (home), M001b.png (away)
flags/              FIFA 3-letter codes: HKG.png, ARG.png, ENG.png…
photos/             M001_1.jpg, M001_2.jpg…
```

Missing images never break anything: a missing emblem falls back to a monogram, a missing
flag to a small code chip, a missing photo disappears from the gallery.

## Adding a match

1. Copy an existing CSV in `data/` to the next number, e.g. `M042.csv`.
2. Fill it in. Any field you leave blank is simply not displayed.
3. Add `M042` as a new line in `data/index.csv`.
4. Drop `M042a.png` / `M042b.png` into `emblems/`.

**Matches you have tickets for but haven't been to yet:** name them `U001`, `U002`… and set
`meta,status,upcoming`. They show in the list with an "upcoming" tag and are excluded from
the statistics. Once you've been, rename the file and its index line to the next free `M###`
and set the status to `played` — so buying tickets out of order never shuffles your numbering.

## CSV reference

Column A is always the record type. Lines starting with `#` are ignored, so keep the
commented column headers — they're the only documentation you'll need while editing.
Write `n/a` for anything that will never exist; it's treated the same as blank.

| Type | Columns |
|---|---|
| `meta` | `field,value` — id, status, competition, round, date (`YYYY-MM-DD`), kickoff, venue, city, country, neutral, attendance, score (`0-7`), aggregate, decision (`AET`, `Pens 4-3`), referee, referee_country |
| `team` | `side,name,country,coach,coach_country,formation,shirt,trim,shorts,pattern,squad_value` |
| `player` | `side,role,number,name,country,pos,on,off,value` |
| `goal` | `minute,side,scorer,type,assist` — type is `goal`, `penalty` or `own goal`; `side` is the team the goal counted **for** |
| `stat` | `name,home,away` — any stat you like; add rows freely |
| `weather` | `field,value` — condition, temp_c, feels_c, humidity, wind_kph, precip_mm |
| `personal` | `field,value` — ticket_price, ticket_currency, ticket_price_usd, section, row, seat, seating, company, rating, notes |
| `photo` | `file,caption` |

**Kits.** `shirt` and `trim` are hex colours; `pattern` is `solid`, `stripes`, `hoops`,
`halves`, `sash` or `checks`. These draw the shirt on the formation diagram and the team card.

**Players.** `role` is `start`, `sub` (came on) or `bench` (unused). List the starting eleven in
shape order — goalkeeper first, then each line as you'd read it across the pitch — because that
order plus `formation` is what positions them on the diagram. `pos` is just a printed label.
`value` is market value in € millions; if every player has one, the squad total is computed,
otherwise `squad_value` on the `team` row is used.

**Seats.** Set `seating,unreserved` for terraces and general admission and leave `seat` blank —
it renders as "Lower Tier West · unreserved" rather than pretending you had a seat number.

## Photos

Resize before committing: **1600px on the long edge, JPEG quality 80** lands around 200–350 KB,
which is plenty (the gallery displays them at 170px tall and lazy-loads them). Anything above
2000px is wasted bytes in a git repo that never shrinks.

## Working on it locally

Browsers block `fetch` on `file://`, so opening `index.html` by double-clicking shows an error.
Run a server instead:

```bash
python3 -m http.server
# then open http://localhost:8000
```

## What's placeholder in the sample data

`M001.csv` and `M031.csv` are parsed from the Word log, but a few things were invented and
need replacing: **weather**, **ticket price / seat / rating / notes**, the **kit colours**,
the **formations** (the Word doc has pitch graphics, not shape strings), and in `M031` the
**market values** and **match stats**. Everything else — teams, scores, scorers, minutes,
line-ups, coaches, referees, attendances — came straight out of the document.

## Fonts

The CSS asks for Adobe Clean and DIN Next Round Pro first, so if you have either installed
locally the site uses them. Neither can be licensed for web delivery, so the fallbacks are
Source Sans 3 (Adobe's open humanist sans, the closest sibling to Adobe Clean) and Archivo
(a grotesque in the DIN family, used for scores, numbers and labels).
