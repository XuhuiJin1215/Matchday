# Matchday

A static site that turns one CSV per match into a browsable log of every game you've been to.
No build step, no dependencies — drop it in a GitHub repo, turn on Pages, done.

```
index.html          the whole site
data/               one CSV per match + index.csv (the manifest)
emblems/            club/country crests: M001a.png (home), M001b.png (away)
kit/                kit icons on the formation card: M001a.png, M001b.png
flags/              FIFA 3-letter codes, reused across matches: HKG.png, ARG.png…
icons/              captain.png, gk.png, yellow.png, red.png, second-yellow.png
photos/             M001_1.jpg, M001_2.jpg…
```

Missing images never break anything: a missing crest falls back to a monogram, a missing
flag to a small code chip, a missing icon to a coloured letter, and a missing kit icon or
photo just disappears.

## Adding a match

1. Copy an existing CSV in `data/` to the next number, e.g. `M042.csv`.
2. Fill it in. Any field you leave blank is simply not displayed.
3. Add `M042` as a new line in `data/index.csv`.
4. Drop `M042a.png` / `M042b.png` into `emblems/` and `kit/`.

**Matches you have tickets for but haven't been to yet:** name them `U001`, `U002`… and set
`meta,status,upcoming`. They show in the list with an "upcoming" tag and are excluded from the
statistics. Once you've been, rename the file and its index line to the next free `M###` and set
the status to `played` — so buying tickets out of order never shuffles your numbering.

## CSV reference

Column A is always the record type. Lines starting with `#` are ignored, so keep the commented
column headers — they're the only documentation you need while editing. Write `n/a` for anything
that will never exist; it's treated the same as blank.

| Type | Columns |
|---|---|
| `meta` | `field,value` — id, status, competition, round, date (`YYYY-MM-DD`), kickoff, venue, city, country, neutral, attendance, score (`0-7`), aggregate, decision (`AET`, `Pens 4-3`), referee, referee_country |
| `team` | `side,name,country,coach,coach_country,formation,squad_value,pitch_fill,pitch_number,gk_fill,stat_color` |
| `player` | `side,role,number,name,kit_name,country,pos,captain,on,off,value` |
| `goal` | `minute,side,scorer,type,assist` — type is `goal`, `penalty` or `own goal`; `side` is the team the goal counted **for** |
| `card` | `minute,side,player,type` — type is `yellow`, `red` or `second yellow` |
| `stat` | `name,home,away` — any stat you like; add rows freely |
| `weather` | `field,value` — condition, temp_c, feels_c, humidity, wind_kph |
| `personal` | `field,value` — ticket_price, ticket_currency, ticket_price_usd, section, row, seat, seating |
| `photo` | `file,caption` |

**Players.** `role` is `start`, `sub` (came on) or `bench` (unused). List the starting eleven in
shape order — goalkeeper first, then each line as you'd read it across the pitch — because that
order plus `formation` is what positions them on the diagram. `pos` also controls the goalkeeper
badge: write `GK` and that player gets the goalkeeper icon wherever they're listed. `kit_name` is
the name shown on the pitch; leave it blank and the last word of the full name is used, which is
why *Lee Chi Ho* needs `Lee` written in but *Gonzalo Higuaín* doesn't. `captain` is `yes` for the
captain, otherwise blank. `on` / `off` are the minutes this player entered / left the pitch —
leave `on` blank for a player who started, and `off` blank for a player still on at full time.
`value` is market value in € millions; if every player has one the squad total is computed,
otherwise `squad_value` on the `team` row is used.

**Cards.** One row per booking. `player` is matched against the name in the `player` rows for the
same `side`, so spell it identically. A second yellow counts as a dismissal in the statistics
(under "most red cards") as well as being its own icon.

**Formation colours.** `pitch_fill` / `pitch_number` set the fill and number colour for this
team's outfield dots on the formation diagram; `gk_fill` does the same for the goalkeeper's dot.
Leave any of them blank for the site's defaults (off-white dots, dark numbers, amber goalkeeper).
`stat_color` sets this team's bar colour in Match stats; leave blank for the default (dark for
home, red for away).

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

`M001.csv` and `M031.csv` are parsed from the Word log, but a few things were invented and need
replacing: **weather**, **ticket price and seat**, and the **formations** (the Word doc has pitch
graphics, not shape strings). In `M031`, market values, match stats, and the pitch/stat colours
are also illustrative. Card events are left out of both samples rather than guessed at. The
captains shown (Messi for Argentina in 2014, Buffon and Ramos in the 2017 final) are real.
Everything else — teams, scores, scorers, minutes, line-ups, coaches, referees, attendances —
came straight out of the document.

## Fonts

The CSS asks for Adobe Clean and DIN Next Round Pro first, so if you have either installed locally
the site uses them. Neither can be licensed for web delivery, so the fallbacks are Source Sans 3
(Adobe's open humanist sans, the closest sibling to Adobe Clean) and Archivo (a grotesque in the
DIN family, used for scores, numbers and labels).
