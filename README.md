# League HQ

A read-only web page showing the current state of **one private Yahoo Fantasy hockey
league**: standings, this week's matchups, trades, waiver priority, and recent roster
moves. One page, light theme, meant to be opened on a phone or a laptop and read in
fifteen seconds.

It is not a draft tool. It runs all season.


## What's on it

**Standings** — records, points for and against, point differential, current streak, and
form over the last five weeks. The playoff cut line is drawn in as an actual line, so
"am I in?" is answered by looking rather than counting. Your own team is highlighted.

**This week** — every matchup with live scores, leader on top, and a bar showing how
lopsided it currently is.

**Trades** — the part the group chat actually argues about. Pending trades first, with
veto vote counts and time remaining, then everything settled this week. Each trade shows
both sides of the deal with the players going each way.

**Recent moves** — adds and drops across all twelve teams in one feed, so nobody has to
dig through Yahoo's transaction page to find out who grabbed the hot free agent.

**Waiver priority** — the current order.

## How it works

The Yahoo Fantasy Sports API has no websocket, so the page polls. Standings, scoreboard,
and transactions refresh on a five-minute timer; everything else is fetched once on load.
Five minutes is well inside the cadence Yahoo suggests, and nothing on this page changes
faster than that anyway.

The page is read-only. It shows a pending trade and its vote count; voting still happens
in Yahoo. That is deliberate — see [SPEC.md](SPEC.md).

## Yahoo API access

Needs approved access to the Yahoo Fantasy Sports API
([application form](https://sports.yahoo.com/developer/access/)). Read access only.

| Endpoint | Feeds |
| --- | --- |
| `league/{key}/standings` | records, points for and against, streaks, playoff seed |
| `league/{key}/scoreboard;week={n}` | the week's matchups and live scores |
| `league/{key}/transactions` | trades, adds and drops |
| `league/{key}/teams` | team names, managers, waiver priority |
| `league/{key}/players` | player names, positions, clubs, headshot URLs |

Player headshots come from the player resource, which exposes `headshot.url` and
`image_url` on Yahoo's CDN. They are hotlinked at render time, never downloaded or
re-hosted.

## Running it

```bash
cp .env.example .env     # Yahoo client ID/secret and your league key
npm install
npm run auth             # one-time OAuth handshake, stores a refresh token
npm start                # open http://localhost:8080
```

## About the screenshot

Every league, team, manager and player name in `mockups/` and `docs/` is invented, and the
headshots are generated silhouettes rather than real photographs. The mockup is built by
[`scripts/build_mockup.py`](scripts/build_mockup.py) from hardcoded fictional data, so this
repository contains no Yahoo Fantasy Sports data and no third-party imagery.

```bash
python3 scripts/build_mockup.py
```

Open `mockups/league-hq.html` directly in a browser to see it at full size.

## Status

Design and specification complete. Implementation is blocked on API access approval.

## License

MIT — see [LICENSE](LICENSE).
