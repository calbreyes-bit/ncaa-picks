# NCAA Picks Dashboard

Self-hosted GitHub Pages dashboard for the NCAA football model, mirroring
`nfl-picks`'s own shape as a separate dashboard (Carlos's explicit choice,
2026-09-10) rather than a tab folded into the NFL one: four tabs (Top Edges /
ATS Picks / Games / Tracker) reading a JSON blob baked into `index.html` at
publish time.

## Files

- `template.html` -- the real template. Has one placeholder,
  `__GAMES_JSON__`, that `nfl-ingest/run/ncaa_07_push_dashboard.py` replaces
  with live data from `ncaa.v_future_games_model` to produce `index.html`.
- `index.html` -- generated. Do not hand-edit; it's overwritten on every
  push-script run. Not committed until the first real run produces it.

## v1 scope: core only, no Top Edges "7pt Teasers" tab

`ncaa.v_future_games_model` is deliberately core-only right now (per Carlos's
"core first" decision, 2026-09-09): moneyline, spread, game total, team
totals. It has none of the NFL view's later additions -- no H2H, no
injuries, no venue-split scoring, no recent form, no 7-point teaser -- so
this dashboard doesn't surface any of those yet. Natural follow-ups once the
NCAA view grows the same way the NFL one did, incrementally.

Two real modeling differences from the NFL dashboard, not oversights:

- **SP+ instead of Elo** for team strength (labeled "SP+" throughout the UI,
  not "Elo") -- CollegeFootballData's SP+ rating is already expressed in
  points, so there's no `/25` scale conversion like nfelo needs.
- **No tie probability.** NCAA football can't end in a tie (mandatory
  overtime since 1996), so `ml.modelHome`/`ml.modelAway` are a plain
  complementary pair -- there's no `modelTie` field to look for.

## One-time setup (do this on your machine)

1. Create an empty GitHub repo named `ncaa-picks` (same account/org as your
   other dashboards) and push this local repo to it:
   ```
   git remote add origin <your-ncaa-picks-repo-url>
   git branch -M main
   git push -u origin main
   ```
2. Enable GitHub Pages for the repo (Settings -> Pages -> Deploy from branch
   `main`, root).
3. Tracker sync backend: create a **new** Google Sheet for NCAA picks (don't
   reuse the NFL or soccer one -- each dashboard's tracker is its own sheet),
   open Extensions -> Apps Script, and paste in the same `TrackerSync.gs`
   you're already using for the other two dashboards (it's generic -- no
   sport-specific logic). Deploy -> New deployment -> Web app -> Execute as
   Me -> Who has access: Anyone. Copy the resulting `/exec` URL.
4. Open `template.html`, find `var TRACKER_SYNC_URL = '';` (right after the
   comment `// Carlos: deploy TrackerSync.gs on a NEW Google Sheet for NCAA
   and paste that Web App /exec URL here`), and set it to that URL. Commit
   that change.
5. Set `NCAA_PICKS_REPO_DIR` in your environment if this repo isn't at
   `~/Documents/ncaa-picks` (that's the default `ncaa_07_push_dashboard.py`
   uses).

## Running it

From `nfl-ingest/`, after your usual NCAA data-refresh scripts
(`run/ncaa_04_odds.py`, `run/ncaa_05_power_ratings.py`) so the view has
fresh lines:

```
python run/ncaa_07_push_dashboard.py
```

This reads `ncaa.v_future_games_model` + team logos, writes `index.html`
here, and commits+pushes it (best-effort on the git step -- if push fails
you'll see a message and can push manually; the JSON build itself never
silently fails).

## Known v1 scope decisions

- No live tradable odds in the model view (only de-vigged probabilities and
  lines), so Tracker odds entry is manual-only -- same as the NFL dashboard.
- No composite star/tier score -- "Top Edges" just ranks every market's raw
  `|edge|` across all games.
- No line-move sparkline -- just a static open -> now display.
- No `preview_sample.html` yet (the NFL dashboard has one with synthetic
  data for previewing the UI before a first real pipeline run) -- can build
  one on request once you've seen real output and want it.
