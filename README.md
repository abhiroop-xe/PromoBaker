# PromoBaker

AI-powered retail marketing operations. PromoBaker turns business documents (PDF, Excel, CSV price lists) into a structured, reviewed product library, then uses that library to build and export promotional campaigns.

Built on the Luo platform as a standalone app: no external integrations are required.

## Quick start

1. Create a Luo workspace and open the assistant chat.
2. Paste the prompt from [LUO_TEMPLATE.md](LUO_TEMPLATE.md) and review the proposed spec.
3. Approve the build.
4. Create a brand kit (PNG/JPG logo) and a template, then verify the template.
5. Import your own price list, or use [sample-data/products.csv](sample-data/products.csv) in Smart Import Studio. The sample prices and units are invented placeholders, not real data.
6. Build a campaign, generate posters, approve and export.

## Workflow

1. **Smart Import Studio** - upload a price list. Deterministic parsing runs first; AI helps with ambiguous headers and PDF/image content.
2. **Extraction Review** - every extracted record is staged for human review. Nothing is auto-approved and missing values stay blank.
3. **Product Library** - approved products with prices, SKU, brand, category and images.
4. **Brand Kits, Assets & Templates** - logo (PNG/JPG), colours, fonts and layout templates with placeholders such as {{product_name}}, {{product_image}}, {{regular_price}}, {{promotional_price}}, {{brand_logo}}.
5. **Campaign Builder & Editor** - pick approved products, a brand kit and a template; generate posters, review and approve.
6. **Export Centre** - PDF, PNG, JPG, per-page PDF or ZIP exports of approved campaigns.

## Pages

Landing, Dashboard, Smart Import Studio, Extraction Review, Product Library, Asset Library, Brand Kits, Template Library, Campaign Builder, Campaign Editor, Export Centre, Settings.

## Principles

- Never silently overwrite products; never auto-approve; never invent product data or prices.
- Reviewer/admin roles guard approvals and bulk actions.
- AI only fills staging records or returns suggestions, and every AI request is logged.

## Known limitations

- The poster renderer draws PNG/JPEG only (SVG logos are not supported).
- Poster generation runs in batches of 10 per call; campaigns need a complete poster set before submit or export.
- Word (.docx) files cannot be read.
- Currency symbol on posters comes from the template configuration.

See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) for the data model and backend APIs.

## Use with Luo

This project is a Luo workspace spec, not a standalone code app. To recreate it, open [LUO_TEMPLATE.md](LUO_TEMPLATE.md), paste the prompt into a new Luo workspace chat and review the proposed spec before building. Full reference: `docs/spec/`.

## Status

This repository contains the Luo template and documentation. The application itself runs in a Luo workspace.
