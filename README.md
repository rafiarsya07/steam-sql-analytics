# Steam Market Intelligence — Live SQL Analytics

A portfolio SQL project that runs **real SQLite queries live in the browser** via WebAssembly — no server, no precomputed data, no build step.

🔗 **Live demo:** (https://steam.rafiarsya.com/)

---

## What this is

This project applies advanced SQL techniques — window functions, recursive CTEs, self-joins, rolling calculations — to a synthetic, statistically-realistic Steam-style game market dataset. The entire analysis runs client-side: when you open the page, a real SQLite engine boots in WASM, loads the `.db` file, and executes every query live, with timing.

The bottom of the page has a **live SQL console** wired directly to the same database. Run the sample queries, or write your own `SELECT` against any of the available tables.

---

## Files

```
index.html        # the whole app — DB embedded inline as base64, runs live SQL
steam_games.db    # raw SQLite database (also embedded in index.html)
sql/
  01_schema.sql           # table definitions
  02_data_cleaning.sql    # deduplication, dirty-row handling
  03_joins.sql            # genre bridge table queries
  04_ctes.sql             # CTEs and subqueries
  05_window_functions.sql # RANK(), ROW_NUMBER(), rolling sums
  06_rolling_calcs.sql    # cumulative and moving aggregates
  07_report.sql           # final 4-part analytical report
```

---

## Database schema

| Table | Description |
|---|---|
| `games` | Raw game records (includes duplicates / dirty rows) |
| `games_deduped` | Cleaned view — use this for analysis |
| `game_genres` | Bridge table linking games to genres |
| `genres` | Genre list with `parent_genre` for hierarchy |

---

## SQL techniques covered

- **Window functions** — `RANK()`, `ROW_NUMBER()`, `SUM() OVER`, `AVG() OVER`
- **CTEs** — used as the idiomatic SQLite workaround for missing `QUALIFY`
- **Self-joins** — for comparative / lag-style queries
- **Rolling calculations** — cumulative release growth, moving averages
- **Data cleaning** — deduplication, NULL handling, normalisation into a bridge table
- **Subqueries** — "hidden gems" filter (high-rating, low-review-count games)
- **Aggregates** — genre × price, studio consistency, rating distributions

---

## Sample queries (also in the live console)

**Top 10 highest-rated games (≥ 100 reviews)**
```sql
SELECT name, developer,
       ROUND(1.0*positive_ratings/(positive_ratings+negative_ratings), 3) AS rating
FROM games_deduped
WHERE (positive_ratings+negative_ratings) >= 100
ORDER BY rating DESC
LIMIT 10;
```

**RANK() top 3 games per genre (window function)**
```sql
WITH ranked AS (
  SELECT g.genre_name, d.name,
         RANK() OVER (PARTITION BY g.genre_name ORDER BY d.positive_ratings DESC) AS rnk
  FROM games_deduped d
  JOIN game_genres g ON g.app_id = d.app_id
)
SELECT * FROM ranked WHERE rnk <= 3
ORDER BY genre_name, rnk;
```

**Releases per year**
```sql
SELECT strftime('%Y', release_date) AS yr, COUNT(*) AS releases
FROM games_deduped
GROUP BY yr ORDER BY yr;
```

> **SQLite tip:** SQLite has no `QUALIFY` — wrap window functions in a CTE and filter in an outer `WHERE` (see query above).

---

## Run locally

The database is embedded as base64 directly in `index.html`, so no local server is needed:

```bash
open index.html        # macOS
xdg-open index.html    # Linux
# or just double-click the file
```

---

## Deploy (free, static — no server needed)

Any static host works. Recommended options:

| Host | How |
|---|---|
| **Cloudflare Pages** | Connect repo → auto-deploys on push. Best for custom subdomain + free SSL. |
| **Vercel** | `vercel --prod` from this folder. |
| **Netlify** | Drag-and-drop the folder in the dashboard. |
| **GitHub Pages** | Push to `gh-pages` branch or `/docs` folder. |

All four give free HTTPS and support custom subdomains via a single CNAME record.

---

## Tech

- **[sql.js](https://github.com/sql-js/sql.js)** — SQLite compiled to WebAssembly
- **[Chart.js](https://www.chartjs.org/)** — charts rendered from live query results  
- Vanilla HTML/CSS/JS — no framework, no build tooling

---

## Part of a series

This is the first entry in a growing hub of live SQL case studies at [`sql.rafiarsya.com`](https://steam.rafiarsya.com). Same pattern, different datasets — check back for more.
