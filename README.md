# Track observed Zillow price changes with n8n and Google Sheets

An inactive DataSmithLab starter for a small ZIP-level listing log. Compare prices only when the same listing is observed again.

[Zillow scraper on Apify](https://apify.com/datasmithlab/zillow-property-scraper) · [Walkthrough](https://datasmith-lab.github.io/datasmithlab-zillow-n8n/) · [Download ZIP](https://github.com/Datasmith-lab/datasmithlab-zillow-n8n/releases/latest/download/zillow-prices-to-sheets-starter.zip)

## Setup

1. Create a dedicated **Listings** tab in your Google Sheet and paste `listings-sheet-headers.csv` into A1. Preserve your other tabs; an existing Listings table must use exactly these headers.
2. Import `zillow-prices-to-sheets.n8n.json`. It is inactive and contains no credentials.
3. In Configuration enter your own spreadsheet ID. The default is sale listings in ZIP **78701**, maxItems 20, Actor tracking off and summaries off.
4. Create n8n Header Auth with name `Authorization` and value `Bearer YOUR_APIFY_TOKEN`. Select it on all Apify HTTP nodes. Keep the token in n8n credentials, not in code, Git or Sheets.
5. Connect your own Google Sheets OAuth2 credential on Read saved listings and Save listings.
6. Run manually and inspect HEALTH, SEARCHES and the resulting Sheet. Repeat to verify stable keys and unchanged prices.
7. After validation enable the daily **09:30 UTC** schedule. Self-hosted n8n must remain running. One execution at a time; concurrent writes are not supported.

The example supports one sale ZIP and a 1–100 row cap. Changing the ZIP or filters starts a new scope. The configured $0.10 Actor charge cap does not guarantee total n8n/cloud/Google costs; Store pricing is authoritative.

## How changes are interpreted

First observations are BASELINE. A lower numeric price for the same saved ID produces PRICE_DROP; a higher price produces PRICE_INCREASE. The table retains the most recent price event with `priceChangedAt`, `previousPrice`, `priceChange` and `priceChangePct`. An unchanged repeat does not create a new event; the retained event is not a fresh alert. The execution summary lists only price changes observed in that execution.

Missing observed prices preserve the last known price and mark `sourceStatus` as `observed-price-missing`. Zillow's `zillowPriceChange` is copied separately to `sourcePriceChange`; it is not treated as a new change between our scans. Listings absent from a later bounded snapshot are retained. Absence does not prove sold, withdrawn or removed. No notifications are sent.

HEALTH/search failures, unreadable records, charge-limit truncation, count mismatch or duplicate/invalid identifiers stop before writes. Partial Google Sheets writes can still occur; inspect the error, fix it and rerun sequentially. Upserts are not a transactional guarantee.

## Coverage and validation

The live example reads 20 listings from a ZIP with more than 20 available listings. It is not a full ZIP-market scan. A status=ok report does not establish completeness, and sampling can miss price changes. No valuation, complete market statistics or sales conclusions are provided.

On 11 October 2026, two real manual snapshots and an actual timed-trigger execution each observed 20 listings. Final Google Sheets API read-back verified 20 unique keys and zero duplicates. Repeat and scheduled runs found no new IDs or price differences. Each execution launched one Actor. SEARCHES reported 221 available listings, so this verifies a small sample.

Eleven synthetic scenarios cover price drops, increases, unchanged repeats, missing prices, absent listings and failure checks. The synthetic -10% drop is not evidence of a real market price drop. Tested on self-hosted n8n 2.26.4; other versions require your setup check. No official n8n template-directory acceptance claimed.
