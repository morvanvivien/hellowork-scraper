# How to Scrape HelloWork Job Postings (Salary, Remote Policy, No Login)

> **Run it now, no code: [hellowork-jobs-scraper on the Apify Store](https://apify.com/bibian/hellowork-jobs-scraper)** &nbsp;|&nbsp; $0.60 per 1,000 jobs &nbsp;|&nbsp; no account or login needed

HelloWork is one of France's largest job boards, covering postings from local retail roles to senior management and IT positions. It has no public export and no official API for third parties. This guide shows how to get HelloWork job postings as clean structured data: title, company, salary when published, remote policy, location and full description, with a free Apify Actor, from the console or from code.

Searches this page answers: hellowork scraper, hellowork job postings api, france job board scraper, hellowork salary data.

## What you get from the HelloWork scraper

- Every job posting matching your sector and keywords, with title, occupation category, employment type and remote policy.
- Salary range (`salaryMin`, `salaryMax`, `salaryCurrency`, `salaryPeriod`) when the employer publishes one.
- Hiring company name, industry and office location (city, region).
- Full job description as plain text, with posting and expiry dates.

Export the dataset as JSON, CSV or Excel, or read it through the Apify API. A real sample (five results) is in [`sample-output.json`](sample-output.json).

## Output fields

| Field | What it is |
|---|---|
| `id`, `url` | HelloWork job id and public link |
| `title`, `occupation` | Job title and occupational category |
| `companyName`, `industry` | Hiring company and its industry |
| `city`, `region` | Office location |
| `remote` | Whether the posting is flagged as remote-friendly (`TELECOMMUTE`) |
| `salaryMin`, `salaryMax`, `salaryCurrency`, `salaryPeriod` | Salary range, when published |
| `employmentType` | Contract type (`FULL_TIME`, `PART_TIME`...) |
| `datePosted`, `validThrough` | Publication date and expiry date |
| `description` | Plain text job description |

## Use cases

- **Monitor the French job market.** New postings by sector, with salary and remote-work data when the employer discloses it.
- **Feed a job alert or an aggregator.** Run it daily on a sector and dedupe by `id` to catch only the new postings.
- **Find companies that are hiring.** See which employers recruit for a given role right now, in which city.
- **Benchmark salaries and remote policy.** Compare published salary ranges and `TELECOMMUTE` postings by sector.

## Input

| Field | What it does |
|---|---|
| `domaine` | HelloWork sector category to browse, for example `informatique` or `sante`. Empty rotates through the busiest sectors. |
| `keywords` | Keep only postings whose title or occupational category mentions these words. Empty keeps every posting in the sector. |
| `maxItems` | The run stops once this many postings are saved (default 50). |
| `includeDescriptions` | Adds the full job description as plain text. |

Example input:

```json
{
  "domaine": "informatique",
  "keywords": "",
  "maxItems": 50,
  "includeDescriptions": true
}
```

## Three ways to use it

### 1. Apify console, no code

1. Open [the Actor page](https://apify.com/bibian/hellowork-jobs-scraper) and click **Try for free** (a free Apify account is enough).
2. Fill in your sector and keywords and click **Start**.
3. When the run finishes, open the **Output** tab, check the table, then **Export** as CSV, Excel or JSON.
4. For a daily alert, open **Schedules**, create a daily schedule for the Actor and, optionally, add an integration that sends new postings to Slack, e-mail or Google Sheets.

### 2. Apify API with JavaScript

```bash
npm install apify-client
export APIFY_TOKEN="your token from Apify console, Settings, Integrations"
node examples/run.js
```

```js
import { ApifyClient } from 'apify-client';

const client = new ApifyClient({ token: process.env.APIFY_TOKEN });

const run = await client.actor('bibian/hellowork-jobs-scraper').call({
    "domaine": "informatique",
    "keywords": "",
    "maxItems": 50,
    "includeDescriptions": true
});

const { items } = await client.dataset(run.defaultDatasetId).listItems();
for (const item of items) {
    console.log(`${item.title} | ${item.companyName} | ${item.city} | ${item.salaryMin ?? '-'}-${item.salaryMax ?? '-'} ${item.salaryCurrency ?? ''}`);
}
```

The complete file is [`examples/run.js`](examples/run.js) (ES module, Node 18 or later).

### 3. Apify API with Python

```bash
pip install apify-client
export APIFY_TOKEN="your token from Apify console, Settings, Integrations"
python examples/run.py
```

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])

run = client.actor("bibian/hellowork-jobs-scraper").call(run_input={
    "domaine": "informatique",
    "keywords": "",
    "maxItems": 50,
    "includeDescriptions": True,
})

for item in client.dataset(run.default_dataset_id).iterate_items():
    print(f"{item['title']} | {item.get('companyName')} | {item.get('city')} | {item.get('salaryMin')}-{item.get('salaryMax')} {item.get('salaryCurrency') or ''}")
```

The complete file is [`examples/run.py`](examples/run.py). Never put your token in the code: read it from the `APIFY_TOKEN` environment variable as above.

## Pricing

Pay per result: **$0.60 per 1,000 jobs** (that is $0.0006 per job posting), plus a tiny start fee of $0.00005 per run. You pay only for the results saved to the dataset. Apify's free plan includes monthly credits, enough to try it on a few hundred results. Prices are those shown on the [Store page](https://apify.com/bibian/hellowork-jobs-scraper) when this guide was written; the Store page is the reference.

## Limits and honest notes

- HelloWork's search page is off-limits to robots (its own robots.txt blocks `/fr-fr/emploi/recherche.html` and any `?` query string); this actor instead browses the sector category pages, which the site explicitly allows.
- Each sector page lists about 20 postings with no further pagination, so a very narrow sector plus a strict keyword filter can return fewer results than `maxItems`.
- `salaryMin`/`salaryMax` are `null` on most postings: French job boards rarely publish a salary, this is not a scraping gap (visible in `sample-output.json`).
- Only the hiring company and office location are collected: no recruiter or employee personal data.
- This is an unofficial tool, not affiliated with HelloWork. Check the site terms and the data protection rules that apply to your use.

## FAQ

**Does it need a browser or a proxy?**  
No. Every page used is a plain HTTP request to pages the site's own robots.txt allows.

**How do I only get new postings every day?**  
Schedule the Actor once a day with your usual sector and keywords. Results come newest first, so duplicates across runs are easy to filter out by `id`.

**Does it collect recruiter contact details?**  
No. Only the hiring company's public details (name, industry) and the office location are collected, never an individual's data. A line that carries a recruiter's e-mail address is dropped from the description before it is saved.

## Try it

**[Open hellowork-jobs-scraper on the Apify Store](https://apify.com/bibian/hellowork-jobs-scraper)**

The code in `examples/` is released under the MIT license (see [LICENSE](LICENSE)). The data belongs to its original publishers.
