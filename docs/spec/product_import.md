# Smart Import & Product Library

Upload PDF, Excel and CSV price lists, extract products with AI, review them and maintain the product library.

Documents are uploaded in Smart Import Studio and processed into products with status pending_review. Reviewers approve or reject products in the Product Library.

## Entities
```ts
interface ImportDocument {
  id: string;
  file: FileRef | null;
  file_name: string;
  file_type: 'pdf' | 'xlsx' | 'csv' | 'docx' | 'png' | 'jpg';
  status: 'uploaded' | 'processing' | 'needs_review' | 'completed' | 'failed' | 'partially_parsed';
  products_extracted: number;
  error_message: string | null;
  uploaded_by: string;
  created_at: string;
  extraction_status: 'not_started' | 'in_progress' | 'completed' | 'failed';
  approved_records: number;
  rejected_records: number;
  source: 'upload' | 'google_drive';
  document_purpose: 'product_catalogue' | 'price_list' | 'promotional_flyer' | 'brand_guideline' | 'marketing_brief' | 'unknown' | null;
  drive_file_id: string | null;
  drive_file_link: string | null;
  worksheet_names: string[] | null;
  extraction_log: string | null;
  warnings: string | null;
  findings: string | null;
  interpretation_issue: string | null;
}

interface Product {
  id: string;
  name: string;
  sku: string | null;
  description: string | null;
  category: string | null;
  price: number | null;
  currency: string;
  unit: string | null;
  status: 'pending_review' | 'approved' | 'rejected';
  extraction_confidence: number | null;
  import_document_id: string | null;
  reviewed_by: string | null;
  created_at: string;
  updated_at: string;
  barcode: string | null;
  brand: string | null;
  image_url: string | null;
  regular_price: number | null;
  promotional_price: number | null;
  is_active: boolean;
  category_id: string | null;
}

interface PendingProduct {
  id: string;
  name: string;
  sku: string | null;
  barcode: string | null;
  brand: string | null;
  category: string | null;
  description: string | null;
  regular_price: number | null;
  promotional_price: number | null;
  currency: string;
  unit: string | null;
  image_url: string | null;
  status: 'pending_review' | 'approved' | 'rejected';
  extraction_confidence: number | null;
  import_document_id: string | null;
  approved_product_id: string | null;
  reviewed_by: string | null;
  created_at: string;
  updated_at: string;
  validation_status: 'valid' | 'missing_information' | 'duplicate' | 'invalid';
  duplicate_product_id: string | null;
  validation_notes: string | null;
}

interface ExtractedField {
  id: string;
  import_document_id: string;
  pending_product_id: string | null;
  record_type: 'product' | 'brand' | 'campaign';
  record_label: string | null;
  field_name: string;
  extracted_value: string | null;
  corrected_value: string | null;
  source_location: string | null;
  confidence: number | null;
  confidence_level: 'high' | 'medium' | 'low' | null;
  validation_status: 'valid' | 'warning' | 'invalid' | 'missing' | 'corrected';
  corrected_by: string | null;
  corrected_at: string | null;
  created_at: string;
  normalized_value: string | null;
}

interface ProductChangeLog {
  id: string;
  product_id: string;
  change_type: 'created' | 'edited' | 'price_change' | 'activation_change' | 'imported' | 'image_change';
  field_name: string | null;
  old_value: string | null;
  new_value: string | null;
  import_document_id: string | null;
  changed_by: string;
  created_at: string;
}

interface ImageMatch {
  id: string;
  asset_id: string;
  product_id: string | null;
  match_method: 'sku' | 'barcode' | 'filename' | 'product_name' | 'manual';
  confidence: number;
  status: 'confirmed' | 'needs_review' | 'rejected';
  candidate_product_ids: string[] | null;
  reviewed_by: string | null;
  created_at: string;
}

interface FieldMappingRule {
  id: string;
  source_header: string;
  canonical_field: string;
  confirmed_count: number;
  created_by: string;
  created_at: string;
  updated_at: string;
}

interface Category {
  id: string;
  name: string;
  normalized_name: string;
  created_at: string;
}

interface ReviewAuditLog {
  id: string;
  pending_product_id: string | null;
  import_document_id: string | null;
  action: string;
  field_name: string | null;
  old_value: string | null;
  new_value: string | null;
  actor_id: string;
  created_at: string;
}
```

