
# n8n Workflow: URL Indexing Status Checker (Google Search Console) 🔍

This n8n workflow automatically checks whether your website URLs are **indexed** in Google Search using the **Google Search Console URL Inspection API**.  
It reads URLs from **Google Sheets**, inspects each URL, and then **updates the same sheet** with the indexing result and related details.

---

## 🚀 What it does

- Reads a list of URLs from Google Sheets (column: `url`)
- Sends each URL to **Google Search Console URL Inspection**
- Detects if the page is:
  - ✅ Indexed
  - ❌ Not Indexed
- Writes back the inspection details (coverage state, verdict, crawl time, etc.)
- Runs automatically on schedule (e.g. every day at 10:00 AM)

---

## 📁 Repository Structure

```

.
├─ workflow/
│  └─ url_index_checker.json     # exported n8n workflow
├─ images/
│  └─ workflow-diagram.png       # workflow diagram / screenshot
└─ README.md

````

---

## ⚙️ Requirements

- **n8n** (self-hosted or n8n Cloud)
- **Community node:** `n8n-nodes-google-search-console`
- **Google Cloud credentials** with Search Console URL Inspection access
- A **verified property** in Google Search Console (matching your `siteUrl`)
- **Google Sheets** credentials connected to n8n

---

## 🧩 Setup Instructions

1. **Create your Google Sheet**
   - Add columns:
     - `url`
     - `status`
     - `coverageState`
     - `verdict`
     - `lastCrawlTime`
     - `pageFetchState`
     - `robotsTxtState`
   - Add at least two test URLs:
     - One that is indexed
     - One that is not indexed

2. **Import the workflow**
   - Go to your n8n instance ➜ *Import Workflow*  
   - Select `workflow/url_index_checker.json`

3. **Configure credentials**
   - Connect your **Google Sheets** and **Google Search Console** credentials.
   - Make sure your GSC property (`siteUrl`) matches exactly (including `https://` and trailing `/`).

4. **Configure nodes**
   - **Google Sheets – Read:** gets URLs from your sheet.
   - **Google Search Console – URL Inspection:**  
     - `siteUrl`: `https://yourdomain.com/`
     - `inspectionUrl`: `{{ $json.url }}`
   - **Google Sheets – Append/Update:** writes results back into the same sheet.

5. **Add Scheduler (optional)**
   - Add a **Cron node** to run daily at **10:00 AM** and keep your data updated automatically.

---

## 🧠 Key Expressions

**Status detection example:**
```js
{{
  ($json.indexStatusResult?.coverageState || $json.coverageState || "")
    .toLowerCase()
    .includes("indexed")
    ? "Indexed"
    : "Not Indexed"
}}
````

> ✅ This expression automatically sets the status to "Indexed" or "Not Indexed"
> 💡 Works even if your node output is nested or flattened.

---

## 📊 Example (Google Sheets Output)

| url                                                                          | status      | coverageState            | verdict | lastCrawlTime        | pageFetchState | robotsTxtState |
| ---------------------------------------------------------------------------- | ----------- | ------------------------ | ------- | -------------------- | -------------- | -------------- |
| [https://example.com/indexed-page](https://example.com/indexed-page)         | Indexed     | Submitted and indexed    | PASS    | 2025-10-12T08:41:02Z | SUCCESS        | ALLOWED        |
| [https://example.com/not-indexed-page](https://example.com/not-indexed-page) | Not Indexed | Discovered – not indexed | NEUTRAL | —                    | SOFT_404       | ALLOWED        |

---

## 🧰 Troubleshooting

* **No results?**
  Ensure your `siteUrl` matches the verified property in Search Console exactly (protocol + trailing slash).
* **Large number of URLs?**
  Add a **Split In Batches** node to respect API rate limits.
* **Unclear data?**
  Insert a temporary **Function** node to `return [{raw: $json}]` and inspect the full API response.

---

## 🔗 Related Projects

* **n8n – Cannibalization Link Detector**
  [https://github.com/zahraraeisi/n8n-cannibalization-link-detector](https://github.com/zahraraeisi/n8n-cannibalization-link-detector)

---

## 👩‍💻 Author

**Zahra Raeisi**
Automation & Applied AI (n8n)
GitHub: [https://github.com/zahraraeisi](https://github.com/zahraraeisi)

