CORENET X Events Tracker
Storage repo for a daily automated check of two CORENET X pages:
Past Events → `snapshot.txt`
Circulars → `snapshot-circulars.txt`
A scheduled Claude Code cloud routine runs every day at 23:59 Asia/Singapore time. For each tracked page, each run:
Fetches the current page content.
Compares it to the page's snapshot file (the content saved from the previous run).
If the content differs, sends a push notification summarizing what changed, and commits the updated content to that snapshot file with a descriptive commit message.
If nothing changed for a page, does nothing further for it (no commit).
Git history on each snapshot file doubles as a changelog of every detected update to that page.
Files
`snapshot.txt` — the normalized text content of the Past Events page as of the last detected change.
`snapshot-circulars.txt` — the normalized text of the newest circulars (see below) as of the last detected change.
Fetching notes
Past Events
The page is server-rendered — a plain HTTP GET of the URL returns the full event listing in the HTML. Strip tags/scripts and normalize whitespace to compare against `snapshot.txt`.
Circulars
The circulars list is not in the server-rendered HTML — it's loaded client-side by a React widget (`CircularsLayout`) that calls a JSON API. A plain page fetch will show no circulars. To get the data:
`GET https://info.corenet.gov.sg/api/token/init` with browser-like headers (`User-Agent`, `Referer: https://info.corenet.gov.sg/resources/circulars`, `Origin: https://info.corenet.gov.sg`, `Accept: application/json, text/plain, */*`) — requests without these headers get a `403 Access denied`. Returns `{"accessToken": "..."}` (a short-lived JWT).
`POST https://info.corenet.gov.sg/api/Circulars/list` with `Authorization: Bearer <accessToken>` and the same browser-like headers, body:
```json
   {"page":1,"pageSize":100,"searchString":null,"startDate":null,"endDate":null,"selectedAgencies":[],"selectedRegulatories":[],"selectedNonRegulatories":[],"clickedKeyword":null,"clickedAgency":null}
   ```
Returns `{"items": [...], "currentPage": 1, "totalPages": N}`. Items are sorted newest-first by `publishedDate`. `pageSize` is capped server-side (100 works; large values like 2000 return `500`).
Since there are ~1700 circulars in total but they're sorted newest-first, only the first page (100 items) is tracked — plenty of headroom for a daily check. Each item has `title`, `contentDescription`, `agencies`, `regulatoryCategories`/`nonRegulatoryCategories`, `attachmentUrl`, `effectiveDate`, and `publishedDate`. `snapshot-circulars.txt` renders each item as a short block (date, title, agencies, categories, description, link) — see the file for the exact format. Diff against it to find new/changed/removed circulars in the top 100.
Note: headless-browser rendering (Playwright/Chromium) does not work in this environment for this site — the sandboxed network proxy resets the connection for browser TLS handshakes. Use the direct API calls above instead.
