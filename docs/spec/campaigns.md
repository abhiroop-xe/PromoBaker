# Campaigns & Export

Build promotional campaigns from approved products, a brand kit and a template, and export PDF flyers/posters and PNG/JPG images.

Campaign Builder organises campaigns and their products. Export Centre generates and lists exported files.

## Entities
```ts
interface Campaign {
  id: string;
  name: string;
  objective: string | null;
  status: 'draft' | 'active' | 'completed' | 'archived' | 'in_review' | 'approved' | 'exported';
  start_date: string | null;
  end_date: string | null;
  brand_kit_id: string | null;
  template_id: string | null;
  created_by: string;
  created_at: string;
  approved_by: string | null;
  approved_at: string | null;
  updated_at: string | null;
}

interface CampaignProduct {
  id: string;
  campaign_id: string;
  product_id: string;
  promo_price: number | null;
  promo_label: string | null;
  product_name: string | null;
  product_sku: string | null;
  product_brand: string | null;
  snapshot_regular_price: number | null;
  discount_percentage: number | null;
  image_ref: string | null;
  page_number: number | null;
  slot_index: number | null;
  snapshot_at: string | null;
}

interface ExportJob {
  id: string;
  campaign_id: string;
  format: 'pdf' | 'png' | 'jpg' | 'pdf_pages' | 'zip';
  status: 'completed' | 'failed' | 'processing';
  output: FileRef | null;
  error_message: string | null;
  requested_by: string;
  created_at: string;
  poster_ids: string[] | null;
  output_width_px: number | null;
  output_height_px: number | null;
  quality: number | null;
  processed_count: number | null;
  total_count: number | null;
}

interface CampaignEvent {
  id: string;
  campaign_id: string;
  event_type: string;
  note: string | null;
  actor_id: string;
  created_at: string;
}

interface CampaignPoster {
  id: string;
  campaign_id: string;
  campaign_product_id: string | null;
  template_id: string | null;
  page_number: number;
  design: string;
  status: 'pending' | 'rendered' | 'failed';
  render_error: string | null;
  preview: FileRef | null;
  is_customised: boolean;
  version: number;
  created_at: string;
  updated_at: string;
  updated_by: string | null;
}
```

**Table names:** `campaign` (Campaign), `campaign _product` (Campaign Product), `export _job` (Export Job), `campaign _event` (Campaign Event), `campaign _poster` (Campaign Poster)

## APIs
**Files with call_api / call_streaming_api** (also from run_typescript):
- A `FileRef` in a result — at any depth, entities and lists included — arrives as a file object `{ fileRef, filePath: "filerefs/<id>", filename, mimeType, size }` (filename, mimeType and size when its metadata could be read; `missing: true` when this workspace has no such file). Read, analyze or copy the file at `filePath`; to show it to the user, `copy_file` it into `artifacts/` and `present_files` that.
- A `FileRef` parameter takes `{ filePath: "<any readable VFS path, e.g. artifacts/report.pdf>" }` (uploaded for the call; optional `filename` / `mimeType`), a file object from a previous result as-is, or an existing FileRef id. A plain path string is not a FileRef.
- `run_sql` rows hold the bare FileRef id; the file is at `filerefs/<id>`.

```ts
saveCampaign(request: { id?: string; name: string; objective?: string | null; status: 'draft' | 'active' | 'completed' | 'archived' | 'in_review' | 'approved' | 'exported'; start_date?: string | null; end_date?: string | null; brand_kit_id?: string | null; template_id?: string | null; products?: { /** Format: UUID (e.g., '550e8400-e29b-41d4-a716-446655440000') */ product_id: string; promo_price?: number | null; promo_label?: string | null; page_number?: number | null; slot_index?: number | null }[] }): { id: string; name: string; objective: string | null; status: 'draft' | 'active' | 'completed' | 'archived' | 'in_review' | 'approved' | 'exported'; start_date: string | null; end_date: string | null; brand_kit_id: string | null; template_id: string | null; created_by: string; created_at: string; approved_by: string | null; approved_at: string | null; updated_at: string | null }

listCampaigns(request: { status?: 'draft' | 'active' | 'completed' | 'archived' | 'in_review' | 'approved' | 'exported' }): { items: Campaign[] }

getCampaign(request: { campaign_id: string }): { campaign: Campaign; items: { link: CampaignProduct; product: Product; product_changed_since_snapshot: boolean }[]; generated_files: ExportJob[]; events: CampaignEvent[] }

createExport(request: { campaign_id: string; format: 'pdf' | 'png' | 'jpg' | 'pdf_pages' | 'zip'; poster_ids?: string[]; output_width_px?: number; output_height_px?: number; quality?: number }): { id: string; campaign_id: string; format: 'pdf' | 'png' | 'jpg' | 'pdf_pages' | 'zip'; status: 'completed' | 'failed' | 'processing'; output: FileRef | null; error_message: string | null; requested_by: string; created_at: string; poster_ids: string[] | null; output_width_px: number | null; output_height_px: number | null; quality: number | null; processed_count: number | null; total_count: number | null }

listExports(request: { campaign_id?: string }): { items: ExportJob[] }

// Moves a campaign through draft, in review, approved.
setCampaignStatus(request: { campaign_id: string; action: 'submit' | 'approve' | 'send_back' | 'reopen'; note?: string | null }): { id: string; name: string; objective: string | null; status: 'draft' | 'active' | 'completed' | 'archived' | 'in_review' | 'approved' | 'exported'; start_date: string | null; end_date: string | null; brand_kit_id: string | null; template_id: string | null; created_by: string; created_at: string; approved_by: string | null; approved_at: string | null; updated_at: string | null }

// Manually refreshes a draft campaign product's snapshot from the live product.
refreshCampaignSnapshot(request: { campaign_id: string; product_ids?: string[] }): { refreshed: number }

// Creates individual poster design instances from the template and campaign snapshots.
generateCampaignPosters(request: { campaign_id: string; regenerate_ids?: string[] }): { created: number; failed: number; remaining: number }

listCampaignPosters(request: { campaign_id: string }): { items: CampaignPoster[] }

// Saves an edited poster design as a campaign-specific version.
savePoster(request: { poster_id: string; design: string }): { id: string; campaign_id: string; campaign_product_id: string | null; template_id: string | null; page_number: number; design: string; status: 'pending' | 'rendered' | 'failed'; render_error: string | null; preview: FileRef | null; is_customised: boolean; version: number; created_at: string; updated_at: string; updated_by: string | null }

duplicatePoster(request: { poster_id: string }): { id: string; campaign_id: string; campaign_product_id: string | null; template_id: string | null; page_number: number; design: string; status: 'pending' | 'rendered' | 'failed'; render_error: string | null; preview: FileRef | null; is_customised: boolean; version: number; created_at: string; updated_at: string; updated_by: string | null }

// Renders one poster at the chosen size and quality for preview before download.
previewExportPoster(request: { poster_id: string; format: 'png' | 'jpg'; output_width_px?: number; output_height_px?: number; quality?: number }): { preview?: FileRef | null; error?: string | null }

// Archives a campaign or permanently deletes it with its posters, products, events and exports.
removeCampaign(request: { campaign_id: string; action: 'archive' | 'delete'; confirmed: boolean }): { success: boolean }
```

## Pages
- **Campaign Builder** `/campaigns/builder` - Standalone bulk-production builder for campaigns.
- **Export Centre** `/exports/centre` - Generate and download campaign exports.
- **Campaign Editor** `/campaigns/editor` - Visual editor for individual campaign posters.

## Features
- **Campaign builder**
- **Export centre**
- **Campaign approval workflow**
- **Poster generation and editing**