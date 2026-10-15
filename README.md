# V1 API Documentation



## Overview

Welcome to the Shortcuty V1 API documentation. This API provides developers with programmatic access to manage iOS Shortcuts, enabling you to create, update, submit, and track shortcuts through their lifecycle.

### What You Can Do

With the V1 API, you can:

- **Create shortcuts** as drafts or submit them immediately for review
- **Manage your shortcuts** by retrieving, updating, and tracking their status
- **Submit drafts** for review and approval
- **Track version history** with changelogs and version information
- **Categorize shortcuts** across 20+ predefined categories
- **Configure update behavior** with different updater types (Shortcuty, third-party, or none)

### Submission flow

Shortcuts follow a simple submission flow:

1. **Draft** → Create a shortcut as a draft to work on it before submission
2. **Pending** → Submit the draft for review and security scanning
3. **Approved** → Once approved, the shortcut is live and can be updated
4. **Rejected** → If rejected, review the rejection reason and update accordingly

### Getting Started

**Base URL:** `https://shortcuty.app/api/v1`

> **💡 Using the CLI?** Check out the [Shortcuty CLI](https://github.com/Shortcuty/Shortcuty-CLI) for a command-line interface to interact with the API. It provides a convenient way to manage shortcuts without writing HTTP requests.

You can obtain your API key from [browse.shortcuty.app](https://browse.shortcuty.app) in your account settings. Include your API key in the `Authorization` header for every request. Once authenticated, you can start creating and managing shortcuts using the endpoints documented below.



## Authentication

Most endpoints require API key authentication:

```

Authorization: Bearer <your_api_key>

```



---



## Request body formats

> Rollout status: JSON and Form compatibility is prepared; production deployment confirmation is pending. This branch documents the upcoming contract.

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



Returns shortcuts owned by the authenticated user.



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

      "updater_type": "...",

      "rejection_reason": "..."  // Only if status is "rejected"

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



Returns a single shortcut by UUID.



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

    "rejection_reason": "..."  // Only if status is "rejected"

  }

}

```



---



### Update Shortcut

**POST** `/api/v1/shortcuts/<uuid>/update`



Updates a shortcut owned by the authenticated API key's account. Draft changes apply immediately. Changes to submitted shortcuts use the review and scanning flow as applicable.

**Request body:** JSON or Form. See [Request body formats](#request-body-formats). Send only the fields you want to change.

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

**Response (200/201):**

```json

{

  "message": "Draft shortcut updated"  // or "Update queued for processing and scan"

}

```



---



### Submit Draft Shortcut

**POST** `/api/v1/shortcuts/<uuid>/submit`



Submits a draft shortcut for review.



**Response (200 OK):**

```json

{

  "message": "Shortcut queued for review and scanning"

}

```



---



### Get Shortcut History

**GET** `/api/v1/shortcuts/<uuid>/history`



Returns version history for a shortcut (owner only).



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

  "status": "draft | pending | approved | rejected",

  "category": "string | null",

  "created_at": "string | null (ISO 8601)",

  "updated_at": "string | null (ISO 8601)",

  "requires_ios26_ai": "boolean",

  "updater_type": "shortcuty | third_party | none",

  "rejection_reason": "string (only if status is 'rejected')"

}

```



---


### Error Responses



Errors are JSON objects with an `error` message. Authentication and ownership checks still apply to every body format.

For shortcut creation and updates:

| Status | Cause |
|---|---|
| `400 Bad Request` | Empty or malformed body; JSON that is not a nonempty object; duplicate body fields; file parts; invalid field types, boolean values, field names, or field values; or an update with no changes. |
| `401 Unauthorized` | Missing or invalid API key. |
| `403 Forbidden` | The API key's account does not own the shortcut. |
| `404 Not Found` | The shortcut does not exist. |
| `413 Request Entity Too Large` | The request exceeds the server's body size limit. |
| `415 Unsupported Media Type` | Missing or unsupported request `Content-Type`. Use JSON, URL-encoded Form, or multipart Form. |

Versions longer than 50 characters return `400` with `Version must be 50 characters or less`.

For example, a numeric JSON version is rejected with `400`:

```json
{
  "error": "Field version must be a string"
}
```

Repeated fields are rejected rather than choosing one value, even when the repeated values match:

```json
{
  "error": "Duplicate field: version"
}
```

An unsupported content type returns `415`:

```json
{
  "error": "Content-Type must be application/json, application/x-www-form-urlencoded, or multipart/form-data"
}
```

**400 Bad Request:**

```json

{

  "error": "Field 'name' is read-only and cannot be set"

}

```



```json

{

  "error": "Unknown field(s): submit. Allowed fields: auto_submit, category, description, requires_ios26_ai, sharing_url, updater_type, visibility"

}

```



```json

{

  "error": "Sharing URL must be an iCloud sharing URL (https://www.icloud.com/shortcuts/...)"

}

```



**401 Unauthorized:**

```json

{

  "error": "Invalid API key"

}

```



**403 Forbidden:**

```json

{

  "error": "Permission denied"

}

```



**404 Not Found:**

```json

{

  "error": "Shortcut not found"

}

```



---
