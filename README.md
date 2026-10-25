<p align="center"><a href="https://apify.com/DevilScrapes/smartrecruiters-jobs-scraper"><img src="assets/banner.svg" alt="SmartRecruiters Jobs Scraper — JavaScript example" width="100%"></a></p>

# SmartRecruiters Jobs Scraper for JavaScript and Node.js: employer job feeds for your app

<p align="center">
<a href="https://apify.com/DevilScrapes/smartrecruiters-jobs-scraper"><img alt="Run on Apify" src="https://img.shields.io/badge/Run%20on-Apify-DC2626?logo=apify&logoColor=white"></a>
<img alt="JavaScript" src="https://img.shields.io/badge/Node.js-20%2B-339933?logo=nodedotjs&logoColor=white">
<a href="LICENSE"><img alt="MIT license" src="https://img.shields.io/badge/license-MIT-0A0606"></a>
<a href="https://github.com/devilscrapes/smartrecruiters-jobs-scraper-javascript/actions/workflows/ci.yml"><img alt="CI" src="https://github.com/devilscrapes/smartrecruiters-jobs-scraper-javascript/actions/workflows/ci.yml/badge.svg"></a>
</p>

With this repository a Node.js project can call the **[SmartRecruiters Jobs Scraper](https://apify.com/DevilScrapes/smartrecruiters-jobs-scraper)** through a single ES module and Apify's `apify-client` package. Name the employers by the identifier in their `jobs.smartrecruiters.com` careers address, optionally narrow by keyword, city or country, and receive one object per open role with location parts, remote flag, function, employment type, industry, release time and an apply link. Turn on descriptions and each object also carries the posting body as HTML with the official apply URL. Apify does the collecting, so a job board backend, a serverless function or a nightly worker only starts the run and consumes the objects, saved to `results.json` and `results.csv` or kept in memory when imported.

> **Actor:** [SmartRecruiters Jobs Scraper](https://apify.com/DevilScrapes/smartrecruiters-jobs-scraper) · **Publisher:** [Devil Scrapes](https://apify.com/DevilScrapes) · **Also in Python:** [smartrecruiters-jobs-scraper-python](https://github.com/devilscrapes/smartrecruiters-jobs-scraper-python)

## 🔥 What you get

- **Identifier or link, either works**: a careers address such as `jobs.smartrecruiters.com/Visa` is reduced to its first path segment; anything without the SmartRecruiters host is sent as written, capitals included.
- **Paging follows the employer's own total**: postings are requested 100 at a time until the count SmartRecruiters reports for that employer is reached or your per-company cap is hit, whichever comes first.
- **Filters applied at the source**: keyword, department, city and country travel with every page request, so postings outside your filter are never downloaded or billed. Country codes are lowercased for you.
- **Descriptions on demand**: with `includeDescription` on, each posting's company blurb, job text, qualifications and additional information are joined into one HTML string under `<h2>` headings, and the apply link is replaced with the one SmartRecruiters returns for that posting.
- **One employer failing leaves the others running**: when an employer's requests keep failing after retries, it is marked failed and the run moves on. The final status message counts how many employers returned data.
- **Empty boards are a normal answer**: an employer with no open postings contributes zero rows without raising an error.

## 🎯 Use cases

### Power a niche job board

Pick the SmartRecruiters employers that belong on your board, run them nightly and upsert the objects keyed by `posting_id`. Postings that disappear from the run can be expired on your site the same night, so candidates stop landing on closed roles.

### Publish structured job data for search engines

Each object maps cleanly onto schema.org `JobPosting`: title, release date, employer name, address parts and employment type are all present. Rendering that markup on your own pages is how job listings are usually described to search engines.

### Hiring alerts in team chat

A serverless function on a schedule can run a few target employers with a keyword such as `data engineer`, compare `posting_id` values with the previous batch and post the new ones to a Slack or Teams webhook with title, city and apply link.

### Keep an ATS integration honest

Vendors integrating with employers' SmartRecruiters boards can use the objects as an outside view of what is publicly live, and flag records in their own system that the public board no longer lists.

## ⚡ Quick start

You need an Apify account and an API token from [Console → Settings → Integrations](https://console.apify.com/settings/integrations). Apify's free plan includes monthly usage credit; this Actor's event prices are listed under Pricing below.

```bash
git clone https://github.com/devilscrapes/smartrecruiters-jobs-scraper-javascript.git
cd smartrecruiters-jobs-scraper-javascript
npm install
export APIFY_TOKEN=your_token_here
node index.js                       # uses input.json
node index.js my-input.json --out data --max-charge 2
```

The script starts the [SmartRecruiters Jobs Scraper](https://apify.com/DevilScrapes/smartrecruiters-jobs-scraper), waits for it to finish, prints a link to the run in Apify Console and saves every result to `results.json` and `results.csv`. It exits with code 1 and the run's status message if the run fails.

## 🧩 The core call

Here is all of the Actor integration that `index.js` contains, minus option parsing and file output. `call()` resolves when the run has finished; `listItems()` fetches what the run stored.

```javascript
import { ApifyClient } from 'apify-client';

const client = new ApifyClient({ token: 'YOUR_APIFY_TOKEN' });
const run = await client.actor('DevilScrapes/smartrecruiters-jobs-scraper').call({"companyIds": ["Visa"], "maxResultsPerCompany": 10, "includeDescription": false, "proxyConfiguration": {"useApifyProxy": true}});
const { items } = await client.dataset(run.defaultDatasetId).listItems();
console.log(items);
```

## 📥 Input

Whatever is in `input.json` is sent as the run input. The default looks at Visa's board, returns at most ten postings and skips descriptions, which keeps a first call to a start event and ten row events or fewer. A backend would construct this object per job, for example from the employer list in its own database.

```json
{
  "companyIds": [
    "Visa"
  ],
  "maxResultsPerCompany": 10,
  "includeDescription": false,
  "proxyConfiguration": {
    "useApifyProxy": true
  }
}
```

| Field | Type | Required | Default | Allowed values |
|---|---|---|---|---|
| `companyIds` | array | yes | `["Visa"]` | — |
| `searchQuery` | string | no | — | — |
| `department` | string | no | — | — |
| `city` | string | no | — | — |
| `country` | string | no | — | — |
| `maxResultsPerCompany` | integer | no | `100` | — |
| `includeDescription` | boolean | no | `false` | — |
| `proxyConfiguration` | object | no | `{"useApifyProxy": true}` | — |

## 📤 Output

Every object represents one open posting. Use `posting_id` as your primary key. Country codes are lowercase ISO-style pairs, so uppercase them before showing them to users. `released_date` is an ISO timestamp you can hand to `new Date()` directly. `description_html` is `null` when descriptions were off and otherwise contains section headings as `<h2>` elements, which you should sanitize before rendering in a browser.

| Field | Type | Always present |
|---|---|---|
| `posting_id` | string | yes |
| `title` | string | yes |
| `company_id` | string | yes |
| `company_name` | string | yes |
| `location_city` | string or null | no |
| `location_region` | string or null | no |
| `location_country` | string or null | no |
| `is_remote` | boolean | yes |
| `department` | string or null | no |
| `function` | string or null | no |
| `employment_type` | string or null | no |
| `industry` | string or null | no |
| `ref_number` | string or null | no |
| `released_date` | string | yes |
| `apply_url` | string | yes |
| `description_html` | string or null | no |
| `scraped_at` | string | yes |

## 💰 Pricing

| Event | Price (USD) |
|---|---|
| Actor start | $0.2 |
| Result emitted | $0.002 |

Prices as configured on 2026-08-20. The [Actor page](https://apify.com/DevilScrapes/smartrecruiters-jobs-scraper) always shows the current price. Runs are billed to the Apify account that owns the token. Use `--max-charge` to cap what a single run can cost.

## 🛠️ Working with the data in JavaScript

The script below uses only Node built-ins. It converts `results.json` into schema.org `JobPosting` objects, maps SmartRecruiters' employment labels onto the vocabulary search engines expect, skips postings without a description (the markup requires one) and writes `jobposting.jsonld.json`. Set `includeDescription` to `true` before the run that feeds it, since the bundled input leaves descriptions off and would produce an empty file.

```javascript
import { readFile, writeFile } from 'node:fs/promises';

const rows = JSON.parse(await readFile('results.json', 'utf8'));
const EMPLOYMENT = { 'full-time': 'FULL_TIME', 'part-time': 'PART_TIME', contract: 'CONTRACTOR', intern: 'INTERN' };
const employmentType = (label) => {
    const key = Object.keys(EMPLOYMENT).find((k) => (label ?? '').toLowerCase().includes(k));
    return key ? EMPLOYMENT[key] : 'OTHER';
};

const postings = rows
    .filter((row) => row.description_html)
    .map((row) => ({
        '@context': 'https://schema.org',
        '@type': 'JobPosting',
        identifier: row.posting_id,
        title: row.title,
        description: row.description_html,
        datePosted: row.released_date,
        employmentType: employmentType(row.employment_type),
        hiringOrganization: { '@type': 'Organization', name: row.company_name },
        jobLocation: {
            '@type': 'Place',
            address: {
                '@type': 'PostalAddress',
                addressLocality: row.location_city,
                addressRegion: row.location_region,
                addressCountry: row.location_country?.toUpperCase(),
            },
        },
        ...(row.is_remote ? { jobLocationType: 'TELECOMMUTE' } : {}),
        url: row.apply_url,
    }));

await writeFile('jobposting.jsonld.json', JSON.stringify(postings, null, 2));
console.log(`${postings.length} of ${rows.length} postings converted (rows without description skipped)`);
```

## ❓ FAQ

### Does SmartRecruiters offer an API for this?

SmartRecruiters publishes a postings feed that its careers boards read from, and the Actor reads the same feed, so no SmartRecruiters account or key is involved; your Apify token is enough. What the Actor adds on top is paging, retries, filters and one object shape for every employer.

### How do I know if an employer failed during a run?

The run ends as succeeded even when some employers could not be fetched, because rows already collected are worth keeping. Fetch the run object after `call()` and read `statusMessage`; it reports how many employers out of the total returned data.

### Does it work with TypeScript?

Yes. `apify-client` includes its own typings. Declare an interface for the output fields, with `description_html` typed as `string | null`, and pass it as the generic to `client.dataset()` to get typed objects in your handlers.

### How large can one employer's board be?

Postings are fetched in pages of 100, and `maxResultsPerCompany` accepts up to 5,000 per employer. For large enterprise boards, keep the cap generous in production and small during development, and add `--max-charge` as a hard spending limit.

## 🧯 Troubleshooting

- **`Set APIFY_TOKEN first`**: export the token in the same shell, or put it in a local `.env` file your shell loads. Never commit it.
- **Run finished with `FAILED` or `ABORTED`**: open the printed Console link; the run log says why. Input validation errors name the field.
- **Fewer rows than expected**: a `--max-charge` cap also limits how many results can be charged. Raise the cap or lower the limits in your input.
- **Something looks wrong in the data**: open an issue on the [Actor page](https://apify.com/DevilScrapes/smartrecruiters-jobs-scraper) so the maintainers see it.

## 🔗 More from Devil Scrapes

- [SmartRecruiters Jobs Scraper on Apify Store](https://apify.com/DevilScrapes/smartrecruiters-jobs-scraper)
- [Python version of this example](https://github.com/devilscrapes/smartrecruiters-jobs-scraper-python)
- [All Devil Scrapes Actors](https://apify.com/DevilScrapes)
- [Apify web scraping cookbook](https://github.com/devilscrapes/apify-web-scraping-cookbook)

Related Actors:

- [Workday Jobs Scraper](https://apify.com/DevilScrapes/workday-jobs-scraper) — openings from large employers whose careers sites run on Workday.
- [Workable Jobs Scraper](https://apify.com/DevilScrapes/workable-jobs-scraper) — boards hosted on Workable, common among smaller tech firms.
- [Lever Jobs Scraper](https://apify.com/DevilScrapes/lever-jobs-scraper) — jobs.lever.co boards, split by team and department.

---

MIT licensed example code. The Actor itself runs on [Apify](https://apify.com/DevilScrapes/smartrecruiters-jobs-scraper) and is maintained by [Devil Scrapes](https://apify.com/DevilScrapes). Not affiliated with SmartRecruiters.
