---
name: meta-ads-cli
description: Operate Meta Ads through Meta's official Ads CLI (`meta-ads`) for campaign management, asset discovery, insights extraction, and analytics workflows.
version: 1.0.0
author: Hermes Agent
license: Internal
platforms: [linux, macos]
prerequisites:
  commands: [meta]
metadata:
  hermes:
    tags: [meta, ads, marketing, cli, analytics, automation]
    homepage: https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/ads-cli-overview
---

# Meta Ads CLI

Use this skill when the user wants to manage Meta/Facebook ads from the terminal: discover ad accounts, create campaigns/ad sets/ads/creatives, work with datasets/catalogs, or pull insights and analytics.

This skill is built around Meta's official Ads CLI, published on Apr 29, 2026 and exposed as the `meta` command via the `meta-ads` Python package.

---

## Safety rules

### Secret hygiene
- **Never** ask the user to paste Meta access tokens into chat.
- **Never** print or read `.env` files containing live tokens unless the user explicitly asks and understands the risk.
- Prefer checking presence of env vars over displaying their values.
- Do not echo `ACCESS_TOKEN` back to the user.

### Operational guardrails
- Default new resources to **PAUSED** unless the user explicitly wants activation.
- Treat deletes as **dangerous and cascading**.
- Before any write, verify auth and account context.
- Prefer read/list/preview steps before mutating anything.
- Prefer `meta --output json ...` for automation and parsing.

### Caveats
- This is a **day-one alpha** CLI.
- Docs and package examples may not perfectly match.
- No public official GitHub repo is currently known.
- If a command appears missing from docs but hinted in package metadata (`product-set`, `product-item`, `product-feed`), treat it as underdocumented and verify with `--help` before use.

---

## Install / verify

Install:

```bash
python3 -m pip install meta-ads
```

Verify:

```bash
meta --version
meta --help
meta auth status
```

If `meta` is not found after install, check the Python user bin path or active virtualenv.

---

## Required configuration

Expected environment variables:

- `ACCESS_TOKEN` — required
- `AD_ACCOUNT_ID` — required for most ad operations
- `BUSINESS_ID` — useful for datasets/catalogs/business-scoped work

Expected auth model:
- Meta **system user access token** with the right assets attached

Commonly required scopes:
- `business_management`
- `ads_management`
- `pages_show_list`
- `pages_read_engagement`
- `pages_manage_ads`
- `catalog_management`
- `read_insights`

---

## Known-good command form

Prefer global flags **before** subcommands:

```bash
meta --output json ads campaign list
```

Even if some examples show trailing `--output json`, use the form above for consistency.

---

## Preflight checklist

Before writes, run these in order:

```bash
meta auth status
meta --output json ads adaccount list
meta --output json ads adaccount current
meta --output json ads page list
```

If the workflow needs conversion tracking or commerce assets, also inspect:

```bash
meta --output json ads dataset list
meta --output json ads catalog list
```

What to verify:
- auth works
- correct ad account is selected
- page exists and is accessible
- required dataset/pixel/catalog exists
- media files exist locally before creative creation

---

## Core workflow recipes

### 1) Discover accounts and pages

```bash
meta --output json ads adaccount list
meta --output json ads adaccount current
meta --output json ads page list
```

Use this first whenever the user is vague about which assets to operate on.

### 2) Create a campaign safely

```bash
meta --output json ads campaign create \
  --name "Traffic Campaign" \
  --objective OUTCOME_TRAFFIC \
  --daily-budget 5000
```

Immediately capture the returned campaign ID.

### 3) Create an ad set

```bash
meta --output json ads adset create <CAMPAIGN_ID> \
  --name "US Audience" \
  --optimization-goal LINK_CLICKS \
  --billing-event IMPRESSIONS \
  --targeting-countries US
```

For conversion workflows:

```bash
meta --output json ads adset create <CAMPAIGN_ID> \
  --name "Purchase Set" \
  --optimization-goal OFFSITE_CONVERSIONS \
  --billing-event IMPRESSIONS \
  --pixel-id <PIXEL_ID> \
  --custom-event-type PURCHASE \
  --targeting-countries US
```

### 4) Create creatives

Image creative:

```bash
meta --output json ads creative create \
  --name "Hero Banner" \
  --page-id <PAGE_ID> \
  --image ./banner.jpg \
  --body "Check out our latest deals" \
  --title "Shop Now" \
  --link-url https://example.com \
  --call-to-action SHOP_NOW
```

