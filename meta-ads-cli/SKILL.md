---
name: meta-ads-cli
description: Operate Meta Ads through Meta's official Ads CLI (`meta-ads`) for campaign management, asset discovery, insights extraction, and analytics workflows.
version: 1.1.0
author: Hermes Agent
license: Internal
platforms: [linux, macos]
metadata:
  hermes:
    tags: [meta, ads, marketing, cli, analytics, automation]
    homepage: https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/ads-cli-overview
---

# Meta Ads CLI

Use this skill when the user wants to manage Meta/Facebook ads from the terminal: discover ad accounts, create campaigns/ad sets/ads/creatives, work with datasets/catalogs, or pull insights and analytics.

This skill is built around Meta's official Ads CLI, published on Apr 29, 2026 and exposed as the `meta` command via the `meta-ads` Python package.

The trick is not just knowing commands — it is making the CLI actually usable for real people who may have:
- no `pip`
- blocked global pip installs
- the wrong Python version
- missing `uv`
- a token that exists but lacks the right scopes/assets
- ad account context that is only half-configured

This skill is opinionated about setup and safety so you do not waste time on dumb environment failures.

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
- Docs and package examples do not perfectly match each other.
- No public official GitHub repo is currently known.
- If a command appears hinted in package metadata but absent from docs (`product-set`, `product-item`, `product-feed`), treat it as underdocumented and verify with `meta --help` before using it.

---

## The recommended install path

For most users, the best path is **isolated install in a virtualenv**.

### Recommended (works around blocked global pip)

```bash
python3 --version
python3 -m venv .venv
. .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install meta-ads
meta --version
meta --help
```

Why this path:
- avoids global package policy issues
- avoids distro-managed Python conflicts
- keeps the CLI easy to remove or upgrade
- makes `meta` reliably appear on PATH while the venv is active

### If the docs' `uv` flow is desired

Meta's docs mention `uv sync` and `uv run meta`, but they do not explain the flow clearly. The practical takeaway:

```bash
python3 -m venv .venv
. .venv/bin/activate
python -m pip install meta-ads
uv run meta --help
```

If `uv` is installed, `uv run meta` can be a useful fallback when the environment is not activated or PATH is weird.

### If `pip` is missing

Try:

```bash
python3 -m ensurepip --upgrade
python3 -m pip --version
```

If that fails, the machine likely needs OS packages such as `python3-pip` and possibly `python3-venv` installed by the user/admin.

### If global pip is blocked / externally managed

Do **not** fight the system Python. Use a venv:

```bash
python3 -m venv .venv
. .venv/bin/activate
python -m pip install meta-ads
```

### If `meta` installs but command is not found

Usually one of these is true:
- the virtualenv is not activated
- the install went into a user-site bin dir not on PATH
- the shell needs a fresh session

Best fix: activate the venv and retry.

```bash
. .venv/bin/activate
meta --version
```

### If Python is too old

Meta docs require **Python 3.12+**. Check with:

```bash
python3 --version
```

If the machine is below 3.12, do not keep debugging package errors — upgrade Python first.

---

## Quick verification checklist

After install, verify in this order:

```bash
meta --version
meta --help
meta auth status
meta --output json ads adaccount list
meta --output json ads page list
```

If `meta auth status` works but asset reads fail, it is usually a permissions/asset assignment problem, not an install problem.

---

## Required configuration

Expected environment variables:
- `ACCESS_TOKEN` — required
- `AD_ACCOUNT_ID` — required for most ad operations
- `BUSINESS_ID` — useful for datasets/catalogs/business-scoped work; avoids surprise prompts in automation

Expected auth model:
- Meta **system user access token** with the right assets attached

Commonly required scopes from Meta docs:
- `business_management`
- `ads_management`
- `pages_show_list`
- `pages_read_engagement`
- `pages_manage_ads`
- `catalog_management`
- `read_insights`

Strong recommendation: store config in a local project `.env` or export env vars in the shell session, rather than sprinkling flags everywhere.

Example:

```bash
export ACCESS_TOKEN=<ACCESS_TOKEN>
export AD_ACCOUNT_ID=<AD_ACCOUNT_ID>
export BUSINESS_ID=<BUSINESS_ID>
```

Or in `.env`:

```bash
ACCESS_TOKEN=<ACCESS_TOKEN>
AD_ACCOUNT_ID=<AD_ACCOUNT_ID>
BUSINESS_ID=<BUSINESS_ID>
```

---

## The auth and asset setup most people get wrong

Meta's docs make this clear, but people still trip over it.

The token is not enough by itself. The **system user must also be configured correctly**:

1. Create a **System User** in Meta Business Suite
2. Set role to **Admin**
3. Assign the system user to the required assets:
   - ad accounts
   - Business Pages
   - datasets / Pixels
   - product catalogs
