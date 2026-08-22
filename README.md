# Matchday

A static site that turns one CSV per match into a browsable log of every game you've been to.
No build step, no dependencies — drop it in a GitHub repo, turn on Pages, done.

```
index.html          the whole site
competition_type.csv  competition name -> type (used for the "Competitions by type" stat
                    and the type filter on Matches)
continents.csv      country code -> confederation (AFC, UEFA, CONCACAF...), used for
                    the "Continents" stat
continents/          confederation icons for the Continents stat: AFC.png, UEFA.png…
data/               one CSV per match + index.csv (the manifest)
emblems/            crests: M001a.png (home), M001b.png (away), M001.png (competition)
kit/                kit icons on the formation card: M001a.png, M001b.png
flags/              FIFA 3-letter codes, reused across matches: HKG.png, ARG.png…
icons/              gk, captain, sub-on, sub-off, yellow, second-yellow, red (.png)
trophies/           trophy photos for title deciders: M031.png, M009.png…
photos/             M001_1.jpg, M001_2.jpg…
```

Missing images never break anything: a missing crest falls back to a monogram, a missing
flag to a small code chip, a missing icon to a coloured letter, and a missing kit icon or
photo just disappears.

## Adding a match

1. Copy an existing CSV in `data/` to the next number, e.g. `M042.csv`.
2. Fill it in. Any field you leave blank is simply not displayed.
3. Add `M042` as a new line in `data/index.csv`.
4. Drop `M042a.png` / `M042b.png` into `emblems/` and `kit/`, plus `M042.png` in `emblems/`
   for the competition logo.

**Matches you have tickets for but haven't been to yet:** name them `U001`, `U002`… and set
`meta,status,upcoming`. They show in the list with an "upcoming" tag and are excluded from the
statistics. Once you've been, rename the file and its index line to the next free `M###` and set
the status to `played` — so buying tickets out of order never shuffles your numbering.

**A match that didn't happen** — postponed, abandoned, called off — set `meta,status,postponed`
(or whatever word you like; the site just displays it as a tag, it doesn't check for a specific
value beyond `played`). It still gets its own page, with whatever line-ups, weather, or notes you
recorded, so the visit itself isn't lost. But it's excluded from every statistic — teams seen,
countries, stadiums, weather, all of it — the same as an upcoming match, since `status` only ever
needs to be exactly `played` to count toward anything. This applies even if you were there in
person and the game just never kicked off.

## CSV reference

Column A is always the record type. Lines starting with `#` are ignored, so keep the commented
column headers — they're the only documentation you need while editing. Write `n/a` for anything
that will never exist; it's treated the same as blank.

| Type | Columns |
|---|---|
| `meta` | `field,value` — id, status, competition, round, date (`YYYY-MM-DD`), kickoff, venue, city, country, neutral, attendance, score (`0-7`), aggregate, decision (`AET`, `Pens 4-3`), referee, referee_country |
| `team` | `side,name,country,coach,coach_country,formation,squad_value,pitch_fill,pitch_number,gk_fill,stat_color` |
| `player` | `side,role,number,name,kit_name,country,id,captain,goalkeeper,on,off,yellow,double yellow,red,value` |
| `goal` | `minute,side,scorer,type,assist` — type is `goal`, `penalty` or `own goal`; `side` is the team the goal counted **for** |
| `pen` | `order,side,player,result` — a penalty-shootout kick; result is `scored` or `missed` |
| `champion` | `field,value` — team, side, title, season, note; only for a title-deciding match |
| `stat` | `name,home,away` — any stat you like; add rows freely |
| `weather` | `field,value` — condition, temp_c |
| `personal` | `field,value` — ticket_price, ticket_currency, ticket_price_usd, section, row, seat, seating |
| `photo` | `file,caption` |

**Players.** `role` is `start`, `sub` (came on) or `bench` (unused). List the starting eleven in
shape order — goalkeeper first, then each line as you'd read it across the pitch — because that
order plus `formation` is what positions them on the diagram. `kit_name` is the name shown on the
pitch; leave it blank and the last word of the full name is used, which is why *Lee Chi Ho* needs
`Lee` written in but *Gonzalo Higuaín* doesn't. `value` is market value in € millions; if every
player has one the squad total is computed, otherwise `squad_value` on the `team` row is used.

**`id`.** Leave this blank almost always — it has nothing to do with player identity across the
whole site by default. It exists for one situation: two different real people who happen to share
the exact same `name` (two players both called "Jorginho", say). Give each of them a distinct,
arbitrary `id` and the statistics tab will count them as separate people instead of merging their
appearances into one. A blank `id` matches every other blank `id` for that name, so this only
needs filling in when a genuine name collision comes up.

**Player markers.** `captain` and `goalkeeper` take `yes` (or anything non-blank); everything else
takes a **minute**: `on` and `off` for entering and leaving the pitch, and `yellow`, `double
yellow`, `red` for cards. Markers appear right after the player's name in a fixed order —
goalkeeper, then captain, then every timed event in the order it happened. So a starter booked on
55 and substituted on 71 shows the yellow card first, then the down arrow.

**Formation colours.** `pitch_fill` / `pitch_number` set the fill and number colour for this
team's outfield dots on the formation diagram; `gk_fill` does the same for the goalkeeper's dot.
Leave any of them blank for the site's defaults (off-white dots, dark numbers, amber goalkeeper).
`stat_color` sets this team's bar colour in Match stats; leave blank for the default (dark for
home, red for away).

