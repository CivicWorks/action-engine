# Action Engine — Data Model

## Core Entities

### Organization (`organizations`)

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Primary key |
| `name` | VARCHAR(255) | Organization display name |
| `slug` | VARCHAR(128) | URL-friendly identifier |
| `description` | TEXT | Organization mission/description |
| `logo_url` | VARCHAR(512) | Organization logo |
| `website_url` | VARCHAR(512) | Organization website |
| `contact_email` | VARCHAR(255) | Primary contact (encrypted at rest) |
| `trust_score` | DECIMAL(3,2) | 0.00–1.00, computed from submission history |
| `status` | ENUM | `active`, `warned`, `suspended`, `terminated` |
| `flag_count` | INTEGER | Cumulative flags for policy violations |
| `created_at` | TIMESTAMP | |
| `updated_at` | TIMESTAMP | |

### Organization Member (`org_members`)

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Primary key |
| `org_id` | UUID | FK → organizations |
| `display_name` | VARCHAR(255) | Name shown in audit trail (no PII stored beyond this) |
| `auth_hash` | VARCHAR(512) | Hashed authentication credential |
| `role` | ENUM | `creator`, `editor`, `admin` |
| `status` | ENUM | `active`, `suspended` |
| `created_at` | TIMESTAMP | |

### Issue (`issues`)

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Primary key |
| `name` | VARCHAR(128) | Display name |
| `slug` | VARCHAR(128) | URL-friendly identifier |
| `description` | TEXT | Brief description of the issue area |
| `parent_id` | UUID | FK → issues (nullable, for future nesting) |
| `sort_order` | INTEGER | Display ordering |
| `active` | BOOLEAN | Soft enable/disable |

**Seed values:** Uncategorized, Worker Rights/Unions, Reproductive Rights, Infrastructure, Immigration, Health Care, Gun Violence Prevention, Government, Global Peace, Equality, Climate/Environment, Voting Rights, Corruption, Education, Housing, Criminal Justice Reform

### Action (`actions`)

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Primary key |
| `org_id` | UUID | FK → organizations |
| `created_by` | UUID | FK → org_members |
| `source` | ENUM | `native`, `mobilize_import`, `api_ingest` |
| `source_ref` | VARCHAR(512) | External ID/URL if imported |
| `title` | VARCHAR(255) | Action title |
| `description` | TEXT | Rich-text description |
| `image_url` | VARCHAR(512) | Action image/banner |
| `action_url` | VARCHAR(512) | External URL with more info |
| `action_type` | ENUM | `physical`, `virtual`, `educational` |
| `weight` | ENUM | `lightweight`, `moderate`, `heavyweight` |
| `start_date` | TIMESTAMP | |
| `end_date` | TIMESTAMP | Nullable for open-ended actions |
| `all_day` | BOOLEAN | |
| `timezone` | VARCHAR(64) | IANA timezone string |
| `is_virtual` | BOOLEAN | |
| `venue` | VARCHAR(255) | Nullable if virtual |
| `street_address` | VARCHAR(255) | |
| `city` | VARCHAR(128) | |
| `state_region` | VARCHAR(128) | |
| `postal_code` | VARCHAR(32) | |
| `country` | VARCHAR(64) | |
| `latitude` | DECIMAL(10,7) | Geocoded for proximity matching |
| `longitude` | DECIMAL(10,7) | Geocoded for proximity matching |
| `scope` | ENUM | `local`, `state_regional`, `national`, `global` |
| `visibility` | ENUM | `public`, `subscribers_only` |
| `status` | ENUM | `draft`, `pending_ai_review`, `auto_approved`, `flagged_for_review`, `approved`, `rejected`, `published`, `expired`, `cancelled` |
| `ai_review_score` | DECIMAL(3,2) | AI confidence score |
| `ai_review_notes` | TEXT | AI review feedback (structured JSON) |
| `rejection_reason` | TEXT | Reason sent back to org if rejected |
| `published_at` | TIMESTAMP | |
| `created_at` | TIMESTAMP | |
| `updated_at` | TIMESTAMP | |

### Action Issues (`action_issues`) — many-to-many

| Field | Type | Description |
|-------|------|-------------|
| `action_id` | UUID | FK → actions |
| `issue_id` | UUID | FK → issues |

### Subscriber Issue Affinity (`subscriber_affinities`)

| Field | Type | Description |
|-------|------|-------------|
| `subscriber_id` | UUID | Anonymized subscriber identifier |
| `issue_id` | UUID | FK → issues |
| `weight` | DECIMAL(3,2) | 0.00–1.00, strength of affinity |
| `geo_lat` | DECIMAL(10,7) | Subscriber general location (reduced precision for privacy) |
| `geo_lng` | DECIMAL(10,7) | Reduced precision — city-level, not street |
| `scope_preference` | ENUM | `local`, `state_regional`, `national`, `global`, `all` |
| `updated_at` | TIMESTAMP | |

**Privacy note:** Subscriber location is stored at reduced precision (~city level). No street addresses, no precise coordinates. Affinity weights are derived from engagement, not stored preferences where possible.

### Action Distribution Log (`distribution_log`)

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Primary key |
| `action_id` | UUID | FK → actions |
| `channel` | ENUM | `bluesky`, `civicworks_feed`, `subscriber_push`, `email_digest` |
| `distributed_at` | TIMESTAMP | |
| `recipient_count` | INTEGER | Aggregate count, no individual tracking |
| `status` | ENUM | `sent`, `failed`, `partial` |

### Action Analytics (`action_analytics`) — aggregate only

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Primary key |
| `action_id` | UUID | FK → actions |
| `period` | DATE | Aggregation date |
| `views` | INTEGER | |
| `calendared` | INTEGER | Subscribers who added to calendar |
| `completed` | INTEGER | Subscribers who marked complete |
| `shared` | INTEGER | Social shares |

**Privacy note:** All analytics are aggregate. No individual subscriber behavior is linked to analytics records. Organizations receive totals, not subscriber lists.

### Org Review History (`org_review_history`)

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Primary key |
| `org_id` | UUID | FK → organizations |
| `action_id` | UUID | FK → actions |
| `event` | ENUM | `auto_approved`, `flagged`, `rejected`, `warning_issued`, `suspended`, `terminated` |
| `reason` | TEXT | |
| `created_at` | TIMESTAMP | |

Used for computing trust scores and detecting patterns of ill-formed or harmful submissions.

---

## Key Design Decisions

1. **Privacy by architecture**: Subscriber data uses anonymized IDs, reduced-precision geolocation, and aggregate-only analytics. No PII flows to organizations.

2. **Source-agnostic actions**: Every action has a `source` field. Native, Mobilize.us imports, and future API ingestion all normalize to the same schema and go through the same AI review pipeline.

3. **AI-first review**: The `status` field workflow is `pending_ai_review` → `auto_approved` (happy path) or `flagged_for_review` (exception path). Human review is the exception, not the rule.

4. **Org accountability**: `trust_score`, `flag_count`, and `org_review_history` combine to enable automated escalation: pattern of rejections → warning → suspension → termination.

5. **Extensible issue taxonomy**: Issues support `parent_id` for future hierarchical nesting without breaking the flat-list prototype.

6. **Geocoded actions**: Latitude/longitude on actions enables proximity-based matching to subscriber locations for distribution targeting.