Video creative:

```bash
meta --output json ads creative create \
  --name "Video Promo" \
  --page-id <PAGE_ID> \
  --video ./promo.mp4 \
  --body "Watch our new collection" \
  --title "New Arrivals" \
  --link-url https://example.com/new
```

Dynamic creative optimization example:

```bash
meta --output json ads creative create \
  --name "DCO Test" \
  --page-id <PAGE_ID> \
  --link-url https://example.com \
  --images ./img1.jpg --images ./img2.jpg \
  --titles "Shop Now" --titles "Learn More" \
  --bodies "50% off today" --bodies "Free shipping today" \
  --call-to-actions SHOP_NOW --call-to-actions LEARN_MORE
```

### 5) Create ads and activate deliberately

```bash
meta --output json ads ad create <AD_SET_ID> \
  --name "Hero Banner Ad" \
  --creative-id <CREATIVE_ID>
```

Only activate if explicitly requested:

```bash
meta --output json ads campaign update <CAMPAIGN_ID> --status ACTIVE
meta --output json ads adset update <AD_SET_ID> --status ACTIVE
meta --output json ads ad update <AD_ID> --status ACTIVE
```

### 6) Pull insights / analytics

Quick checks:

```bash
meta --output json ads insights get --date-preset yesterday
meta --output json ads insights get --date-preset last_30d
```

Campaign-specific report:

```bash
meta --output json ads insights get \
  --campaign-id <CAMPAIGN_ID> \
  --date-preset last_30d \
  --fields spend,impressions,clicks,ctr,cpc,purchase_roas \
  --sort spend_descending
```

Breakdowns:

```bash
meta --output json ads insights get \
  --since 2026-04-01 \
  --until 2026-04-29 \
  --fields spend,impressions,clicks,ctr,cpc \
  --breakdown age \
  --breakdown gender
```

### 7) Dataset / catalog workflows

Inspect first:

```bash
meta --output json ads dataset list
meta --output json ads catalog list
```

If the user needs conversion tracking or commerce workflows, verify dataset/pixel or catalog availability before building campaigns that depend on them.

---

## Recommended agent workflow

### For management tasks
1. Verify auth/context
2. List relevant assets
3. Ask/confirm the target account/page/campaign if ambiguous
4. Create or update resources in the correct order
5. Capture all returned IDs
6. Leave paused unless activation is explicitly requested
7. Summarize what changed

### For analytics tasks
1. Verify auth/context
2. Determine reporting scope: account / campaign / ad set / ad
3. Determine date window and fields
4. Use `--output json`
5. Parse and compute derived metrics if needed
6. Return a concise narrative plus structured numbers

---

## Common pitfalls

- Wrong ad account selected
- Page exists but system user lacks access
- Pixel/dataset not connected when creating conversion workflows
- Media file path wrong or missing
- Attempting to update immutable creative fields
- Using destructive commands without realizing they cascade to children
- Assuming docs cover every subcommand in this alpha release

---

## Troubleshooting

### Auth seems valid but commands fail
Usually an asset-permission problem, not a token-format problem. Check:
- system user role
- ad account assignment
- page assignment
- dataset/catalog assignment
- app admin linkage

### `--output json` behaves oddly
Move global flags earlier:

```bash
meta --output json ads campaign list
```

### Conversion workflow errors
Check that the dataset/pixel exists and is attached to the relevant business/account context.

### Creative updates fail
Some creative properties may be effectively immutable. Recreate the creative instead of trying to patch it.

---

## Good reporting patterns

### Daily account snapshot
- spend
- impressions
- clicks
- ctr
- cpc
- conversions / purchase_roas if available

### Campaign audit
- list campaigns
- pull 30-day insights per campaign
- identify spend concentration, underperformers, paused waste, and winners

### Breakdown report
- age/gender breakdowns
- compare CTR/CPC/ROAS by segment

### Change-safe launch summary
After creation, always report:
- account used
- page used
- campaign/ad set/creative/ad IDs
- final statuses
- whether anything was activated

---

## References

- Official overview: https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/ads-cli-overview
- Command reference: https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/command-reference
- Insights docs: https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/insights
- PyPI package: https://pypi.org/project/meta-ads/
