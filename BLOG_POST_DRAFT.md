---
title: "Taking my SQL portfolio project from static export to a live, in-browser database"
date: 2026-06-21
slug: steam-sql-live-dashboard
tags: [sql, sqlite, wasm, data-analysis, portfolio]
---

A while back I built a "Steam Market Intelligence" case study to put the
advanced-SQL techniques I'd just learned — window functions, recursive
CTEs, self-joins, rolling calculations — to work on something more
interesting than another tutorial dataset. The result: a synthetic,
statistically-realistic game-market dataset, seven `.sql` files walking
through the full curriculum, and a dashboard summarizing the findings.

The first version worked, but it bugged me a little: the dashboard was
just static HTML reading from a `dashboard_data.json` file I'd exported
ahead of time. The "SQL project" had already finished running by the time
anyone opened the page. It *looked* like a live analytics tool. It wasn't
one.

## Making it actually live

So I rebuilt it. The new version ships only two files — `index.html` and
the raw `steam_games.db` SQLite file — and runs **SQLite compiled to
WebAssembly (sql.js)** directly in the visitor's browser. When the page
loads, it:

1. Boots a real SQLite engine in WASM
2. Fetches the `.db` file
3. Runs every query — the cumulative release-growth chart, the genre
   breakdown, the price-vs-rating analysis, the "hidden gems" subquery,
   the consistent-studios aggregate — live, on load, timing each one
4. Renders the results into the same charts as before

Nothing is precomputed. If I edit a query, the page's behavior changes
immediately on refresh — no rebuild step, no export script.

## The part I'm most happy with: a real SQL console

The bottom of the page is a live query box. It's not a styled mockup of a
terminal — it's wired straight to the same `db.exec()` call the rest of
the page uses. You can run the sample queries, or write your own
`SELECT` against `games`, `games_deduped`, `game_genres`, or `genres`,
and see real results with real timing, computed against the actual
database sitting in your browser's memory.

That one feature turns the project from "here's a report I made" into
"here's a database, go poke at it" — which is a much better demonstration
of actually knowing SQL than a chart ever was.

## Why this approach (and not a backend)

I deliberately didn't stand up a server to run these queries. Shipping
the `.db` file and running it client-side via WASM means:

- **Zero hosting cost** — it's a static site, deployable anywhere
  (Cloudflare Pages, Vercel, Netlify, GitHub Pages)
- **No backend to keep alive** — nothing to monitor, nothing that goes
  down
- **Genuinely live** — "live" usually means "a server ran a query
  recently." Here it means the query runs on *your* machine, right now

The dataset is small (under 200 KB), so shipping the whole database to
the browser is trivial. For a bigger dataset I'd reach for a real backend
or a hosted SQLite-over-HTTP approach — but for a portfolio piece, running
the actual engine client-side is both more honest and more fun to show
off.

## What's next

This is going up at `sql.rafiarsya.com/steam`, as the first entry in what
I'm planning as a small hub of live SQL case studies at
`sql.rafiarsya.com`. Next one's already sketched out — same pattern, new
dataset.

If you want to look at the queries directly, the full SQL files (data
cleaning, joins, CTEs, window functions, the final 4-part report) are in
the [GitHub repo](https://github.com/rafiarsya/steam-sql-analytics).
