# Changelog

## 0.3.11

- Cache storage usage after scrapes to stop panel timeouts

## 0.3.10

- Drain in-flight board on shutdown; remove workers option
- Quietly clamp request\_delay\_ms below 1100

## 0.3.9

- Restore analytics trends after stopword-score skip

## 0.3.8

- Fix Storage panel hang with request timeout and slow-scan logs
- Keep spaCy NLP cache effective on multi-board Pi schedules
- Raise default NLP cache to 256 MiB with score-first eviction and stopword skip

## 0.3.7

- Fix Home Assistant changelog entries for merged pull requests
- Keep build and development tools out of the Home Assistant image
- Explain scrape progress, partial samples and shutdowns in Lurker logs
- Preserve scheduled scrape outcomes across restarts
- Test Home Assistant image startup and shutdown through s6
- Publish only release images that passed the s6 lifecycle test
- Warm recent wordclouds asynchronously and cancel warming on shutdown
- Record scheduled scrape timing and missed-slot causes
- Expose read-only SQLite history readiness to operators and CI
- Keep wordcloud galleries available across independent source failures
- Document Home Assistant rollback after the history schema upgrade
- Retain schedule overrun evidence until history is committed
- Preserve analytics history across equivalent measurement releases
- Refuse API dispatch when saved cooldowns are corrupt
- Keep slow gallery polls alive until their scope changes
- Address PR #14 code-quality P1/P2 findings
- Keep gallery poll timeouts longer than the status interval

## 0.3.6

- Maintenance and reliability updates.

## 0.3.5

- Maintenance and reliability updates.

## 0.3.4

- Add Rising Topics to the Lurker dashboard

## 0.3.3

- Show release notes in Home Assistant, load Latest images promptly, and share wordclouds

## 0.3.2

- Show recent wordclouds independently of analytics retention

## 0.3.1

- Maintenance and reliability updates.
