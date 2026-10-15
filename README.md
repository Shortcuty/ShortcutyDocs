# V1 API Documentation

Create shortcut drafts, submit them for review, release updates, and check their status.

## Getting started

**Base URL:** `https://shortcuty.app/api/v1`

[Quickstart](#quickstart-create-a-draft-then-submit-it) · [Apple Shortcuts](#apple-shortcuts-setup) · [Endpoints](#endpoints) · [Fields](#shortcut-fields) · [Troubleshooting](#errors-and-retries)

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
  --data '{"sharing_url":"https://www.icloud.com/shortcuts/REPLACE_WITH_ICLOUD_SHARE_ID"}'
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

**Response:** `200`, an object containing a `categories` array of valid category names. Use these values in the `category` field.

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

**Body:** JSON or Form using the update fields in [Shortcut fields](#shortcut-fields). Send a change; unchanged values alone return `400`.

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

A new update request replaces the **entire pending proposal**. Include all unapproved changes and release notes you want to keep; omitted proposal fields are dropped.

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
    "changelog": "Bug fixes",
    "date": "2026-10-15T12:00:00Z",
    "status": "pending",
    "sharing_url": "https://www.icloud.com/shortcuts/EXAMPLE_SHARE_ID",
    "changes": {"new_version": "1.2.0"}
  }]
}
```

Entries include approved, pending, and rejected updates. `changes` contains proposed field changes; sharing URLs are exposed separately in `sharing_url`.

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
| `category` | String or null | Optional | Optional | A value from [Get Categories](#get-categories) |
| `requires_ios26_ai` | Boolean | Optional; default `false` | Optional | Whether iOS 26 AI features are required |
| `updater_type` | String | Optional; default `none` | Optional | `shortcuty`, `third_party`, `none` |
| `visibility` | String | Optional; default `public` | Optional | `public`, `private` |
| `auto_submit` | Boolean | Optional; default `false` | — | Submit at creation instead of saving a draft |
| `version` | String | — | Optional | Up to 50 characters, e.g. `"1.2.0"` |
| `changelog` | String | — | Optional | Release notes, up to 10,000 characters |

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
| Clearing optional values | JSON `null` for `description` or `category` | JSON `null` for other fields; Form `null` remains literal text |
| Files | Use [Upload Screenshot](#upload-screenshot) | File parts in shortcut creation or update bodies |

## Response Fields

### Shortcut Object

| Field | Type / values |
|---|---|
| `uuid` | String; Shortcuty UUID |
| `name` | String; extracted from iCloud |
| `description` | String or null |
| `sharing_url` | String |
| `version` | String or null |
| `status` | `draft`, `pending`, `approved`, `rejected`, `delisted` |
| `category` | String or null |
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
