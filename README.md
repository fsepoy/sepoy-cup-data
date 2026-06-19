# sepoy-cup-data

Tournament data store for the **Sepoy Cup** family FC24 gaming tournament.

## Why this repo exists

The Sepoy Cup webapp ([fsepoy/sepoy-cup-2026](https://github.com/fsepoy/sepoy-cup-2026)) is a fully static site — no backend, no database, no server costs. It needs somewhere to read and write match scores. This repo is that place.

The app reads `data.json` directly from GitHub's raw content URL on every page load. The admin panel writes back by committing a new version of `data.json` via the GitHub Contents API. Git history then doubles as a complete audit trail of every score entry.

## Why it's public

The tournament scoreboard is meant to be visible to every participant. Making this repo public lets the webapp fetch `data.json` without any authentication, keeping the static-site architecture simple and free.

## Structure

```
sepoy-cup-data/
└── 2026/
    └── fc24/
        └── data.json   ← all tournament state: teams, groups, fixtures, scores, champion
```

The path structure (`YEAR/GAME/`) is intentional — future editions add a new folder without touching the app:

```
2027/fc25/data.json
2027/fifa/data.json
```

## The Personal Access Token (PAT)

The admin panel writes scores by committing changes here via the **GitHub Contents API**. This requires a GitHub Personal Access Token.

| Property | Detail |
|---|---|
| Scope required | `contents:write` on this repo only (fine-grained PAT recommended) |
| Where it is stored | The tournament organiser's **browser `localStorage`** — nowhere else |
| What it can do | Read and write files in this repo only |
| What it cannot do | Access any other repo, user data, or GitHub settings |
| How it's used | `Authorization: Bearer <token>` header on GitHub API requests |
| If compromised | Regenerate at **GitHub → Settings → Developer settings → Personal access tokens**, then re-enter in the admin panel |

> The PAT is never committed to any repository, never logged, and never sent anywhere except the GitHub Contents API endpoint (`api.github.com`).

## Data schema

See [`data.json`](2026/fc24/data.json) for the live data. Key fields:

```json
{
  "meta":     { "name": "Sepoy Cup", "year": 2026, "game": "FC24" },
  "groups":   { "A": ["croatia", "belgium", "germany"], "B": ["england", "france", "argentina"] },
  "teams":    { "croatia": { "name": "Croatia", "flag": "🇭🇷", "short": "CRO" } },
  "fixtures": [
    { "id": "GA1", "stage": "group", "group": "A",
      "home": "croatia", "away": "belgium",
      "homeScore": null, "awayScore": null }
  ],
  "champion": null
}
```

Scores are `null` until played. The webapp computes all standings, bracket seeding, and knockout advancement client-side from this single file — no server-side logic anywhere.
