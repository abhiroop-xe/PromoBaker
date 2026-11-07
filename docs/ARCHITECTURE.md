# PromoBaker architecture

## Entities

**Overview:** activity_log, workspace_setting, ai_usage_log

**Product import & library:** import_document, product, pending_product, extracted_field, product_change_log, image_match, field_mapping_rule, category, review_audit_log

**Creative libraries:** asset, brand_kit, template

**Campaigns & export:** campaign, campaign_product, campaign_poster, campaign_event, export_job

## Backend APIs by feature set

- **Dashboard & Settings:** dashboard summary, recent activity, settings, AI usage report
- **Smart Import & Product Library:** register/process/re-import documents; extracted-field edits; pending-product review (single and bulk, duplicate resolution, audit); product list/save/filter; bulk price updates; activate/deactivate; categories; image matching and linking; AI product-image generation
- **Assets, Brand Kits & Templates:** asset create/list/delete; brand kit save/list/AI suggestions; template save/list/preview and export verification
- **Campaigns & Export:** campaign save/list/get; status workflow (submit, approve, send back, reopen); snapshot refresh; archive/delete; poster generation, listing, editing, duplication; export creation/listing/preview

## AI usage

Server-side only, 60 s timeout, up to 2 retries, schema-validated JSON output, each request logged. Operations: document classification, column mapping, product interpretation, category suggestion, anomaly explanation, promotion extraction, brand suggestions, image-match assistance, product image generation.

## Campaign lifecycle

draft -> in_review -> approved -> exported. Submit/approve require a complete, non-failed poster set; export refuses partial sets. Exports require an approved campaign and a verified template.