**Table names:** `import _document` (Import Document), `product` (Product), `pending _product` (Pending Product), `extracted _field` (Extracted Field), `product _change _log` (Product Change Log), `image _match` (Image Match), `field _mapping _rule` (Field Mapping Rule), `category` (Category), `review _audit _log` (Review Audit Log)

## APIs
**Files with call_api / call_streaming_api** (also from run_typescript):
- A `FileRef` in a result — at any depth, entities and lists included — arrives as a file object `{ fileRef, filePath: "filerefs/<id>", filename, mimeType, size }` (filename, mimeType and size when its metadata could be read; `missing: true` when this workspace has no such file). Read, analyze or copy the file at `filePath`; to show it to the user, `copy_file` it into `artifacts/` and `present_files` that.
- A `FileRef` parameter takes `{ filePath: "<any readable VFS path, e.g. artifacts/report.pdf>" }` (uploaded for the call; optional `filename` / `mimeType`), a file object from a previous result as-is, or an existing FileRef id. A plain path string is not a FileRef.
- `run_sql` rows hold the bare FileRef id; the file is at `filerefs/<id>`.

```ts
registerImportDocument(request: { file: FileRef; file_name: string; file_type: 'pdf' | 'xlsx' | 'csv' | 'docx' | 'png' | 'jpg' }): { id: string; file: FileRef | null; file_name: string; file_type: 'pdf' | 'xlsx' | 'csv' | 'docx' | 'png' | 'jpg'; status: 'uploaded' | 'processing' | 'needs_review' | 'completed' | 'failed' | 'partially_parsed'; products_extracted: number; error_message: string | null; uploaded_by: string; created_at: string; extraction_status: 'not_started' | 'in_progress' | 'completed' | 'failed'; approved_records: number; rejected_records: number; source: 'upload' | 'google_drive'; document_purpose: 'product_catalogue' | 'price_list' | 'promotional_flyer' | 'brand_guideline' | 'marketing_brief' | 'unknown' | null; drive_file_id: string | null; drive_file_link: string | null; worksheet_names: string[] | null; extraction_log: string | null; warnings: string | null; findings: string | null; interpretation_issue: string | null }

listImportDocuments(request: void): { items: ImportDocument[] }

processImportDocument(request: { document_id: string }): { status: string; productsExtracted: number; documentPurpose: string; warnings: string[] }

listProducts(request: { status?: 'pending_review' | 'approved' | 'rejected'; search?: string; import_document_id?: string; category?: string; brand?: string; sku?: string; is_active?: boolean }): { items: Product[] }

saveProduct(request: { id?: string; name: string; sku?: string | null; barcode?: string | null; brand?: string | null; description?: string | null; category?: string | null; regular_price?: number | null; promotional_price?: number | null; currency: string; unit?: string | null; image_url?: string | null; is_active?: boolean }): { id: string; name: string; sku: string | null; description: string | null; category: string | null; price: number | null; currency: string; unit: string | null; status: 'pending_review' | 'approved' | 'rejected'; extraction_confidence: number | null; import_document_id: string | null; reviewed_by: string | null; created_at: string; updated_at: string; barcode: string | null; brand: string | null; image_url: string | null; regular_price: number | null; promotional_price: number | null; is_active: boolean; category_id: string | null }

reviewProduct(request: { pending_product_id: string; decision: 'approved' | 'rejected' }): { id: string; name: string; sku: string | null; barcode: string | null; brand: string | null; category: string | null; description: string | null; regular_price: number | null; promotional_price: number | null; currency: string; unit: string | null; image_url: string | null; status: 'pending_review' | 'approved' | 'rejected'; extraction_confidence: number | null; import_document_id: string | null; approved_product_id: string | null; reviewed_by: string | null; created_at: string; updated_at: string; validation_status: 'valid' | 'missing_information' | 'duplicate' | 'invalid'; duplicate_product_id: string | null; validation_notes: string | null; product?: Product | null }

listPendingProducts(request: { status?: 'pending_review' | 'approved' | 'rejected'; import_document_id?: string; filter?: 'all_records' | 'high_confidence' | 'needs_review' | 'missing_information' | 'duplicate' | 'rejected'; search?: string }): { items: PendingProduct[]; counts: { all_records: number; high_confidence: number; needs_review: number; missing_information: number; duplicate: number; rejected: number } }

getImportDocument(request: { document_id: string }): { document: ImportDocument; pending_products: PendingProduct[]; extracted_fields: ExtractedField[] }

// Corrects an extracted value, keeping the original.
updateExtractedField(request: { field_id: string; corrected_value?: string | null }): { id: string; import_document_id: string; pending_product_id: string | null; record_type: 'product' | 'brand' | 'campaign'; record_label: string | null; field_name: string; extracted_value: string | null; corrected_value: string | null; source_location: string | null; confidence: number | null; confidence_level: 'high' | 'medium' | 'low' | null; validation_status: 'valid' | 'warning' | 'invalid' | 'missing' | 'corrected'; corrected_by: string | null; corrected_at: string | null; created_at: string; normalized_value: string | null }

listExtractedFields(request: { import_document_id?: string; pending_product_id?: string; record_type?: 'product' | 'brand' | 'campaign' }): { items: ExtractedField[] }

// Edits individual cells of a pending product and refreshes its validation status.
updatePendingProduct(request: { pending_product_id: string; name?: string; sku?: string | null; barcode?: string | null; category?: string | null; regular_price?: number | null; promotional_price?: number | null; image_url?: string | null }): { id: string; name: string; sku: string | null; barcode: string | null; brand: string | null; category: string | null; description: string | null; regular_price: number | null; promotional_price: number | null; currency: string; unit: string | null; image_url: string | null; status: 'pending_review' | 'approved' | 'rejected'; extraction_confidence: number | null; import_document_id: string | null; approved_product_id: string | null; reviewed_by: string | null; created_at: string; updated_at: string; validation_status: 'valid' | 'missing_information' | 'duplicate' | 'invalid'; duplicate_product_id: string | null; validation_notes: string | null }

// Moves a value that landed in the wrong field to the correct field.
remapPendingProductField(request: { pending_product_id: string; from_field: 'name' | 'sku' | 'barcode' | 'brand' | 'category' | 'description' | 'regular_price' | 'promotional_price' | 'image_url'; to_field: 'name' | 'sku' | 'barcode' | 'brand' | 'category' | 'description' | 'regular_price' | 'promotional_price' | 'image_url' }): { id: string; name: string; sku: string | null; barcode: string | null; brand: string | null; category: string | null; description: string | null; regular_price: number | null; promotional_price: number | null; currency: string; unit: string | null; image_url: string | null; status: 'pending_review' | 'approved' | 'rejected'; extraction_confidence: number | null; import_document_id: string | null; approved_product_id: string | null; reviewed_by: string | null; created_at: string; updated_at: string; validation_status: 'valid' | 'missing_information' | 'duplicate' | 'invalid'; duplicate_product_id: string | null; validation_notes: string | null }

// Removes an incorrect pending record and its extracted fields.
deletePendingProduct(request: { pending_product_id: string }): { success: boolean }

// Restores a rejected record to pending review.
restorePendingProduct(request: { pending_product_id: string }): { id: string; name: string; sku: string | null; barcode: string | null; brand: string | null; category: string | null; description: string | null; regular_price: number | null; promotional_price: number | null; currency: string; unit: string | null; image_url: string | null; status: 'pending_review' | 'approved' | 'rejected'; extraction_confidence: number | null; import_document_id: string | null; approved_product_id: string | null; reviewed_by: string | null; created_at: string; updated_at: string; validation_status: 'valid' | 'missing_information' | 'duplicate' | 'invalid'; duplicate_product_id: string | null; validation_notes: string | null }

// Approves or rejects selected records, or approves all valid records.
bulkReviewProducts(request: { pending_product_ids?: string[]; all_valid?: boolean; decision: 'approved' | 'rejected' }): { processed: number; skipped: { /** Format: UUID (e.g., '550e8400-e29b-41d4-a716-446655440000') */ pending_product_id: string; reason: string }[] }

// Side-by-side comparison of a pending record with the existing product it matches.
comparePendingProduct(request: { pending_product_id: string }): { pending: PendingProduct; existing?: Product | null; differing_fields: string[]; possible_name_matches: Product[] }

// Resolves an approval that conflicts with an existing product after the user saw the comparison.
resolveDuplicateApproval(request: { pending_product_id: string; action: 'update_existing' | 'keep_existing'; fields_to_apply?: string[]; confirmed: boolean }): { id: string; name: string; sku: string | null; barcode: string | null; brand: string | null; category: string | null; description: string | null; regular_price: number | null; promotional_price: number | null; currency: string; unit: string | null; image_url: string | null; status: 'pending_review' | 'approved' | 'rejected'; extraction_confidence: number | null; import_document_id: string | null; approved_product_id: string | null; reviewed_by: string | null; created_at: string; updated_at: string; validation_status: 'valid' | 'missing_information' | 'duplicate' | 'invalid'; duplicate_product_id: string | null; validation_notes: string | null; product?: Product | null }

listProductFilterOptions(request: void): { categories: string[]; brands: string[] }

getProductDetails(request: { product_id: string }): { product: Product; change_log: ProductChangeLog[]; source_documents: ImportDocument[]; linked_images: Asset[]; matches: ImageMatch[] }

bulkUpdatePrices(request: { product_ids: string[]; price_field: 'regular_price' | 'promotional_price'; mode: 'set' | 'percent_change' | 'fixed_change'; value: number; preview_only?: boolean }): { changes: { /** Format: UUID (e.g., '550e8400-e29b-41d4-a716-446655440000') */ product_id: string; name: string; old_price?: number | null; new_price?: number | null; error?: string | null }[]; applied: number }

setProductsActive(request: { product_ids: string[]; is_active: boolean }): { updated: number }

runImageMatching(request: { asset_ids?: string[] }): { confirmed: number; needs_review: number; unmatched: number }

listImageMatches(request: { status?: 'confirmed' | 'needs_review' | 'rejected' }): { items: { match: ImageMatch; asset: Asset; product?: Product | null }[] }

reviewImageMatch(request: { image_match_id: string; decision: 'confirm' | 'reject'; product_id?: string }): { id: string; asset_id: string; product_id: string | null; match_method: 'sku' | 'barcode' | 'filename' | 'product_name' | 'manual'; confidence: number; status: 'confirmed' | 'needs_review' | 'rejected'; candidate_product_ids: string[] | null; reviewed_by: string | null; created_at: string }

linkProductImage(request: { product_id: string; asset_id: string; action: 'link' | 'unlink' }): { id: string; name: string; sku: string | null; description: string | null; category: string | null; price: number | null; currency: string; unit: string | null; status: 'pending_review' | 'approved' | 'rejected'; extraction_confidence: number | null; import_document_id: string | null; reviewed_by: string | null; created_at: string; updated_at: string; barcode: string | null; brand: string | null; image_url: string | null; regular_price: number | null; promotional_price: number | null; is_active: boolean; category_id: string | null }

bulkUpdatePendingProducts(request: { pending_product_ids: string[]; brand?: string | null; category?: string | null; currency?: string | null; unit?: string | null; image_url?: string | null }): { updated: number; skipped: { /** Format: UUID (e.g., '550e8400-e29b-41d4-a716-446655440000') */ pending_product_id: string; reason: string }[] }

reimportImportDocument(request: { document_id: string }): { status: string; productsExtracted: number }

listReviewAudit(request: { pending_product_id?: string; import_document_id?: string }): { items: ReviewAuditLog[] }

listCategories(request: void): { items: { category: Category; product_count: number }[] }

saveCategory(request: { id?: string; name: string; merge_into_id?: string }): { id: string; name: string; normalized_name: string; created_at: string }

// AI suggests categories for pending records; suggestions only.
suggestCategory(request: { pending_product_ids: string[] }): { suggestions: { /** Format: UUID (e.g., '550e8400-e29b-41d4-a716-446655440000') */ pending_product_id: string; category: string; is_new: boolean; confidence: number }[] }

// AI explains why a pending record was flagged.
explainAnomaly(request: { pending_product_id: string }): { explanation: string; suggested_checks: string[] }

// AI extracts promotional information from an imported document.
extractPromotions(request: { document_id: string }): { findings: { label: string; value: string; confidence: number }[] }

// Generates a product image with AI, stores it as an asset and links it to the product.
generateProductImage(request: { product_id: string }): { asset?: Asset | null; image_url?: string | null; message: string }
```

## Pages
- **Smart Import Studio** `/import/studio` - Upload or pick price lists and documents and extract products.
- **Product Library** `/products/library` - Browse, search, edit and manage approved products and match images.
- **Extraction Review** `/import/review` - Editable review table for all pending imported records.

## Features
- **Smart document import**
- **Product library and review**
- **Field-level extraction review**
- **Extraction review table**
- **Product library management**
- **Product image matching**