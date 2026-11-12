# Lurker

Lurker samples 4chan boards on a schedule, extracts keywords and their sentiment, draws wordclouds and keeps a history of topic frequency and sentiment that you can chart in the dashboard.

Data source: the public read-only [4chan API](https://github.com/4chan/4chan-API) at a.4cdn.org. Lurker is an independent project and is not affiliated with, endorsed by or sponsored by 4chan. Images are never downloaded.

## Installation

1. Add https://github.com/snorrea/ha-apps under Settings → Apps → Install app → ⋮ → Repositories.
2. Enable Advanced Mode and add ghcr.io under the App store's private registries, using username snorrea and a classic GitHub token with read:packages.
3. Install Lurker from the new repository, set the boards and notification targets, and start it.
4. Enable Start on boot and Automatic updates.

## Operation

The sidebar opens the ingress dashboard on port 8099. All generated wordclouds and scraper history are under /data/output.
New versions are published here only after private GHCR images have been built and verified. Home Assistant discovers them on repository refresh; no Samba deployment is required.

Lurker spaces API requests at least 1.1 seconds apart across boards, including retries, and waits at least 10 seconds before requesting the same thread again. The first scrape downloads every sampled thread (about 4 minutes for 200 threads). Later scrapes reuse cached threads where the catalog confirms they are current.

## Options

| Option | Default | Meaning |
| --- | --- | --- |
| boards | biz, pol | Boards to sample each cycle |
| interval_hours | 6 | Hours between scrape cycles |
| retention_days | 30 | Keep wordcloud images and summaries this long |
| history_retention_days | 180 | Keep analytics snapshots this long (the database file is kept) |
| max_threads | 200 | Threads sampled per board, most recently modified first |
| request_delay_ms | 1100 | Minimum spacing between API requests; lower values are raised to 1100 |
| scrape_timeout_seconds | 10800 | Per-board timeout |
| run_on_startup | true | Scrape once when the app starts |
| log_level | info | debug, info, warning or error |
| notification_targets | [] | Opted-in notify.mobile_app_... actions |

The workers option from 0.2 is deprecated and ignored.

## Analytics

The Analytics section charts a topic's mentions per 1,000 sampled posts and its sentiment (−1 to +1) over 7, 30 or 90 days or all history. Summary cards show the latest absolute frequency, normalized rate, robust sentiment and observation count. Gaps mean no data: failed scrapes and partial samples are marked rather than shown as zero.

## Upgrading from 0.2

Existing wordclouds and options are kept. History starts with the first 0.3 scrape; older summaries are not imported. Topic storage defaults to 1500 terms per focus per scrape (`history_max_topics: 0` stores all). The analytics database is included in app backups; lower `history_retention_days` or `history_max_topics` if it grows too large.

## Troubleshooting

Check Settings → System → Logs → Supervisor for repository or image-pull errors. Renew the GHCR token before it expires.
Scrape problems are summarised on the dashboard and in the app log. Repeated HTTP 429 responses mean the API is throttling Lurker: increase request_delay_ms or interval_hours.

## Security

The app enables homeassistant_api to send opt-in mobile notifications. This grants privileged access to the HA Core API inside the container.
Do not put GitHub tokens in app configuration, a Git repository, or chat.


## Historical data quality

The dashboard excludes incomplete samples and runs using incompatible
measurement definitions from trend calculations by default. Opt-in controls
can show them with visible warnings. Lurker stores the measurement definition
alongside each new historical snapshot. On Home Assistant restart it also
records scheduled scraping slots missed during downtime as gaps; no missing
topic values are substituted with zero. SQLite schema migrations preserve
existing historical data. Keep /data/output/history.sqlite3 during upgrades.


## Storage and concurrency (v0.3+)

Historical topic storage defaults to 1500 terms per focus per scrape; set
`history_max_topics: 0` to retain all terms at the cost of disk space.
The dashboard displays SQLite storage (including WAL), with a warning above
256 MiB. Configured boards are safely serialized across concurrent scraping
processes, and API cooldown state is atomically persisted. Malformed cooldown
state stops requests rather than risking an API-rate violation. Changes to
`interval_hours` are detected while the scheduler waits. New historical
snapshots include the start and end of the actual collection window.


## Wordclouds on demand (v0.3)

Home Assistant no longer renders or permanently archives a new PNG after each
scheduled scrape. Instead it saves **all original wordcloud frequencies and
sentiment colours** as a compressed snapshot in the history database, separate
from the top-1,500 historical topic metrics. Opening a wordcloud renders it
from that snapshot without contacting 4chan or rerunning NLP.

- **Latest** and **History** still show wordclouds and support the image viewer.
  New snapshots remain available for `history_retention_days` (default 180).
  Loading many images can use CPU on a Raspberry Pi; the dashboard lazy-loads
  them and the server renders one at a time within each process.
- Generated PNGs go in a temporary, **64 MiB maximum** disk cache; least-recently
  used entries are removed. These are not the source of truth. Removing
  `/data/output/render-cache` while the app is stopped only causes images to
  be rendered again on request.
- Existing pre-upgrade PNGs and JSON summaries continue working; they are
  automatically cleaned up by the old `retention_days` option (default 30).
  Older history records without rendering snapshots cannot be reconstructed
  exactly from the chart metrics.
- Mobile notifications now announce a **new successful analysis**, regardless
  of whether anyone has opened and rendered its wordcloud. Existing opt-in
  recipient settings are unchanged.
- This is an in-place history SQLite migration to schema **v5**. Do **not**
  delete `history.sqlite3` when upgrading. Rendering snapshots add compressed
  aggregate data to that database, so actual disk savings depend on the board.
  The existing 256 MiB history-size warning still applies.
- The CLI remains backward-compatible: `wordcloud` still writes PNGs unless
  `--defer-wordclouds` is supplied with history enabled. The HA scheduler
  supplies that flag.

The rendering snapshot holds aggregate terms and scores, not original post
text. Historical retention removes each snapshot and its rendering inputs
together; it never retains old visual data beyond the configured history period.
