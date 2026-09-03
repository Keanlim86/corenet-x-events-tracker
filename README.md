# CORENET X Events Tracker

Storage repo for a daily automated check of the [CORENET X Past Events page](https://info.corenet.gov.sg/resources/past--events).

A scheduled Claude Code cloud routine runs every day at 23:59 Asia/Singapore time. Each run:

1. Fetches the current page content.
2. Compares it to `snapshot.txt` (the content saved from the previous run).
3. If the content differs, sends a push notification summarizing what changed, and commits the updated content to `snapshot.txt` with a descriptive commit message.
4. If nothing changed, does nothing further (no commit).

Git history on `snapshot.txt` doubles as a changelog of every detected update to the page.

## Files

- `snapshot.txt` — the normalized text content of the Past Events page as of the last detected change.
