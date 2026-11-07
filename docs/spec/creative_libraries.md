# Assets, Brand Kits & Templates

Libraries of uploaded assets, brand kits and layout templates used by campaigns.

Users upload images and files, define brand kits (logo, colours, fonts, tone) and manage templates for flyers, posters and social posts.

## Entities
```ts
interface Asset {
  id: string;
  name: string;
  file: FileRef | null;
  tags: string | null;
  created_by: string;
  created_at: string;
  asset_type: 'image' | 'logo' | 'document' | 'other';
  category: string | null;
  product_id: string | null;
  import_document_id: string | null;
}

interface BrandKit {
  id: string;
  name: string;
  logo: FileRef | null;
  primary_color: string;
  secondary_color: string;
  accent_color: string | null;
  heading_font: string;
  body_font: string;
  tone_guidelines: string | null;
  created_at: string;
  secondary_colors: string[] | null;
}

interface Template {
  id: string;
  name: string;
  description: string | null;
  format: 'flyer' | 'poster' | 'social_post';
  width_px: number;
  height_px: number;
  preview: FileRef | null;
  layout_notes: string | null;
  status: 'active' | 'archived';
  created_at: string;
  brand_kit_id: string | null;
  configuration: string | null;
  category: string | null;
  size_preset: 'a4_portrait' | 'a4_landscape' | 'square_1080' | 'portrait_1080x1350' | 'custom' | null;
  background_file: FileRef | null;
  reference_file: FileRef | null;
  is_exportable: boolean;
  render_verified_at: string | null;
}
```

**Table names:** `asset` (Asset), `brand _kit` (Brand Kit), `template` (Template)

## APIs
**Files with call_api / call_streaming_api** (also from run_typescript):
- A `FileRef` in a result — at any depth, entities and lists included — arrives as a file object `{ fileRef, filePath: "filerefs/<id>", filename, mimeType, size }` (filename, mimeType and size when its metadata could be read; `missing: true` when this workspace has no such file). Read, analyze or copy the file at `filePath`; to show it to the user, `copy_file` it into `artifacts/` and `present_files` that.
- A `FileRef` parameter takes `{ filePath: "<any readable VFS path, e.g. artifacts/report.pdf>" }` (uploaded for the call; optional `filename` / `mimeType`), a file object from a previous result as-is, or an existing FileRef id. A plain path string is not a FileRef.
- `run_sql` rows hold the bare FileRef id; the file is at `filerefs/<id>`.

```ts
createAsset(request: { name: string; file: FileRef; tags?: string; asset_type?: 'image' | 'logo' | 'document' | 'other'; category?: string | null; product_id?: string | null; import_document_id?: string | null }): { id: string; name: string; file: FileRef | null; tags: string | null; created_by: string; created_at: string; asset_type: 'image' | 'logo' | 'document' | 'other'; category: string | null; product_id: string | null; import_document_id: string | null }

listAssets(request: { search?: string }): { items: Asset[] }

deleteAsset(request: { asset_id: string }): { success: boolean }

saveBrandKit(request: { id?: string; name: string; logo?: FileRef | null; primary_color: string; secondary_color: string; accent_color?: string | null; heading_font: string; body_font: string; tone_guidelines?: string | null; secondary_colors?: string[] | null }): { id: string; name: string; logo: FileRef | null; primary_color: string; secondary_color: string; accent_color: string | null; heading_font: string; body_font: string; tone_guidelines: string | null; created_at: string; secondary_colors: string[] | null }

listBrandKits(request: void): { items: BrandKit[] }

saveTemplate(request: { id?: string; name: string; description?: string | null; format: 'flyer' | 'poster' | 'social_post'; width_px: number; height_px: number; preview?: FileRef | null; layout_notes?: string | null; status: 'active' | 'archived'; brand_kit_id?: string | null; configuration?: string | null; category?: string | null; size_preset?: 'a4_portrait' | 'a4_landscape' | 'square_1080' | 'portrait_1080x1350' | 'custom' | null; background_file?: FileRef | null; reference_file?: FileRef | null }): { id: string; name: string; description: string | null; format: 'flyer' | 'poster' | 'social_post'; width_px: number; height_px: number; preview: FileRef | null; layout_notes: string | null; status: 'active' | 'archived'; created_at: string; brand_kit_id: string | null; configuration: string | null; category: string | null; size_preset: 'a4_portrait' | 'a4_landscape' | 'square_1080' | 'portrait_1080x1350' | 'custom' | null; background_file: FileRef | null; reference_file: FileRef | null; is_exportable: boolean; render_verified_at: string | null }

listTemplates(request: { format?: 'flyer' | 'poster' | 'social_post' }): { items: Template[] }

// Resolves template placeholders with real approved products and renders a test image.
previewTemplate(request: { template_id: string; product_id?: string; verify?: boolean }): { bindings: { placeholder: string; value?: string | null }[]; rendered?: FileRef | null; is_exportable: boolean; error?: string | null }

// AI suggests brand kit values from an imported brand document; suggestions only.
suggestBrandValues(request: { document_id: string }): { primary_color?: string | null; secondary_colors: string[]; heading_font?: string | null; body_font?: string | null; tone_guidelines?: string | null }
```

## Pages
- **Asset Library** `/assets/library` - Upload and manage images and files.
- **Brand Kits** `/brand/kits` - Manage brand kits.
- **Template Library** `/templates/library` - Manage layout templates.

## Features
- **Asset library**
- **Brand kits**
- **Template library**