4. Add the system user as **App Admin** in Meta for Developers > App Settings > Roles
5. Generate a token with the required scopes

Common symptoms:
- `meta auth status` succeeds but `page list` fails → missing page access or page scopes
- campaign commands work but insights fail → missing `read_insights`
- conversion workflows fail → dataset/pixel not assigned or not connected
- catalog commands fail → missing `catalog_management` or missing catalog assignment

---

## Known-good command form

Prefer global flags **before** subcommands:

```bash
meta --output json ads campaign list
```

Meta's docs also say the global options belong before the subcommand. Stick to that form.

---

## Output modes

Meta docs define 3 output modes:

### `table` (default)
Human-readable. Good for quick inspection.

### `json`
Best for automation.

```bash
meta --output json ads campaign list
```

### `plain`
Tab-separated, one record per line. Good for shell tools.

```bash
meta --output plain ads campaign list
```

Examples:

```bash
meta --output json ads campaign list | jq '.[].name'
meta --output plain ads campaign list | sort -t$'\t' -k5 -rn
```

If `jq` is unavailable, do not pretend it exists. Fall back to JSON parsing in Python or inspect the raw JSON.

---

## Non-interactive and automation flags

These matter a lot for scripts and agents.

- `--no-input` — disable interactive prompts
- `--force` — skip confirmation on destructive actions
- `--debug` — verbose troubleshooting

Example:

```bash
meta --no-input ads campaign delete <CAMPAIGN_ID> --force
```

### Important automation caveat
Some flows can unexpectedly become interactive if required context is missing.

Most notably:
- missing `BUSINESS_ID` on dataset/catalog flows
- business tools terms acceptance required by Meta

If the workflow must be scriptable, set `BUSINESS_ID` explicitly and expect that terms acceptance may still need a human business admin.

---

## Config precedence

From Meta's configuration docs:

1. command-line flags
2. environment variables
3. project-level `.env`
4. user-level config `~/.config/meta/`

Also useful:
- shell env vars override `.env`
- `XDG_CONFIG_HOME` can affect the user config location

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
- `BUSINESS_ID` is set if dataset/catalog automation is involved

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

Instagram actor example:

```bash
meta --output json ads creative create \
  --name "Insta Ad" \
  --page-id <PAGE_ID> \
  --instagram-actor-id <INSTAGRAM_ACCOUNT_ID> \
  --image ./banner.jpg \
  --body "Shop our collection" \
  --link-url https://example.com
```

Dynamic creative optimization example:

```bash
meta --output json ads creative create \
  --name "DCO Test" \
  --page-id <PAGE_ID> \
  --link-url https://example.com \
  --images ./img1.jpg --images ./img2.jpg --images ./img3.jpg \
  --titles "Shop Now" --titles "Learn More" \
  --bodies "50% off everything!" --bodies "Free shipping today!" \
  --descriptions "Limited time offer" --descriptions "While supplies last" \
  --call-to-actions SHOP_NOW --call-to-actions LEARN_MORE
```

DCO limits from docs:
- images/videos: max 10
- titles/bodies/descriptions/CTAs: max 5

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

Custom range:

```bash
meta --output json ads insights get --since 2026-04-01 --until 2026-04-29
```

Important rule: `--since` and `--until` must be used together and override `--date-preset`.

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

Time increment examples:

```bash
meta --output json ads insights get --campaign-id <CAMPAIGN_ID> --time-increment daily --fields spend
meta --output json ads insights get --campaign-id <CAMPAIGN_ID> --time-increment weekly
```

Scope-specific examples:

```bash
meta --output json ads insights get --campaign-id <CAMPAIGN_ID>
meta --output json ads insights get --adset-id <AD_SET_ID>
meta --output json ads insights get --ad-id <AD_ID>
```

Common date presets from docs:
- `today`
- `yesterday`
- `last_3d`
- `last_7d`
- `last_14d`
- `last_30d`
- `last_90d`
- `this_month`
- `last_month`

Common time increments:
- `all_days`
- `daily`
- `weekly`
- `monthly`

### 7) Dataset / catalog workflows

Inspect first:

```bash
meta --output json ads dataset list
meta --output json ads catalog list
```

Create dataset:

```bash
meta --output json ads dataset create --name "My Website Pixel"
```

Connect dataset:

```bash
meta --output json ads dataset connect <PIXEL_ID> --ad-account-id <AD_ACCOUNT_ID>
meta --output json ads dataset connect <PIXEL_ID> --catalog-id <CATALOG_ID>
meta --output json ads dataset connect <PIXEL_ID> --ad-account-id <AD_ACCOUNT_ID> --catalog-id <CATALOG_ID>
```

Assign user tasks:

```bash
meta --output json ads dataset assign-user <PIXEL_ID> --tasks ADVERTISE --tasks ANALYZE --tasks EDIT
```

Catalog examples:

