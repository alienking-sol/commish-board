# Specification

## Goal

One page that answers "where does the league stand right now?" without opening Yahoo.
Standings, this week's scores, what trades are live, who picked up whom. Readable in
fifteen seconds on a phone.

## Non-goals

- Anything to do with the draft
- Voting on trades, setting lineups, or any other write operation
- Supporting leagues other than the one configured in `.env`
- Public hosting or multi-tenancy

## Architecture

```
Yahoo Fantasy API  ──poll (5m)──>  Node server  ──>  static page + JSON
```

Single process, no database. The latest snapshot is held in memory and served as JSON to
a page that re-renders on a timer. Nothing persists between restarts; a cold start just
refetches.

### Polling

| Data | Cadence |
| --- | --- |
| `standings`, `scoreboard`, `transactions` | every 5 minutes |
| `teams`, `players` | once on start, then daily |

Yahoo's responses carry `refresh_rate="60"`. Five minutes is well under that, and none of
this data moves faster. Scoreboard polling pauses overnight when no games are live.

On a 429 or 5xx, back off exponentially to a 30-minute cap and keep serving the last good
snapshot. The page header shows the time of the last successful fetch, so a stale page
reads as stale rather than as wrong.

## Auth

OAuth 2.0 authorization code flow. `npm run auth` runs it once and stores the refresh
token in `.tokens.json` (gitignored). Access tokens last about an hour; the client
refreshes at 45 minutes rather than waiting for a 401.

Redirect URI is `https://localhost:8080/callback` and must match the registered value
exactly. Yahoo requires https; plain `http://localhost` is rejected.

## Data model

```
League {
  key, name, season, week, scoringType, numTeams, lastSync
  teams:    [{ teamKey, name, manager, logoUrl, waiverPriority, moves }]
  standings:[{ teamKey, rank, wins, losses, ties, pointsFor, pointsAgainst,
               streakType, streakLength, playoffSeed }]
  matchups: [{ week, teamKeys[2], points[2], status }]
  trades:   [{ id, status, proposedAt, expiresAt, vetoVotes,
               sides: [{ teamKey, players[] }] }]
  moves:    [{ type, teamKey, playerKey, timestamp }]
}
```

Derived in the browser:

- **Point differential** and **last five** form, from standings and matchup history
- **Playoff cut line**, from `num_playoff_teams` in league settings
- **Matchup bar**, each team's share of the combined score so far

## Trades

`league/{key}/transactions` returns trades alongside adds and drops. Pending trades carry
a status plus vote and expiry data; completed and vetoed ones carry their outcome. The
page groups them: pending at the top, then everything settled in the current week.

Each side of a trade lists the players going out, resolved against the cached player map
for names, positions, clubs and headshots.

**Read-only by design.** Showing a pending trade and its veto count is useful. Casting the
vote is not something this page should be able to do, so write scope is not requested —
which also makes the API access application narrower and easier to approve.

## Visual notes

Light theme, built to be read in a bright room and to print legibly.

- Blue marks structure (playoff line, matchup bars, kicker text); green and red are
  reserved for outcomes, so colour never means two things at once
- Numbers are tabular-figure aligned so columns scan vertically
- Form is five small blocks rather than a "WWLWL" string, which reads faster at a glance
- The viewer's own team row is tinted, the one piece of personalization on the page

## Testing

Standings and transactions persist after the fact, so a completed prior season provides
real data to develop against. `npm run snapshot -- <league_key>` writes a fixture the page
can render offline, which keeps development off the live API.
