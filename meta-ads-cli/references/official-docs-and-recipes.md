# Meta Ads CLI — Official Docs and Recipe Notes

Primary docs inspected:

- Overview: https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/ads-cli-overview
- Get started: https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/setup/get-started
- Configuration: https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/setup/configuration
- Command reference: https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/command-reference
- Tutorials and recipes: https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/tutorials-and-recipes
- Insights: https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/insights
- Datasets and catalogs: https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/datasets-and-catalogs
- Ad creatives: https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/ad-creatives
- PyPI package: https://pypi.org/project/meta-ads/

## Hard facts from the docs

### Requirements
- Python 3.12+
- virtual environment
- `pip` and `uv`
- system user access token
- ad account with assets

### Config precedence
1. command-line flags
2. environment variables
3. project `.env`
4. user config `~/.config/meta/`

### Required auth/scopes
- `business_management`
- `ads_management`
- `pages_show_list`
- `pages_read_engagement`
- `pages_manage_ads`
- `catalog_management`
- `read_insights`

### Output modes
- `table`
- `json`
- `plain`

### Exit codes
- `0` success
- `1` general error
- `2` usage / argument error
- `3` authentication error
- `4` API error
- `5` resource not found

## Useful command examples from docs

### JSON / shell workflows
```bash
meta --output json ads campaign list | jq '.[].name'
meta --output plain ads campaign list | sort -t$'\t' -k5 -rn
```

### Non-interactive delete
```bash
meta --no-input ads campaign delete <CAMPAIGN_ID> --force
```

### Insights examples
```bash
meta --output json ads insights get --date-preset yesterday
meta --output json ads insights get --campaign-id <CAMPAIGN_ID> --date-preset last_30d --fields spend,impressions,clicks,ctr,cpc,purchase_roas --sort spend_descending
meta --output json ads insights get --since 2026-04-01 --until 2026-04-29 --fields spend,impressions,clicks,ctr,cpc --breakdown age --breakdown gender
meta --output json ads insights get --campaign-id <CAMPAIGN_ID> --time-increment daily --fields spend
```

### Conversion tracking / dataset flow
```bash
meta --output json ads dataset create --name "My Website Pixel"
meta --output json ads dataset connect <PIXEL_ID> --ad-account-id <AD_ACCOUNT_ID>
meta --output json ads adset create <CAMPAIGN_ID> --name "Purchase Set" --optimization-goal OFFSITE_CONVERSIONS --billing-event IMPRESSIONS --pixel-id <PIXEL_ID> --custom-event-type PURCHASE --targeting-countries US
```

### Creative examples
```bash
meta --output json ads creative create --name "Summer Sale" --page-id <PAGE_ID> --image ./banner.jpg --body "50% off everything!" --title "Shop Now" --link-url https://example.com/sale --description "Limited time offer" --call-to-action SHOP_NOW

meta --output json ads creative create --name "DCO Test" --page-id <PAGE_ID> --link-url https://example.com --images ./img1.jpg --images ./img2.jpg --titles "Shop Now" --titles "Learn More" --bodies "50% off everything!" --bodies "Free shipping today!" --call-to-actions SHOP_NOW --call-to-actions LEARN_MORE
```

## Sharp edges worth remembering

- Docs oddly mention both `pip install meta-ads` and `uv sync` without clearly explaining when to use each.
- Global flags should go **before** subcommands.
- `--since` and `--until` must be used together and override `--date-preset`.
- Dataset/catalog flows may prompt if `BUSINESS_ID` is not set.
- Dataset creation may be blocked until a business admin accepts Meta business tools terms.
- Creative updates are not universally mutable; recreating is often the practical fallback.
- Some deletes fail when objects are still actively referenced (creatives in active ads, catalogs with active feeds/ads).
