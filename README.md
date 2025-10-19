
---

## Requirements

- **n8n** (self-hosted or n8n Cloud)
- **Community node**: `n8n-nodes-google-search-console`
- **Google Cloud credentials** with Search Console URL Inspection access
- A **verified property** in Google Search Console (must match `siteUrl`)
- **Google Sheets** credentials connected in n8n

---

## Setup (quick)

1. **Google Sheet**
   - Create a sheet (e.g., tab name: `URLs`) with at least these columns in row 1:
     - `url`, `status`, `coverageState`, `verdict`, `lastCrawlTime`, `pageFetchState`, `robotsTxtState`
   - Add a couple of test rows (one indexed URL, one not indexed).

2. **Import workflow**
   - Import `workflow/url_index_checker.json` into n8n.
   - Set credentials for **Google Sheets** and **Google Search Console**.

3. **Configure nodes (key fields)**
   - **GSC – URL Inspection**
     - `siteUrl`: your verified property, e.g. `https://example.com/`
     - `inspectionUrl`: `{{ $json.url }}`
     - `languageCode`: `en`
   - **Google Sheets – Read**: read all rows from your sheet/tab.
   - **Set** (optional but recommended): normalize fields from inspection result:
     ```js
     {
       url: $json.url || $prevNode["Google Sheets – Read"].json.url,
       coverageState: $json.indexStatusResult?.coverageState || $json.coverageState,
       verdict: $json.indexStatusResult?.verdict || $json.verdict,
       lastCrawlTime: $json.indexStatusResult?.lastCrawlTime || $json.lastCrawlTime,
       pageFetchState: $json.indexStatusResult?.pageFetchState || $json.pageFetchState,
       robotsTxtState: $json.indexStatusResult?.robotsTxtState || $json.robotsTxtState,
       status: (
         ($json.indexStatusResult?.coverageState || $json.coverageState || "")
           .toLowerCase()
           .includes("indexed")
       ) ? "Indexed" : "Not Indexed"
     }
     ```
   - **Google Sheets – Append or Update row**
     - Match on: `url`
     - Values to send: the fields above.

4. **Schedule (optional)**
   - Add a **Cron** node to run **daily at 10:00** (server time).  
     > If you need Europe/Berlin, align your n8n instance timezone accordingly.

---

## Field mapping quick-reference

- **Input**  
  - `inspectionUrl`: `{{ $json.url }}`
  - `siteUrl`: **must equal** the verified property in GSC (protocol + trailing slash).

- **Status expression**
  ```js
  {{
    ($json.indexStatusResult?.coverageState || $json.coverageState || "")
      .toLowerCase()
      .includes("indexed") ? "Indexed" : "Not Indexed"
  }}