```bash
meta --output json ads catalog create --name "Hotel Inventory" --vertical hotels
meta --output json ads catalog update <CATALOG_ID> --name "Renamed Catalog"
```

### 8) Scriptable reporting loop

Meta's docs show the CLI being used in JSON loops.

```bash
CAMPAIGN_IDS=$(meta --output json ads campaign list | jq -r '.[].id')

for id in $CAMPAIGN_IDS; do
  echo "Processing ad campaign $id"
  meta --output json ads insights get --campaign-id "$id" --fields conversions,purchase_roas
done
```

If `jq` is missing, use Python instead of stalling out.

---

## Recommended agent workflow

### For management tasks
1. Verify install works (`meta --version`)
2. Verify auth/context
3. List relevant assets
4. Ask/confirm the target account/page/campaign if ambiguous
5. Create or update resources in the correct order
6. Capture all returned IDs
7. Leave paused unless activation is explicitly requested
8. Summarize exactly what changed

### For analytics tasks
1. Verify install works
2. Verify auth/context
3. Determine reporting scope: account / campaign / ad set / ad
4. Determine date window, fields, breakdowns, and increment
5. Use `--output json`
6. Parse and compute derived metrics if needed
7. Return a concise narrative plus structured numbers

### For automation/CI usage
1. Ensure `ACCESS_TOKEN`, `AD_ACCOUNT_ID`, and `BUSINESS_ID` are pre-set
2. Use `meta --output json ...`
3. Use `--no-input` where prompts would block
4. Check exit codes explicitly
5. Avoid destructive actions unless intentional and confirmed

---

## Common pitfalls

### Environment/setup
- `pip` missing
- Python below 3.12
- global pip blocked by system policy
- virtualenv not activated
- `meta` command not on PATH
- `uv` absent even though docs mention it

### Auth and permissions
- wrong token type (not a system user token)
- missing App Admin linkage in Meta for Developers
- missing asset assignment for page/ad account/pixel/catalog
- missing `read_insights` / page scopes / catalog scopes

### Workflow/data issues
- wrong ad account selected
- page exists but system user lacks access
- pixel/dataset not connected when creating conversion workflows
- media file path wrong or missing
- trying to update immutable creative fields
- using destructive commands without realizing they cascade to children
- assuming docs cover every subcommand in this alpha release
- business tools terms block dataset creation
- dataset/catalog automation hangs because `BUSINESS_ID` was not set

---

## Troubleshooting

### `python3 -m pip` fails immediately
Likely causes:
- pip missing
- externally managed system Python
- missing `venv`

Best fix path:
1. use `python3 -m ensurepip --upgrade` if available
2. otherwise install OS Python tooling (`python3-pip`, `python3-venv`)
3. install inside a venv, not globally

### `meta` is not found after install
Usually a shell/venv issue. Fix by activating the venv:

```bash
. .venv/bin/activate
meta --version
```

If that still fails, confirm the package actually installed:

```bash
python -m pip show meta-ads
```

### Auth seems valid but commands fail
Usually an asset-permission problem, not a token-format problem. Check:
- system user role
- ad account assignment
- page assignment
- dataset/catalog assignment
- app admin linkage
- token scopes

### `--output json` behaves oddly
Move global flags earlier:

```bash
meta --output json ads campaign list
```

### Insights are failing or empty
Check:
- token has `read_insights`
- correct scope object (`campaign-id`, `adset-id`, `ad-id`)
- date args are valid
- `--since` and `--until` were supplied together

### Conversion workflow errors
Check that the dataset/pixel exists, is assigned, and is attached to the relevant business/account context.

### Dataset or catalog command hangs or prompts unexpectedly
Set `BUSINESS_ID` explicitly. Docs say the CLI may try to derive it from the ad account, but if that fails it can prompt.

### Creative updates fail
Some creative properties may be effectively immutable. Recreate the creative instead of trying to patch it.

### Delete fails
Possible reasons from docs:
- creative still used by active ads
- catalog still has active product feeds or ads referencing it

---

## Exit codes

Meta docs provide these, and automation should use them:

- `0` success
- `1` general error
- `2` usage / argument error
- `3` authentication error
- `4` API error
- `5` resource not found

Example:

```bash
meta ads campaign list
if [ $? -eq 3 ]; then
  echo "Not authenticated -- add token to .env"
fi
```

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
- platform breakdowns
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
- Get started: https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/setup/get-started
- Configuration: https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/setup/configuration
- Command reference: https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/command-reference
- Tutorials and recipes: https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/tutorials-and-recipes
- Insights docs: https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/insights
- Datasets and catalogs: https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/datasets-and-catalogs
- Ad creatives: https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/ad-creatives
- PyPI package: https://pypi.org/project/meta-ads/

Also see: [official-docs-and-recipes.md](./references/official-docs-and-recipes.md)
