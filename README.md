# V1 API Documentation

Create shortcut drafts, submit them for review, release updates, and check their status.

## Getting started

**Base URL:** `https://shortcuty.app/api/v1`

[Quickstart](#quickstart-create-a-draft-then-submit-it) · [Apple Shortcuts](#apple-shortcuts-setup) · [Endpoints](#endpoints) · [Fields](#shortcut-fields) · [Taglines](#taglines) · [Action counts](#action-counts) · [Troubleshooting](#errors-and-retries)

## Authentication

Get an API key from your account settings at [browse.shortcuty.app](https://browse.shortcuty.app). Shortcut and screenshot endpoints require this header:

```http
Authorization: Bearer YOUR_API_KEY
```

Only [Get Categories](#get-categories) is public. The [Shortcuty CLI](https://github.com/Shortcuty/Shortcuty-CLI) is also available.

## Quickstart: create a draft, then submit it

Replace the placeholders below with your own values.

### 1. Create a draft

```sh
curl --request POST 'https://shortcuty.app/api/v1/shortcuts' \
  --header 'Authorization: Bearer YOUR_API_KEY' \
  --header 'Content-Type: application/json' \
  --data '{"sharing_url":"https://www.icloud.com/shortcuts/REPLACE_WITH_ICLOUD_SHARE_ID","tagline":"A quick daily plan"}'
```

Save `shortcut.uuid` from the `201` response. This is the **Shortcuty UUID**, which is different from the ID in the iCloud link.

### 2. Submit the draft

```sh
curl --request POST 'https://shortcuty.app/api/v1/shortcuts/REPLACE_WITH_SHORTCUT_UUID/submit' \
  --header 'Authorization: Bearer YOUR_API_KEY'
```

This request has no body. Submission starts review and scanning; it does not mean approval.

### 3. Check its status

Use **GET** `/shortcuts/<uuid>` with the same Authorization header. The shortcut's `status` shows the result. For later version updates, use [Get Shortcut History](#get-shortcut-history).

## Request body formats

[Create Shortcut](#create-shortcut) and [Update Shortcut](#update-shortcut) support these formats with the same [field names](#shortcut-fields):

| Format | Content-Type | Client option |
|---|---|---|
| JSON | `application/json` | JSON object / Shortcuts **JSON** |
| URL-encoded Form | `application/x-www-form-urlencoded` | curl `--data-urlencode` |
| Multipart Form | `multipart/form-data` | curl `--form-string` / Shortcuts **Form** |

See the [JSON and Form examples](#update-examples) or the [validation reference](#request-validation-reference).

### Apple Shortcuts setup

In **Get Contents of URL**:

1. Set **Method → POST**.
2. Add **Authorization → `Bearer YOUR_API_KEY`** in Headers.
3. Choose **Request Body → JSON** or **Request Body → Form**.
4. Set the URL and add the fields for your task:

| Task | URL | Fields for a minimal example |
|---|---|---|
| Create a draft | `https://shortcuty.app/api/v1/shortcuts` | `sharing_url` as **Text** |
| Release an update | `https://shortcuty.app/api/v1/shortcuts/<uuid>/update` | `sharing_url`, `version`, `changelog`, all as **Text** |

Use the Shortcuty UUID in the update URL. For example, enter `1.2.0` in the version Text field. Additional fields and their types are in [Shortcut fields](#shortcut-fields).

## Endpoints

Paths below are relative to the base URL.

| Method | Path | Task |
|---|---|---|
| GET | `/categories` | [Get Categories](#get-categories) |
| POST | `/shortcuts` | [Create Shortcut](#create-shortcut) |
| GET | `/shortcuts/my` | [Get My Shortcuts](#get-my-shortcuts) |
| GET | `/shortcuts/<uuid>` | [Get Shortcut](#get-shortcut) |
| POST | `/shortcuts/<uuid>/update` | [Update Shortcut](#update-shortcut) |
| POST | `/shortcuts/<uuid>/submit` | [Submit Draft Shortcut](#submit-draft-shortcut) |
| GET | `/shortcuts/<uuid>/history` | [Get Shortcut History](#get-shortcut-history) |
| POST | `/upload/screenshot` | [Upload Screenshot](#upload-screenshot) |
| DELETE | `/shortcuts/<uuid>/screenshots/<screenshot_id>` | [Delete Screenshot](#delete-screenshot) |

## Categories

### Get Categories

**GET** `/categories`

**Authentication:** None.

**Response:** `200`, an object containing a `categories` array of valid category names. Use these values in the `category` or `categories` fields.

```json
{"categories":["Productivity","Utilities"]}
```

The example is abbreviated; the endpoint returns the full list.

## Shortcuts

### Create Shortcut

**POST** `/shortcuts`

**Body:** JSON or Form. `sharing_url` is required; see [Shortcut fields](#shortcut-fields) for optional fields. Use `auto_submit: true` to submit immediately, or omit it to create a draft.

**Response:** `201`, containing `message` and a [shortcut object](#shortcut-object). A submitted response also includes `scan_queued`.

```json
{
  "message": "Draft shortcut created",
  "shortcut": {
    "uuid": "123e4567-e89b-12d3-a456-426614174000",
    "status": "draft"
  }
}
```

This response is abbreviated. For a complete request, use the [quickstart](#quickstart-create-a-draft-then-submit-it).

### Get My Shortcuts

**GET** `/shortcuts/my`

Returns the authenticated account's shortcuts, including private shortcuts and all lifecycle statuses.

| Query parameter | Default |
|---|---|
| `page` | `1` |
| `per_page` | `20` |

**Response:** `200`, containing:

| Field | Value |
|---|---|
| `shortcuts` | Array of [shortcut objects](#shortcut-object) |
| `total` | Total matching shortcuts |
| `pages` | Total pages |
| `current_page` | Requested page |

### Get Shortcut

**GET** `/shortcuts/<uuid>`

Returns one shortcut. See [Visibility and pending versions](#visibility-and-pending-versions) for access rules.

**Response:** `200`, containing `shortcut`, a [shortcut object](#shortcut-object).

### Update Shortcut

**POST** `/shortcuts/<uuid>/update`

**Access:** Owner only.

**Body:** JSON or Form using the update fields in [Shortcut fields](#shortcut-fields). Send an applicable change; requests with no applicable changes may return `400`. An explicit `tagline` value is a proposal input even when it equals the current text, so a repeated tagline request can still return `200` or `201`.

Draft edits apply immediately. Other edits follow the applicable approval or review flow. If a proposal is already pending, see [Replacing a pending update](#replacing-a-pending-update).

**Response:** `200` or `201`; see [Understanding successful responses](#understanding-successful-responses).

#### Update examples

**JSON**

```sh
curl --request POST 'https://shortcuty.app/api/v1/shortcuts/REPLACE_WITH_SHORTCUT_UUID/update' \
  --header 'Authorization: Bearer YOUR_API_KEY' \
  --header 'Content-Type: application/json' \
  --data '{"sharing_url":"https://www.icloud.com/shortcuts/REPLACE_WITH_ICLOUD_SHARE_ID","version":"1.2.0","changelog":"Bug fixes"}'
```

**URL-encoded Form**

For Form requests, let the client set `Content-Type`. Remove any manual `application/json` header; multipart clients must also generate their own boundary.

```sh
curl --request POST 'https://shortcuty.app/api/v1/shortcuts/REPLACE_WITH_SHORTCUT_UUID/update' \
  --header 'Authorization: Bearer YOUR_API_KEY' \
  --data-urlencode 'sharing_url=https://www.icloud.com/shortcuts/REPLACE_WITH_ICLOUD_SHARE_ID' \
  --data-urlencode 'version=1.2.0' \
  --data-urlencode 'changelog=Bug fixes'
```

**Multipart Form**

```sh
curl --request POST 'https://shortcuty.app/api/v1/shortcuts/REPLACE_WITH_SHORTCUT_UUID/update' \
  --header 'Authorization: Bearer YOUR_API_KEY' \
  --form-string 'sharing_url=https://www.icloud.com/shortcuts/REPLACE_WITH_ICLOUD_SHARE_ID' \
  --form-string 'version=1.2.0' \
  --form-string 'changelog=Bug fixes'
```

### Replacing a pending update

A new update request replaces the pending proposal. Include all unapproved changes and release notes you want to keep. Omitted proposal fields are dropped **except `tagline`**: omitting it preserves the pending tagline, including a pending clear, for compatibility with older clients. Send `tagline` explicitly to replace or clear it.

For example, if the pending proposal changes the URL and version, sending only `changelog` drops those proposed URL and version changes. Send all three fields to retain them.

### Submit Draft Shortcut

**POST** `/shortcuts/<uuid>/submit`

**Access:** Owner only; the shortcut must be draft or rejected.

**Body:** None.

**Response:** `200` when scanning is admitted, or `202` when admission is deferred for recovery. Both include `message` and `scan_queued`. The submission remains pending.

### Get Shortcut History

**GET** `/shortcuts/<uuid>/history`

**Access:** See the owner-only history rule in [Visibility and pending versions](#visibility-and-pending-versions).

**Response:** `200`, containing `shortcut_name`, `shortcut_uuid`, and a `changelog` array:

```json
{
  "shortcut_name": "Example Shortcut",
  "shortcut_uuid": "123e4567-e89b-12d3-a456-426614174000",
  "changelog": [{
    "version": "1.2.0",
    "action_count": null,
    "changelog": "Bug fixes",
    "date": "2026-10-15T12:00:00Z",
    "status": "pending",
    "sharing_url": "https://www.icloud.com/shortcuts/EXAMPLE_SHARE_ID",
    "changes": {"new_version": "1.2.0"}
  }]
}
```

Entries include approved, pending, and rejected updates. `changes` contains proposed field changes; sharing URLs are exposed separately in `sharing_url`.

Each entry also includes `action_count`. It is known only when the stored accepted publication proves that exact current approved update. Historical releases, pending/rejected updates, and other unproven entries return `null`; the server does not copy a count between entries that share a version label or URL. See [Action counts](#action-counts).

### Visibility and pending versions

| State or resource | Owner's API key | Another account's API key |
|---|---|---|
| Initial draft, pending, rejected, or delisted shortcut | Available in list/detail | Detail returns `404` |
| Approved shortcut with a pending update | List/detail show the approved base version | Detail shows the approved base version |
| Update history | Includes pending, approved, and rejected entries | `404`, including for an admin key |

An initial submission is stored on the shortcut itself; history does not synthesize an entry for it. While an update is pending, the approved version stays live. Draft edits, visibility changes, and automatically approved edits can apply immediately.

## Screenshots

### Upload Screenshot

**POST** `/upload/screenshot`

**Body:** Multipart Form.

| Field | Required | Value |
|---|---|---|
| `file` | Yes | PNG, JPEG, or GIF image file |
| `shortcut_id` | Yes | Shortcuty UUID |

Maximum five screenshots per shortcut. File extensions and file content are validated.

**Response:** `200`, containing `screenshot` with `id`, `shortcut_id`, `filename`, `url`, `order`, and `created_at`.

### Delete Screenshot

**DELETE** `/shortcuts/<uuid>/screenshots/<screenshot_id>`

**Response:** `200`.

```json
{"message":"Screenshot deleted successfully"}
```

## Shortcut fields

The table is the request field reference for create and update. A dash means the field is not accepted by that endpoint.

| Field | Type | Create | Update | Values / limit |
|---|---|---|---|---|
| `sharing_url` | String | **Required** | Optional | iCloud shortcut sharing URL |
| `description` | String or null | Optional | Optional | Up to 500,000 characters |
| `tagline` | String or null | Optional | Optional | Single-line plain text; up to 120 Unicode scalar values **after normalization**; see [Taglines](#taglines) |
| `category` | String or null | Optional | Optional | Primary; legacy writes preserve secondaries; see [Multiple categories](#multiple-categories) |
| `categories` | String array | Optional | Optional | 1–3 unique preset names; requires `primary_category` |
| `primary_category` | String | With `categories` | With `categories` | Exactly one member; agrees with `category` if supplied |
| `category_revision` | Integer | Omit | With explicit category edits | Last response's revision; stale edits return `409` |
| `requires_ios26_ai` | Boolean | Optional; default `false` | Optional | Whether iOS 26 AI features are required |
| `updater_type` | String | Optional; default `none` | Optional | `shortcuty`, `third_party`, `none` |
| `visibility` | String | Optional; default `public` | Optional | `public`, `private` |
| `auto_submit` | Boolean | Optional; default `false` | — | Submit at creation instead of saving a draft |
| `version` | String | — | Optional | Up to 50 characters, e.g. `"1.2.0"` |
| `changelog` | String | — | Optional | Release notes, up to 10,000 characters |

`action_count` is a response-only field. Sending it in a create or update body returns `400`; it cannot override the server's count.

### Multiple categories

The multi-category release adds `categories`, `primary_category`, and `category_revision` while retaining the existing scalar `category` as the primary. These changes require the coordinated backend schema rollout; this candidate documentation must be published with that release.

Use one to three unique exact category names from [Get Categories](#get-categories), with exactly one `primary_category` contained in the array. If `category` is also sent, it must equal that primary. On an update, include the last shortcut response's `category_revision`; a stale value returns `409`, and no fields are changed. Creation does not require a revision.

```json
{
  "categories": ["Productivity", "Utilities", "Education"],
  "primary_category": "Productivity",
  "category_revision": 4
}
```

Omit all category fields to keep the current selection, including pending category proposals. Explicitly send `["Productivity"]` with that primary to remove secondary categories. Empty/null arrays, duplicates, unknown names, more than three members, missing/conflicting primary values, and invalid field types return `400` atomically. Form callers put the JSON array in one `categories` Text field and the primary/revision in their own Text fields; repeated fields remain invalid.

Existing clients can continue sending only `category`. Selecting an existing secondary promotes it while retaining the other members. A new category replaces the old primary and preserves its secondaries, including when there are already three members. An unchanged legacy scalar preserves a pending primary selection. A blank/null scalar removes only the primary and promotes the first secondary; with no remaining members it preserves the existing uncategorized behavior. Historical uncategorized shortcuts return `category: null`, `primary_category: null`, and `categories: []`; no arbitrary category is assigned.

Responses return the primary first in `categories`; `primary_category` and `category` agree. Category-only edits keep the current version and existing review behavior. Pending sets are preserved in version history `changes.categories` and `changes.primary_category`, and are published only by the normal approval transaction. Every actual selection/proposal change advances `category_revision`; older clients merge under lock without needing that field.

Category browsing/filtering matches any membership. A shortcut counts once in each matching category, so adding category totals can exceed the total number of shortcuts. Pending, private and unapproved data do not enter public counts. Category filtering does not change search ranking.

### Taglines

`tagline` is optional plain text shown below the shortcut name and on cards. It is supported in JSON, URL-encoded Form, and multipart Form **Text** fields on create/update. Taglines do not accept file uploads or a separate approval endpoint.

The server collapses each whitespace run (including line breaks) to one ASCII space, trims surrounding whitespace, and normalizes to NFC. The resulting value can contain at most **120 Unicode scalar values**; the limit is not bytes, UTF-16 code units, or displayed grapheme clusters. Remaining control characters and lone surrogates are rejected with `400`. A decomposed `e` plus combining acute accent normalizes to one `é`; an emoji outside the BMP counts as one scalar. Normalization happens before the length check.

| Request value | Create | Update |
|---|---|---|
| Omitted | Store no tagline (`null`) | Preserve the current tagline, or the tagline in an existing pending proposal |
| Nonblank string | Store the normalized text | Propose/apply the normalized replacement |
| `""`, whitespace-only, or JSON `null` | Store no tagline | Explicitly clear the tagline |
| Form text `null` | Store the literal word `null` | Propose the literal word `null`; use an empty Form field to clear |
| Number, boolean, array, or object | `400` | `400` |

For example, `{"tagline":"  A\r\nquick\tplan  "}` stores `A quick plan`. A tagline-only edit does not require a new sharing URL, version, or changelog:

```sh
curl --request POST 'https://shortcuty.app/api/v1/shortcuts/REPLACE_WITH_SHORTCUT_UUID/update' \
  --header 'Authorization: Bearer YOUR_API_KEY' \
  --header 'Content-Type: application/json' \
  --data '{"tagline":"A faster daily plan"}'
```

Clear it with `--data '{"tagline":null}'` in JSON or `--data-urlencode 'tagline='` in a URL-encoded Form request. The dedicated `/submit` request remains bodyless and reviews the stored tagline; edit it through `/update` before submitting if needed.

Draft edits apply immediately. Editing a first pending submission keeps it pending and queues a fresh scan. On an approved shortcut, a changed tagline joins the normal AI assessment of the shortcut and description; there is no separate tagline-review step. It is not an approval promise: unsafe, incomplete, or failed assessments remain in the existing review/retry flow. The approved tagline remains live while its replacement is pending. Existing rejected-submission safeguards still apply.

`shortcut.tagline` is present in create/detail/list shortcut objects and is `null` when cleared. Pending/history changes expose an explicit clear as `changes.tagline: null`. A tagline-only edit leaves `version` unchanged; the plain-text version and download-redirect endpoints keep their existing formats. Ordinary public/app JSON shortcut responses also include the original tagline, subject to their existing access rules; this does not make Creator API writes public.

Tagline translations are separate from author source. Normal submissions/edits and opt-in translation reads can queue matching translations. Changed/cleared taglines cannot reuse a translation for different source text; while a translation is unavailable, clients retain the original. Creator `/api/v1` detail/list responses always return original author text. Saved translation reads are separate, outside the Creator API: `GET /api/shortcuts/<uuid>/translations/<target>?include_tagline=1` or `POST /api/shortcuts/translations/batch` with `include_tagline: true`. Batch items use numeric IDs from ordinary JSON shortcut metadata, not Creator API UUIDs. These routes retain their existing optional-session access checks: approved public shortcuts are readable anonymously; nonpublic/pending sources require an authorized owner/admin session. A Creator Bearer key does not grant that session access. Opt-in source hashes include name, description, and tagline (`shortcut-fields-v2`); legacy translation requests retain their name/description hash. Apply translated text only with a matching target and `field_hashes.tagline` (SHA-256 of original UTF-8 source). Never write translated display text back as the author's source.

### Action counts

Shortcut objects in create, list, and detail responses include the additive read-only field `action_count`. Ordinary app/website JSON shortcut responses, including catalogue lists, trending/recommendations, search, categories, feeds, and detail, use the same field and semantics under their existing access rules.

| Value | Meaning |
|---|---|
| Nonnegative integer | Number of entries in the accepted workflow's original, validated `WFWorkflowActions` array |
| `0` | The inspected workflow has an empty action array |
| `null` | A count for the published version is unknown or temporarily unavailable |
| Field absent | An older server response; handle it as unknown |

For example, shortcut response objects may contain:

```json
{"version":"1.2.0","action_count":27}
```

```json
{"version":"1.2.0","action_count":0}
```

```json
{"version":"1.2.0","action_count":null}
```

Counts come from the original workflow bytes during normal processing, independently of AI analysis. They count stored action entries, including control-flow and comment entries, without expanding nested shortcuts or estimating how many actions run. They are not derived from descriptions, generated pseudocode, model guesses, or client input.

The count belongs to the currently accepted published version. While an update is pending, list/detail retain the approved base version's count. Draft, pending, rejected, and delisted shortcut objects return `null`. An older shortcut with no proven accepted count, a changed source, or unavailable count storage also returns `null`; a successful security assessment alone does not guarantee a count is available. Reading the field does not trigger a scan or change moderation status.

JSON changelog/history entries and the ordinary detail response's `latest_update` object also include `action_count`, subject to the exact-update rule in [Get Shortcut History](#get-shortcut-history). Plain-text version endpoints, download links, and redirect formats retain their existing formats.

Clients should allow missing/null values and keep unknown distinct from zero. Show a numeric count only when it is a nonnegative integer; never substitute `0` for an unknown count. Existing response fields and request workflows remain compatible. Counts do not expose private workflow content, security reports, or pending scans.

### Field names and common mistakes

| Use | Instead of |
|---|---|
| `sharing_url` | `link` |
| `changelog` | `changes` |
| `auto_submit` during creation | `submit` |
| The name extracted from iCloud | Sending the read-only `name` field |

Unknown and read-only request fields return `400`.

## Request validation reference

These rules apply to shortcut creation and update bodies.

| Rule | Accepted | Rejected |
|---|---|---|
| Body structure | Nonempty JSON object or Form fields at the top level | Empty body, JSON array, wrappers such as `data` or `shortcut` |
| Field uniqueness | Each field once | Duplicate JSON or Form fields, even with identical values |
| Text values | JSON strings or Form Text fields | JSON numbers, objects, or arrays for text fields |
| Boolean values | JSON `true` / `false`; Form text `true` / `false` (case and surrounding whitespace ignored) | JSON strings/numbers/null; other Form boolean text |
| Clearing optional values | JSON `null` for `description`, `category`, or `tagline` | JSON `null` for other fields; Form `null` remains literal text |
| Files | Use [Upload Screenshot](#upload-screenshot) | File parts in shortcut creation or update bodies |

## Response Fields

### Shortcut Object

| Field | Type / values |
|---|---|
| `uuid` | String; Shortcuty UUID |
| `name` | String; extracted from iCloud |
| `description` | String or null |
| `tagline` | String or null; normalized original author text |
| `sharing_url` | String |
| `version` | String or null |
| `action_count` | Nonnegative integer or null; read-only count for the accepted published version; see [Action counts](#action-counts) |
| `status` | `draft`, `pending`, `approved`, `rejected`, `delisted` |
| `category` | String or null, primary |
| `categories` | Array of names, primary first; empty only for legacy uncategorized records |
| `primary_category` | String or null; same primary as `category` |
| `category_revision` | Nonnegative integer for explicit edit concurrency |
| `created_at`, `updated_at` | ISO 8601 string or null |
| `requires_ios26_ai` | Boolean |
| `updater_type` | `shortcuty`, `third_party`, `none` |
| `visibility` | `public`, `private` |
| `rejection_reason` | String or null; included when status is rejected |

### Understanding successful responses

| Request | Status | Meaning |
|---|---|---|
| Create | `201` | A draft was created, or submitted when `auto_submit` is `true`. |
| Update | `200` | Draft/visibility edits applied, automatic approval completed, or an initial pending submission was edited. Read `message` for the outcome. |
| Update | `201` | An update was accepted for processing, scanning, or manual review. Approval is still required. |
| Submit | `200` / `202` | See [Submit Draft Shortcut](#submit-draft-shortcut). |

| Response field | Meaning |
|---|---|
| `message` | Describes the outcome. |
| `update_id` | Identifies an update record, when created or replaced. |
| `status` | For update-record responses, `pending` or `approved`. |
| `replaced_pending` | For update-record responses, whether an existing pending proposal was replaced. |
| `scan_queued` | When present, `true` means scan admission succeeded; `false` means it was deferred for automatic recovery. |

## Errors and retries

Errors use this JSON shape:

```json
{"error":"Field version must be a string"}
```

| Status | Meaning |
|---|---|
| `400` | Invalid body, field, value, or lifecycle action. Check the error message and [validation reference](#request-validation-reference). |
| `401` | Missing or invalid API key. |
| `403` | Inactive account or insufficient ownership for the requested change. |
| `404` | Not found or access is unavailable; see [Visibility and pending versions](#visibility-and-pending-versions). |
| `413` | Request exceeds the server's body size limit. |
| `415` | Missing or unsupported `Content-Type`; see [Request body formats](#request-body-formats). |
| `500` | Server error; the request may already have saved data. |

### Checking an uncertain result

After a server error or lost response, check the result before manually repeating a POST:

| You were trying to… | Check with GET |
|---|---|
| Create a shortcut | `/shortcuts/my` |
| Submit an initial version | `/shortcuts/<uuid>` |
| Release an update | `/shortcuts/<uuid>/history` |

These checks use the owner's Authorization header. A repeated update can [replace the pending proposal](#replacing-a-pending-update).

## Broken links and update notifications

Use **Report → Broken link** on a shortcut detail page if its iCloud link no longer opens. Repeating a report does not create another open report; submission limits keep the review queue useful. Reporting does not remove the shortcut.

Start an update changelog with the most useful change. After approval, notifications for a changed iCloud link can include a short plain-text excerpt (up to 180 characters) and open the correct Shortcuty detail page. Existing likes, follows and notification preferences still determine delivery. Pending or rejected release notes are never used.

## Requests API

[Requests API v1](REQUESTS-API.md) uses the existing account session under `/api`, with capability-gated availability. Its authentication is separate from the bearer-key V1 shortcut endpoints above.
