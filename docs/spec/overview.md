# Dashboard & Settings

PromoBaker dashboard with key counts and recent activity, plus workspace settings.

Shows total products, imported documents, pending reviews, available templates, active campaigns and recent activity. Settings holds organisation defaults and review confidence threshold.

## Entities
```ts
interface ActivityLog {
  id: string;
  event_type: string;
  description: string;
  actor_id: string;
  created_at: string;
  link_path: string | null;
  dedupe_key: string | null;
}

interface WorkspaceSetting {
  id: string;
  organization_name: string;
  default_currency: string;
  default_export_format: 'pdf' | 'png' | 'jpg';
  review_confidence_threshold: number;
}

interface AIUsageLog {
  id: string;
  operation: string;
  status: 'success' | 'failed' | 'timeout';
  input_tokens: number | null;
  output_tokens: number | null;
  duration_ms: number | null;
  retries: number;
  error_message: string | null;
  import_document_id: string | null;
  actor_id: string;
  created_at: string;
}
```

**Table names:** `activity _log` (Activity Log), `workspace _setting` (Workspace Setting), `a_i _usage _log` (AI Usage Log)

## APIs
```ts
// Counts for the dashboard.
getDashboardSummary(request: void): { totalProducts: number; totalImportedDocuments: number; pendingReviews: number; availableTemplates: number; activeCampaigns: number; missingProductImages: number; completedExports: number; outstandingTasks: { kind: 'review_pending_products' | 'fix_failed_imports' | 'review_image_matches' | 'approve_campaigns'; title: string; count: number; pagePath: string }[] }

listRecentActivity(request: { limit?: number }): { items: ActivityLog[] }

getSettings(request: void): { id: string; organization_name: string; default_currency: string; default_export_format: 'pdf' | 'png' | 'jpg'; review_confidence_threshold: number }

updateSettings(request: { organization_name: string; default_currency: string; default_export_format: 'pdf' | 'png' | 'jpg'; review_confidence_threshold: number }): { id: string; organization_name: string; default_currency: string; default_export_format: 'pdf' | 'png' | 'jpg'; review_confidence_threshold: number }

// Token and request usage of AI calls.
getAiUsage(request: { limit?: number }): { total_requests: number; failed_requests: number; total_input_tokens: number; total_output_tokens: number; recent: AIUsageLog[] }
```

## Pages
- **Dashboard** `/dashboard/overview` - PromoBaker overview: key counts and recent activity.
- **Settings** `/settings/general` - Organisation defaults and review threshold.
- **Landing** `/welcome/landing` - Marketing-style landing page introducing PromoBaker and its four-step workflow.

## Features
- **Dashboard overview**
- **Workspace settings**