**Penalties.** One row per kick, only if the match went to a shootout. `order` is that kick's
position in the whole shootout — `1`, `2`, `3`... in the order it was actually taken, regardless
of side. It isn't shown as a number on the page — it just controls the order the rows are listed
in, top to bottom, so whichever row comes first is whichever team actually kicked first. The
running score next to the "Penalty shootout" heading is worked out automatically from these rows.

**Champions.** Add a `champion` block only to a match that decided a title — a cup final, or the
league game that settled it. `team` is who won, `side` (`home`/`away`) lets the site show their
crest, and `title`, `season` and `note` are optional (`title` falls back to the competition name).
That match then gets a Champions block on its page, a gold trophy tag in the match list, and a
card in the Champions tab. Put a trophy picture in `trophies/` named after the match
(`trophies/M031.png`) — without one, a small drawn trophy outline is used instead.

**Weather.** `condition` also picks the icon. It matches on keywords, so anything containing
*clear/sunny*, *partly/fair*, *cloud/overcast*, *rain/shower/drizzle*, *thunder/storm*,
*snow/sleet*, *fog/mist/haze* or *wind* draws the right one.

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

## Lookup tables

Two small CSVs sit in the project root, next to `index.html`, rather than in `data/` — they
describe categories, not matches, and are shared across every match rather than being per-match
files.

**`competition_type.csv`** — `competition,type`. Maps each competition name (exactly as it
appears in a match's `meta,competition`) to a type such as `Club - Domestic League` or
`International - Major Tournament`. Powers the "Competitions by type" stat and the type filter on
the Matches tab. A competition with no row here is grouped under "Uncategorised" rather than
dropped — nothing silently disappears, but it's worth adding the row so it counts correctly.

**`continents.csv`** — `country,continent`. Maps a FIFA country code to its confederation (`AFC`,
`UEFA`, `CONCACAF`, `CONMEBOL`, `CAF`, `OFC`). Powers the "Continents" stat, keyed off each match's
`meta,country`. A country with no row here is grouped under "Unknown".

Both tolerate a stray blank leading line (common from a spreadsheet export) and skip whatever row
matches the header text, so you can edit them in Excel/Sheets and re-export without fuss.

## Icons on the Statistics tab

Most stats show a small badge next to each name — what it is depends on the stat, not something
you configure per row:

- **Countries, Clubs by country, Players by nationality, Cities, Stadiums** — a national flag
  from `flags/`.
- **Teams by appearances, Teams by market value** — the team's own crest from `emblems/`, always
  — the same one that shows on its matches, so club sides and national sides look consistent
  side by side.
- **Players, Scorers, Head coaches, Referees** (by appearances or market value) — the person's
  own flag, from whichever match most recently recorded their country. If it differs across
  matches (a player who changed allegiance, a typo that got corrected later), the newest one wins.
- **Competitions** — that competition's logo from the `emblems/M001.png`-style files.
- **Continents** — the confederation icon from `continents/`.
- **Weather** — the same condition icon used on a match page.
- **Records, and Conditions → Temperature** (anything measured per match — attendance, goals,
  cards, ticket price…) — the competition logo for that specific match.

Any of these fall back gracefully if the underlying image is missing: a flag or crest shows a
small monogram, a competition or continent icon shows its code as text.

## Filling in weather automatically

`fetch_weather.py` pulls historical weather from [Open-Meteo](https://open-meteo.com/) — free,
no signup, no API key — and writes it straight into a match's `weather` block. It uses only the
Python standard library, so there's nothing to `pip install`.

1. Add each ground to `data/venues.csv` (name, latitude, longitude). The name must match a
   match's `meta,venue,...` value exactly.
2. Run it from the project folder:

   ```bash
   python3 fetch_weather.py M001            # one match
   python3 fetch_weather.py M001 M014 M028  # a few matches
   python3 fetch_weather.py --all           # every match in data/index.csv
   ```

A match that already has a weather condition filled in is left alone unless you add `--force`.
Run with no arguments to see the full usage note.

It fetches the temperature for the hour closest to kickoff, and turns Open-Meteo's numeric
weather code into a short phrase the site already recognises for icons (see the Weather section
above) — e.g. *"Overcast"*, *"Light rain"*, *"Partly cloudy"*.

## Editing CSVs on Windows

Avoid opening these files by double-clicking in Excel — it will reinterpret things like a `0-7`
score as a date and rewrite it. Better options:

- **Rainbow CSV** (free VS Code extension) — keeps the file as plain text, colour-codes columns,
  and shows the column name under your cursor. The best match for files that lean on commented
  headers like these.
- **Modern CSV** — a standalone grid editor built for CSV that treats every column as text by
  default, so nothing gets silently reformatted.
- **LibreOffice Calc** (free) — fine too, as long as you use the text-import dialog and set every
  column to "Text" before opening.

If you do use Excel, never double-click the file — use Data → Get Data → From Text/CSV and set
every column's type to Text in the import step.

## What's placeholder in the sample data

`M001.csv` is your own updated file — real data throughout.

`M031.csv` was parsed from the Word log, with a few things invented that still need replacing:
**weather**, **ticket price and seat**, the **formation** (the Word doc has pitch graphics, not
shape strings), and the **market values**, **match stats** and **pitch/stat colours**. No card
events are recorded in it. Everything else — teams, score, scorers, minutes, line-ups, coaches,
referee, attendance — came straight out of the document.

## Fonts

The CSS asks for Adobe Clean and DIN Next Round Pro first, so if you have either installed locally
the site uses them. Neither can be licensed for web delivery, so the fallbacks are Source Sans 3
(Adobe's open humanist sans, the closest sibling to Adobe Clean) and Archivo (a grotesque in the
DIN family, used for scores, numbers and labels).
