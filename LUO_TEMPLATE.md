# PromoBaker - Luo template

Paste the prompt below into a new Luo workspace chat to recreate PromoBaker. Luo generates the entities, backend APIs and pages from it. For the full reference of what the original contains, see the files in `docs/spec/`.

Review the proposed spec before building. Nothing here contains customer data, brand assets or credentials.

---

## Prompt

Build PromoBaker, an AI-powered retail marketing operations app. It turns business documents (PDF, Excel, CSV price lists) into a structured product library and uses that library to build promotional campaigns. Design: professional enterprise SaaS, minimal, desktop-first. Use native Luo entities and backend APIs only. No external integrations; in-app notifications only.

Hard rules:
- Never silently overwrite a product. Never auto-approve extracted data. Never invent product information; missing values stay blank.
- Only reviewers/admins may approve, delete or run bulk actions (enforced in the backend).
- All AI calls run server-side with a 60 s timeout and 2 retries, return schema-validated JSON, and are logged (ai_usage_log).

### Feature set 1 - Overview
- Entities: activity_log, workspace_setting, ai_usage_log.
- Page **Dashboard**: counts (products, pending reviews, campaigns, exports), recent activity.
- Page **Settings**: workspace settings (default currency, AI usage view).

### Feature set 2 - Product import and library
- Entities: import_document, product, pending_product, extracted_field, product_change_log, image_match, field_mapping_rule, category, review_audit_log.
- Page **Smart Import Studio**: upload PDF/Excel/CSV. Parse deterministically first; use AI only for ambiguous headers, PDFs and images. Detect duplicates.
- Page **Extraction Review**: review staged products field by field, edit, approve, reject, resolve duplicates, bulk actions with audit log.
- Page **Product Library**: approved products (name, SKU, brand, unit, category, regular price, promotional price, currency, image), filters, bulk price update, activate/deactivate, image matching and AI illustration generation (labelled as illustrations, not photos).

### Feature set 3 - Creative libraries
- Entities: asset, brand_kit, template.
- Page **Asset Library**: upload and manage images (PNG/JPG).
- Page **Brand Kits**: name, logo (PNG/JPG only), colours, fonts; AI suggestions.
- Page **Template Library**: poster layouts with layers and placeholders {{product_image}}, {{product_name}}, {{promotional_price}}, {{regular_price}} ("Was ..."), {{brand_logo}}. Template configuration holds currency_symbol and currency_code. A template must be verified exportable before use.

### Feature set 4 - Campaigns and export
- Entities: campaign, campaign_product, campaign_poster, campaign_event, export_job.
- Page **Campaign Builder**: choose approved products, a brand kit and a template; snapshot product data into campaign_product; generate posters in batches of 10 per call (return created, failed, remaining) with a "Generate posters" loop.
- Page **Campaign Editor**: edit, duplicate and review posters; workflow draft -> in_review -> approved -> exported. Submit and approve are blocked until every product has a good poster. Archive and delete campaigns.
- Page **Export Centre**: PDF, PNG, JPG, per-page PDF and ZIP exports of approved campaigns. Refuse partial poster sets and unverified templates. Export can take over a minute for 12 pages, so show progress and do not allow retry while processing.
- Page **Landing**: static welcome page describing the workflow.

Posters take their currency symbol from the template configuration, not from the product.

---

## After building

1. Create a brand kit with a PNG/JPG logo, and a template, then verify the template.
2. Upload a price list in Smart Import Studio and approve the extracted products.
3. Build a campaign, generate posters, approve it, and export.

## Known limitations

- The poster renderer draws PNG/JPEG only; SVG logos fail.
- Word (.docx) files cannot be read.
- Large exports are slow; wait for them rather than retrying.
