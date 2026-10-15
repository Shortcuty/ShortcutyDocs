# V1 API Documentation

Create shortcut drafts, submit them for review, publish updates, and check their status. Submitted shortcuts and updates can require scanning and review before approval.

## Getting started

**Base URL:** `https://shortcuty.app/api/v1`

Prefer a command line interface? Use the [Shortcuty CLI](https://github.com/Shortcuty/Shortcuty-CLI).

Get your API key in your account settings at [browse.shortcuty.app](https://browse.shortcuty.app). Use it in the `Authorization` header as shown below.

## Authentication

All shortcut and screenshot endpoints below require an API key. [Get Categories](#get-categories) is public:

```
Authorization: Bearer <your_api_key>
```

---

## Quickstart: create a draft, then submit it

Replace every `YOUR_API_KEY` and `REPLACE_WITH_...` placeholder with your own value. The iCloud share ID is part of the iCloud link; the Shortcuty UUID comes from the API response.

**1. Create a draft.** Only `sharing_url` is required; `auto_submit` defaults to `false`.

```sh
curl --request POST 'https://shortcuty.app/api/v1/shortcuts' \
  --header 'Authorization: Bearer YOUR_API_KEY' \
  --header 'Content-Type: application/json' \
  --data '{"sharing_url":"https://www.icloud.com/shortcuts/REPLACE_WITH_ICLOUD_SHARE_ID"}'
```

A successful `201` response contains `shortcut.uuid` and `shortcut.status`. Save that UUID. The shortcut is a draft and is not published. [Update the draft](#update-shortcut) if needed, or inspect it with **GET** `/api/v1/shortcuts/<uuid>` using the same Authorization header.

**2. Submit when ready.** This request has no body.

```sh
curl --request POST 'https://shortcuty.app/api/v1/shortcuts/REPLACE_WITH_SHORTCUT_UUID/submit' \
  --header 'Authorization: Bearer YOUR_API_KEY'
```

`200` means submitted for review and scanning; `202` means submitted with scan admission deferred for automatic recovery. Neither means approved. To submit at creation instead, set `auto_submit` to `true` in the create request.

**3. Check the result.** Use **GET** `/api/v1/shortcuts/my` or **GET** `/api/v1/shortcuts/<uuid>` to see an initial submission's status. For later updates, use the owner-only **GET** `/api/v1/shortcuts/<uuid>/history` to see pending, approved, and rejected versions. All these requests need the owner's API key.

---

## Request body formats

The following endpoints accept the same field names and validation rules in either JSON or Form bodies:

- **POST** `/api/v1/shortcuts`
- **POST** `/api/v1/shortcuts/<uuid>/update`

| Format | Content-Type | Values |
|---|---|---|
| JSON | `application/json` | One JSON object. Text fields must be strings, including `version`. Boolean fields use JSON `true` or `false`. |
| URL-encoded Form | `application/x-www-form-urlencoded` | Text fields. Boolean fields use the text `true` or `false`. |
| Multipart Form | `multipart/form-data` | Text fields. Boolean fields use the text `true` or `false`. Let the client generate the multipart boundary. |

Choose one body format per request. Send a nonempty body and use each field name once, including in JSON objects. Do not wrap the fields inside a `data` or `shortcut` object. `sharing_url`, `version`, and `changelog` are the canonical update fields; `link` and `changes` are not accepted aliases. Creating a shortcut uses `auto_submit`, not `submit`.

JSON `description` and `category` may also be `null` to clear those optional values. Other text fields must be strings. JSON booleans must be actual `true` / `false` values, not strings, numbers, or `null`. Form boolean text is case-insensitive and surrounding whitespace is ignored; use lowercase `true` / `false` for consistency. Shortcut creation and update bodies accept text fields, not file uploads.

Do not manually set `Content-Type: application/json` when sending a Form body. Curl sets the appropriate Form content type for `--data-urlencode` or `--form-string`.

### Apple Shortcuts setup

In **Get Contents of URL**:

1. Set the URL to `https://shortcuty.app/api/v1/shortcuts/<uuid>/update` for an update, replacing `<uuid>` with the Shortcuty shortcut UUID. To create a shortcut, use `https://shortcuty.app/api/v1/shortcuts`.
2. Set **Method** to **POST**.
3. Add the header **Authorization** with the value `Bearer YOUR_API_KEY`, replacing the placeholder with your own key. Keep the key private.
4. Choose **Request Body → JSON** or **Request Body → Form**.
5. For an update, add `sharing_url`, `version`, and `changelog` as **Text** fields. The `sharing_url` value is the new iCloud shortcut link; `version` is text such as `1.2.0`.
6. For creation, add `sharing_url` as **Text** and any optional creation fields. For boolean fields, use **Boolean** values in JSON or the **Text** `true` / `false` in Form.

For Form, let Shortcuts set the content type and multipart boundary automatically. For JSON, use a JSON body rather than placing JSON text in a Form field.

The separate [screenshot upload endpoint](#upload-screenshot) still uses multipart Form with a file. The body formats above apply to shortcut creation and updates.

---

## Categories

### Get Categories

**GET** `/api/v1/categories`

Returns all available categories. Public endpoint (no authentication required).

**Response (200 OK):**

```json
{
  "categories": [
    "Artificial Intelligence",
    "Education",
    "Entertainment",
    "Finance",
    "Food & Dining",
    "Gaming",
    "Health & Fitness",
    "Music",
    "News",
    "Photography",
    "Productivity",
    "Shopping",
    "Shortcut Extension",
    "Sideloading",
    "Social Media",
    "Sports",
    "Technology",
    "Travel",
    "Utilities",
    "Weather"
  ]
}
```

---

## Shortcuts

### Create Shortcut

**POST** `/api/v1/shortcuts`

Creates a new shortcut (draft or submitted).

**Request body:** JSON or Form. See [Request body formats](#request-body-formats).

| Field | Type | Required | Description |
|---|---|---|---|
| `sharing_url` | String | Yes | iCloud sharing URL beginning with `https://www.icloud.com/shortcuts/`. |
| `description` | String | No | Shortcut description, up to 500,000 characters. |
| `category` | String | No | A value from [Get Categories](#get-categories). |
| `updater_type` | String | No | `shortcuty`, `third_party`, or `none` (default: `none`). |
| `requires_ios26_ai` | Boolean | No | Whether the shortcut requires iOS 26 AI features (default: `false`). |
| `auto_submit` | Boolean | No | Submit immediately for review and scanning (default: `false`). |
| `visibility` | String | No | `public` or `private` (default: `public`). |

The shortcut name is extracted from the iCloud shortcut. Do not send `name` or `submit`; use `auto_submit` for immediate submission.

JSON example:

```json
{
  "sharing_url": "https://www.icloud.com/shortcuts/REPLACE_WITH_ICLOUD_SHARE_ID",
  "description": "Shortcut description",
  "category": "Productivity",
  "updater_type": "none",
  "requires_ios26_ai": false,
  "auto_submit": false,
  "visibility": "public"
}
```

Form example (URL-encoded):

```sh
curl --request POST 'https://shortcuty.app/api/v1/shortcuts' \
  --header 'Authorization: Bearer YOUR_API_KEY' \
  --data-urlencode 'sharing_url=https://www.icloud.com/shortcuts/REPLACE_WITH_ICLOUD_SHARE_ID' \
  --data-urlencode 'description=Shortcut description' \
  --data-urlencode 'category=Productivity' \
  --data-urlencode 'requires_ios26_ai=false' \
  --data-urlencode 'auto_submit=false'
```

**Response (201 Created):**

```json
{
  "message": "Draft shortcut created",
  "shortcut": {
    "uuid": "123e4567-e89b-12d3-a456-426614174000",
    "name": "Shortcut Name",
    "description": "Shortcut description",
    "sharing_url": "https://www.icloud.com/shortcuts/...",
    "version": "1.0.0",
    "status": "draft",
    "category": "Productivity",
    "created_at": "2024-01-15T10:30:00Z",
    "updated_at": "2024-01-15T10:30:00Z",
    "requires_ios26_ai": false,
    "updater_type": "shortcuty"
  }
}
```

---

### Get My Shortcuts

**GET** `/api/v1/shortcuts/my?page=1&per_page=20`

Returns shortcuts owned by the authenticated API key's account, including drafts, pending initial submissions, and rejected submissions. A pending update on an approved shortcut appears in [history](#get-shortcut-history); it does not replace the approved shortcut in this list.

**Query Parameters:**

- `page` (optional, default: 1)

- `per_page` (optional, default: 20)

**Response (200 OK):**

```json
{
  "shortcuts": [
    {
      "uuid": "...",
      "name": "...",
      "description": "...",
      "sharing_url": "...",
      "version": "...",
      "status": "approved",
      "category": "...",
      "created_at": "...",
      "updated_at": "...",
      "requires_ios26_ai": false,
      "updater_type": "..."
    }
  ],
  "total": 10,
  "pages": 1,
  "current_page": 1
}
```

---

### Get Shortcut

**GET** `/api/v1/shortcuts/<uuid>`

Returns a single shortcut by UUID. The owner can read an initial submission while it is draft, pending, or rejected. Other accounts cannot read those non-approved submissions. For an approved shortcut with a pending update, this endpoint returns the approved base version; use owner-only [history](#get-shortcut-history) to inspect the proposal.

**Response (200 OK):**

```json
{
  "shortcut": {
    "uuid": "...",
    "name": "...",
    "description": "...",
    "sharing_url": "...",
    "version": "...",
    "status": "...",
    "category": "...",
    "created_at": "...",
    "updated_at": "...",
    "requires_ios26_ai": false,
    "updater_type": "...",
    "rejection_reason": "..."
  }
}
```

---

### Update Shortcut

**POST** `/api/v1/shortcuts/<uuid>/update`

Updates a shortcut owned by the authenticated API key's account. Draft changes apply immediately. Changes to submitted shortcuts use the review and scanning flow as applicable.

**Request body:** JSON or Form. See [Request body formats](#request-body-formats). All fields below are optional, but the request must make a change. For a draft or a new update, send the fields you want to change. To replace an existing pending update, include all changes you still want, as explained below.

| Field | Type | Description |
|---|---|---|
| `sharing_url` | String | Updated iCloud sharing URL. Use this exact name, not `link`. |
| `description` | String | Updated description, up to 500,000 characters. |
| `category` | String | A value from [Get Categories](#get-categories). |
| `requires_ios26_ai` | Boolean | Whether the shortcut requires iOS 26 AI features. |
| `updater_type` | String | `shortcuty`, `third_party`, or `none`. |
| `version` | String | Version text such as `"1.2.0"`, up to 50 characters. JSON numbers are not accepted. |
| `changelog` | String | Release notes, up to 10,000 characters. Use this exact name, not `changes`. |
| `visibility` | String | `public` or `private`. |

JSON example:

```sh
curl --request POST 'https://shortcuty.app/api/v1/shortcuts/REPLACE_WITH_SHORTCUT_UUID/update' \
  --header 'Authorization: Bearer YOUR_API_KEY' \
  --header 'Content-Type: application/json' \
  --data '{"sharing_url":"https://www.icloud.com/shortcuts/REPLACE_WITH_ICLOUD_SHARE_ID","version":"1.2.0","changelog":"Bug fixes and improvements"}'
```

Form example (URL-encoded):

```sh
curl --request POST 'https://shortcuty.app/api/v1/shortcuts/REPLACE_WITH_SHORTCUT_UUID/update' \
  --header 'Authorization: Bearer YOUR_API_KEY' \
  --data-urlencode 'sharing_url=https://www.icloud.com/shortcuts/REPLACE_WITH_ICLOUD_SHARE_ID' \
  --data-urlencode 'version=1.2.0' \
  --data-urlencode 'changelog=Bug fixes and improvements'
```

Form example (multipart):

```sh
curl --request POST 'https://shortcuty.app/api/v1/shortcuts/REPLACE_WITH_SHORTCUT_UUID/update' \
  --header 'Authorization: Bearer YOUR_API_KEY' \
  --form-string 'sharing_url=https://www.icloud.com/shortcuts/REPLACE_WITH_ICLOUD_SHARE_ID' \
  --form-string 'version=1.2.0' \
  --form-string 'changelog=Bug fixes and improvements'
```

Use the Shortcuty shortcut UUID in the endpoint and the iCloud share ID in `sharing_url`. These are different identifiers. A request with no effective changes returns `400`.

### Replacing a pending update

Send another request to the same update endpoint to replace your current pending proposal. The new request replaces the **entire pending proposal**, rather than merging into it: fields and release notes omitted from the new request are not carried over. Include every unapproved change and the changelog you still want.

For example, if the pending proposal changes both `sharing_url` and `version`, sending only a new `changelog` drops those proposed URL and version changes. Send all three fields to retain them.

While a proposal remains pending, its unapproved changes do not replace the approved shortcut's live version. Draft edits, visibility changes, and automatically approved edits can apply immediately; read the response and history to see the outcome.

### Understanding successful responses

| Request | Status | Meaning |
|---|---|---|
| Create | `201` | A shortcut was created: a draft by default, or submitted when `auto_submit` is `true`. This does not guarantee publication. |
| Update | `200` | Changes were applied immediately, an update was automatically approved, or an initial pending submission was edited. Read `message`; an edited pending submission is still pending. |
| Update | `201` | An update was accepted for processing, scanning, or manual review. This does not guarantee approval or publication. |
| Submit draft | `200` / `202` | Submitted for review; `202` means scan admission is deferred. This does not guarantee approval or publication. |

Responses include a `message`; update-record responses also include `update_id`. When present, `scan_queued: true` means scan admission succeeded, and `false` means admission was deferred for automatic recovery. Confirm approval in the owner's history or initial-submission status.

Example of an accepted update:

```json
{
  "message": "Update queued for processing and scan",
  "scan_queued": true,
  "update_id": 123
}
```

---

### Submit Draft Shortcut

**POST** `/api/v1/shortcuts/<uuid>/submit`

Submits an owned draft or rejected shortcut for review. No request body is required. Returns `200` when scanning is admitted, or `202` when scan admission is deferred; the submission remains pending either way.

**Response (200 OK; 202 if deferred):**

```json
{
  "message": "Shortcut queued for review and scanning",
  "scan_queued": true
}
```

---

### Get Shortcut History

**GET** `/api/v1/shortcuts/<uuid>/history`

Returns version history, including pending and rejected updates, for the shortcut owner only. A key belonging to another account cannot read this history, including an admin account's key. Non-owners receive `404`.

An initial submission is tracked on the shortcut itself; check `/shortcuts/my` or `/shortcuts/<uuid>` for its status. A pending update to an approved shortcut appears here while the list and detail endpoints continue to show the approved base version.

**Response (200 OK):**

```json
{
  "shortcut_name": "My Shortcut",
  "shortcut_uuid": "...",
  "changelog": [
    {
      "version": "1.2.0",
      "changelog": "What's new",
      "date": "2024-01-20T14:22:00Z",
      "status": "approved",
      "sharing_url": "https://...",
      "changes": {
        "description": "Updated description"
      }
    }
  ]
}
```

---

## Screenshots

### Upload Screenshot

**POST** `/api/v1/upload/screenshot`

Uploads a screenshot for a shortcut.

**Content-Type:** `multipart/form-data`

**Form Data:**

- `file` (required): Image file (PNG, JPEG, GIF)

- `shortcut_id` (required): Shortcut UUID

**Limits:**

- Maximum 5 screenshots per shortcut

- Valid file extensions: `.png`, `.jpg`, `.jpeg`, `.gif`

- File content is validated

**Response (200 OK):**

```json
{
  "screenshot": {
    "id": 123,
    "shortcut_id": 456,
    "filename": "screenshots/2025/12/uuid.png",
    "url": "https://shortcuty.app/uploads/screenshots/2025/12/uuid.png",
    "order": 0,
    "created_at": "2024-01-15T10:30:00Z"
  }
}
```

---

### Delete Screenshot

**DELETE** `/api/v1/shortcuts/<uuid>/screenshots/<screenshot_id>`

Deletes a screenshot for a shortcut.

**Response (200 OK):**

```json
{
  "message": "Screenshot deleted successfully"
}
```

---

## Response Fields

### Shortcut Object

```json
{
  "uuid": "string",
  "name": "string",
  "description": "string | null",
  "sharing_url": "string",
  "version": "string | null",
  "status": "draft | pending | approved | rejected | delisted",
  "category": "string | null",
  "created_at": "string | null (ISO 8601)",
  "updated_at": "string | null (ISO 8601)",
  "requires_ios26_ai": "boolean",
  "updater_type": "shortcuty | third_party | none",
  "visibility": "public | private",
  "rejection_reason": "string (only if status is 'rejected')"
}
```

---

## Errors and retries

Errors are JSON objects with an `error` message, for example:

```json
{
  "error": "Field version must be a string"
}
```

For shortcut creation and updates:

| Status | Meaning |
|---|---|
| `400 Bad Request` | Empty or malformed body; JSON that is not a nonempty object; duplicate fields; file parts; invalid types, booleans, names, or values; or an update with no changes. Correct the request before sending it again. |
| `401 Unauthorized` | Missing or invalid API key. |
| `403 Forbidden` | The account is inactive, or the API key's account does not own the shortcut being changed. |
| `404 Not Found` | The shortcut does not exist, or the caller cannot read the requested non-approved shortcut or owner-only history. |
| `413 Request Entity Too Large` | The request exceeds the server's body size limit. |
| `415 Unsupported Media Type` | Missing or unsupported `Content-Type`. Use JSON, URL-encoded Form, or multipart Form. |
| `500 Internal Server Error` | The request failed on the server. It may already have saved a submission or update; check before retrying. |

Field names are exact. `name` is read-only; `link`, `changes`, and `submit` are not accepted request fields. Use `sharing_url`, `changelog`, and creation's `auto_submit`. Duplicate fields return `Duplicate field: <field>`, even when both values match. Versions longer than 50 characters return `Version must be 50 characters or less`.

**After a `500` or an uncertain result, do not automatically repeat the POST.** First inspect the owner's **GET** `/api/v1/shortcuts/my`, **GET** `/api/v1/shortcuts/<uuid>`, and **GET** `/api/v1/shortcuts/<uuid>/history` as applicable. An initial submission may already exist, and an update may already be pending or approved. Confirm the saved state before a manual retry; a repeated update can replace the current pending proposal.
