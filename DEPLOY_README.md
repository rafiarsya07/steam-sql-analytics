# Steam Market Intelligence — Live SQL Dashboard

This is the **realtime version** of the project. The old build pre-baked SQL
results into `dashboard_data.json` at build time — this one is a single
self-contained `index.html` that runs **real SQLite (via sql.js / WASM)
inside the visitor's browser**. The database is embedded directly in the
page as base64, so there's no separate `.db` file to lose track of and no
fetch that can 404 — just one file, drop it anywhere. Every chart, every
number, and the SQL console at the bottom are computed live, on load,
against the actual database. Nothing is precomputed server-side.

## Files
```
index.html        # the whole app — DB embedded inline, runs live SQL
```

## Run it locally
Because the database is embedded as base64 (not fetched), you can open
this file straight from disk — no local server required. Double-click it,
or:
```bash
open index.html        # macOS
xdg-open index.html    # Linux
```
It also works fine served from any static host (Cloudflare Pages, Vercel,
Netlify, GitHub Pages, etc.) if you want a public URL.

## Deploy it (free, static — no server needed)
Because everything runs client-side, any static host works. Pick one:

- **Cloudflare Pages** — fastest global edge, free, easiest custom-domain + SSL setup. Recommended.
- **Vercel** — `vercel --prod` from this folder, near-zero config.
- **Netlify** — drag-and-drop the folder in their dashboard, or `netlify deploy --prod`.
- **GitHub Pages** — free if the repo is public; push this folder to a `gh-pages` branch or `/docs` folder.

All four give you free HTTPS and let you point a subdomain at them with a
single DNS record (CNAME). Cloudflare Pages or Vercel are the least fiddly
if you've never wired a subdomain before.

## Live SQL console — built-in sample queries

The console at the bottom of the page ships with 5 ready-to-run queries
(pick from the dropdown, or copy/adapt them). Tables available:
`games` (raw, has dupes/dirty rows), `games_deduped` (cleaned view — use
this one), `game_genres` (bridge table), `genres` (with `parent_genre`
for the hierarchy).

**1. Top 10 highest-rated games (≥100 reviews)**
```sql
SELECT name, developer, positive_ratings, negative_ratings,
       ROUND(1.0*positive_ratings/(positive_ratings+negative_ratings), 3) AS rating
FROM games_deduped
WHERE (positive_ratings+negative_ratings) >= 100
ORDER BY rating DESC
LIMIT 10;
```
Finds the best-reviewed titles, filtered to games with at least 100
combined reviews so a 2-positive/0-negative outlier doesn't rank #1.

**2. Genre × average price**
```sql
SELECT g.genre_name, COUNT(*) AS games, ROUND(AVG(d.price),2) AS avg_price
FROM games_deduped d
JOIN game_genres g ON g.app_id = d.app_id
GROUP BY g.genre_name
ORDER BY avg_price DESC;
```
A standard JOIN + GROUP BY across the game↔genre bridge table to see
which genres command the highest average price.

**3. Releases per year**
```sql
SELECT strftime('%Y', release_date) AS yr, COUNT(*) AS releases
FROM games_deduped
GROUP BY yr
ORDER BY yr;
```
Counts releases per year using SQLite's `strftime()` date function —
good baseline before reading any single year's number in isolation.

**4. RANK() top 3 games within each genre**
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
The most "advanced" sample — a CTE wrapping a `RANK() OVER (PARTITION BY ...)`
window function, then filtered in an outer `SELECT` (SQLite has no
`QUALIFY`, so this CTE pattern is the idiomatic workaround).

**5. Schema: list all tables**
```sql
SELECT name, type FROM sqlite_master WHERE type IN ('table','view');
```
A meta-query against SQLite's own catalog — useful starting point to see
what tables/views exist before writing custom queries (`games`, `genres`,
`game_genres`, plus the `games_deduped` view).

> Tip: SQLite doesn't support `QUALIFY` — if you write a window function
> and want to filter on it, wrap it in a CTE (like query #4 above) and
> filter in an outer `WHERE` instead.

## Subdomain recommendation

**Use `sql.rafiarsya.com`.**

Why this beats the alternatives, for a portfolio piece you'll keep adding
to long-term:

| Option | Verdict |
|---|---|
| `sql.rafiarsya.com` | **Best.** Short, names the *skill* not the dataset — you can later host other SQL projects at the same subdomain (`sql.rafiarsya.com/steam`, `/nba`, etc.) without re-architecting anything. Reads cleanly on a resume/LinkedIn. |
| `steam.rafiarsya.com` | Ties the subdomain to one dataset. Fine for a single project, but you'll outgrow it the moment you build a second SQL case study. |
| `projects.rafiarsya.com` | Too generic — works as an umbrella, but doesn't say anything about what's there until you click. |
| `rafiarsya.com/sql/steam` (subfolder, no subdomain) | Also valid and zero extra DNS work, but a subdomain looks more like a standalone "product" in a portfolio and is easier to feature as its own card/link with its own clean URL. |

**Long-term structure I'd set up now**, so this doesn't need rework later:
```
rafiarsya.com              → main portfolio / personal site
sql.rafiarsya.com          → landing page listing all SQL case studies
sql.rafiarsya.com/steam    → THIS project (live SQL dashboard)
sql.rafiarsya.com/<next>   → next SQL project, same pattern
blog.rafiarsya.com         → your blog (or rafiarsya.com/blog if you prefer one root)
```
This means `sql.rafiarsya.com` becomes a growing portfolio hub, not a
one-off link you'll abandon. One CNAME, one Cloudflare Pages project, many
projects served from subfolders.

## DNS setup (once you've deployed to e.g. Cloudflare Pages)
1. In your DNS provider (wherever `rafiarsya.com`'s nameservers live), add:
   ```
   Type:  CNAME
   Name:  sql
   Value: <your-project>.pages.dev   (or vercel/netlify equivalent)
   ```
2. In the host's dashboard, add `sql.rafiarsya.com` as a custom domain and
   let it auto-provision SSL (all four hosts do this for free, usually
   within minutes).
3. Done — `https://sql.rafiarsya.com/steam` resolves straight to
   `index.html` in this folder.

## Blog post
A draft write-up for `blog.rafiarsya.com` (or wherever your blog lives) is
included as `BLOG_POST_DRAFT.md` in this same output — copy/paste and
adjust the tone/links to match your site.