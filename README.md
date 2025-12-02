# Shortcuty Creator API Documentation

Manage your shortcuts programmatically with the Creator API. Create, update, and manage your shortcuts.

## Base URL

```
https://shortcuty.app/api/v1
```

## Getting Started

All v1 API endpoints require API key authentication. Get your API key from your account settings on the Shortcuty website (not available in the iOS app).

## Authentication

Include your API key in the `Authorization` header for all requests:

```
Authorization: Bearer <your-api-key>
```

To get an API key, visit your Account Settings on the Shortcuty website (https://shortcuty.app) and create one in the API Key Management section. **Note:** API keys can only be created and managed on the website, not in the iOS app.

---

## API Endpoints

### GET /api/v1/categories

**Authentication:** PUBLIC (No API key required)

Get a list of all available shortcut categories.

#### Response

**Status Code:** `200 OK`

**Response Schema:**

```json
{
  "categories": [
    "string"
  ]
}
```

**Response Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `categories` | `array[string]` | Array of category names |

**Example Response:**

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

**Example Request:**

```bash
curl -X GET "https://shortcuty.app/api/v1/categories"
```

**Notes:**

- No API key required; this is a public endpoint
- Use these category names when creating or updating shortcuts

---

### POST /api/v1/shortcuts

**Authentication:** API KEY (Required)

Create a new shortcut (draft) with optional immediate submission.

#### Request Body

**Content-Type:** `application/json`

**Request Schema:**

```json
{
  "sharing_url": "string (required)",
  "description": "string (optional)",
  "category": "string (optional)",
  "requires_ios26_ai": "boolean (optional)",
  "updater_type": "string (optional)",
  "submit": "boolean (optional)"
}
```

**Request Fields:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `sharing_url` | `string` | Yes | iCloud sharing URL |
| `description` | `string` | No | Description (max 500,000 chars) |
| `category` | `string` | No | Category name |
| `requires_ios26_ai` | `boolean` | No | Whether shortcut requires iOS 26 AI features |
| `updater_type` | `string` | No | One of `"shortcuty"`, `"third_party"`, or `"none"` (default: `"none"`) |
| `submit` | `boolean` | No | If `true`, creates and immediately submits for review |

#### Response

**Status Code:** `201 Created`

**Response Schema:**

```json
{
  "message": "string",
  "shortcut": {
    "id": "integer",
    "uuid": "string (UUID)",
    "name": "string",
    "status": "string",
    "created_at": "string (ISO 8601)"
  }
}
```

**Response Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `message` | `string` | Success message |
| `shortcut` | `object` | Created shortcut object |
| `shortcut.id` | `integer` | Shortcut ID |
| `shortcut.uuid` | `string` | Shortcut UUID |
| `shortcut.name` | `string` | Shortcut name |
| `shortcut.status` | `string` | Shortcut status (e.g., `"draft"`) |
| `shortcut.created_at` | `string` | Creation timestamp in ISO 8601 format (UTC) |

**Example Response:**

```json
{
  "message": "Draft shortcut created",
  "shortcut": {
    "id": 123,
    "uuid": "550e8400-e29b-41d4-a716-446655440000",
    "name": "Untitled Shortcut",
    "status": "draft",
    "created_at": "2025-12-01T10:00:00Z"
  }
}
```

**Example Request:**

```bash
curl -X POST https://shortcuty.app/api/v1/shortcuts \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "sharing_url": "https://www.icloud.com/shortcuts/abc123",
    "description": "My awesome shortcut",
    "category": "productivity"
  }'
```

---

### GET /api/v1/shortcuts/my

**Authentication:** API KEY (Required)

Get paginated list of your shortcuts.

#### Query Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `page` | `integer` | No | `1` | Page number |
| `per_page` | `integer` | No | `20` | Items per page |

#### Response

**Status Code:** `200 OK`

**Response Schema:**

```json
{
  "shortcuts": ["array of shortcut objects"],
  "total": "integer",
  "pages": "integer",
  "current_page": "integer"
}
```

**Response Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `shortcuts` | `array[object]` | Array of shortcut objects |
| `total` | `integer` | Total number of shortcuts |
| `pages` | `integer` | Total number of pages |
| `current_page` | `integer` | Current page number |

**Example Response:**

```json
{
  "shortcuts": [...],
  "total": 25,
  "pages": 2,
  "current_page": 1
}
```

**Example Request:**

```bash
curl -X GET "https://shortcuty.app/api/v1/shortcuts/my?page=1&per_page=20" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

---

### GET /api/v1/shortcuts/{uuid}

**Authentication:** API KEY (Required)

Get shortcut details by UUID, including comments, screenshots, and latest update.

#### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `uuid` | `string` | Yes | Shortcut UUID |

#### Response

**Status Code:** `200 OK`

**Response Schema:**

```json
{
  "id": "integer",
  "uuid": "string (UUID)",
  "name": "string",
  "status": "string",
  "created_at": "string (ISO 8601)",
  "author": "string",
  "author_profile_picture": "string (URL)",
  "category": "string",
  "description": "string",
  "downloads": "integer",
  "icon_filename": "string | null",
  "likes_count": "integer",
  "version": "string",
  "sharing_url": "string (URL)",
  "requires_ios26_ai": "boolean",
  "updater_type": "string",
  "comments": ["array"],
  "screenshots": ["array"],
  "latest_update": "object | null"
}
```

**Response Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `id` | `integer` | Shortcut ID |
| `uuid` | `string` | Shortcut UUID |
| `name` | `string` | Shortcut name |
| `status` | `string` | Shortcut status (e.g., `"draft"`, `"pending"`, `"approved"`, `"rejected"`) |
| `created_at` | `string` | Creation timestamp in ISO 8601 format (UTC) |
| `author` | `string` | Author name |
| `author_profile_picture` | `string` | Full URL to author's profile picture |
| `category` | `string` | Category name |
| `description` | `string` | Shortcut description |
| `downloads` | `integer` | Number of downloads |
| `icon_filename` | `string \| null` | Icon filename or null |
| `likes_count` | `integer` | Number of likes |
| `version` | `string` | Version string |
| `sharing_url` | `string` | iCloud sharing URL |
| `requires_ios26_ai` | `boolean` | Whether shortcut requires iOS 26 AI features |
| `updater_type` | `string` | Updater type (`"shortcuty"`, `"third_party"`, or `"none"`) |
| `comments` | `array` | Array of comments |
| `screenshots` | `array` | Array of screenshots |
| `latest_update` | `object \| null` | Latest update object or null |

**Example Response:**

```json
{
  "id": 123,
  "uuid": "550e8400-e29b-41d4-a716-446655440000",
  "name": "Example Shortcut",
  "status": "draft",
  "created_at": "2025-12-01T10:00:00Z",
  "author": "Example User",
  "author_profile_picture": "https://shortcuty.app/uploads/profiles/example.png",
  "category": "Artificial Intelligence",
  "description": "This is an example shortcut description that demonstrates the response structure...",
  "downloads": 0,
  "icon_filename": null,
  "likes_count": 0,
  "version": "1.0",
  "sharing_url": "https://www.icloud.com/shortcuts/550e8400e4a646aea502f3cf06a97e44",
  "requires_ios26_ai": false,
  "updater_type": "none",
  "comments": [],
  "screenshots": [],
  "latest_update": null
}
```

**Example Request:**

```bash
curl -X GET "https://shortcuty.app/api/v1/shortcuts/550e8400-e29b-41d4-a716-446655440000" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

**Notes:**

- Users can only view their own draft/pending/rejected shortcuts

---

### GET /api/v1/shortcuts/{shortcut_identifier}/history

**Authentication:** API KEY (Required)

Get version history for a shortcut, including changelogs, status changes, and field updates.

#### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `shortcut_identifier` | `string` | Yes | Shortcut UUID or ID |

#### Response

**Status Code:** `200 OK`

**Response Schema:**

```json
{
  "changelog": [
    {
      "version": "string",
      "changelog": "string",
      "date": "string (ISO 8601)",
      "status": "string",
      "sharing_url": "string (URL)",
      "changes": {
        "name": "string",
        "new_version": "string",
        "old_sharing_url": "string (URL)"
      }
    }
  ],
  "shortcut_name": "string",
  "shortcut_uuid": "string (UUID)"
}
```

**Response Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `changelog` | `array[object]` | Array of history entries |
| `changelog[].version` | `string` | Version string (e.g., `"2.0"`) |
| `changelog[].changelog` | `string` | Changelog text for this version |
| `changelog[].date` | `string` | Timestamp in ISO 8601 format (UTC) |
| `changelog[].status` | `string` | Status at this point in history (e.g., `"approved"`, `"pending"`, `"draft"`) |
| `changelog[].sharing_url` | `string` | iCloud sharing URL for this version (convenience field) |
| `changelog[].changes` | `object` | Object containing field changes |
| `changelog[].changes.name` | `string` | Updated name (if changed) |
| `changelog[].changes.new_version` | `string` | New version string (if changed) |
| `changelog[].changes.old_sharing_url` | `string` | Previous sharing URL |
| `shortcut_name` | `string` | The name of the shortcut |
| `shortcut_uuid` | `string` | The UUID of the shortcut |

**Example Response:**

```json
{
  "changelog": [
    {
      "version": "2.0",
      "changelog": "Updated version",
      "date": "2025-12-01T10:00:00Z",
      "status": "approved",
      "sharing_url": "https://www.icloud.com/shortcuts/...",
      "changes": {
        "name": "Updated Name",
        "new_version": "2.0",
        "old_sharing_url": "https://www.icloud.com/shortcuts/old-url"
      }
    }
  ],
  "shortcut_name": "Untitled Shortcut",
  "shortcut_uuid": "a127b74e-9cac-4193-80d4-97f1066eb5f2"
}
```

**Example Request:**

```bash
curl -X GET "https://shortcuty.app/api/v1/shortcuts/550e8400-e29b-41d4-a716-446655440000/history" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

**Notes:**

- The response is an object with a `changelog` array containing history entries, along with `shortcut_name` and `shortcut_uuid` metadata
- In the `changes` object within each history entry, the field `old_sharing_url` represents the previous sharing URL
- Users can only view history for their own shortcuts

---

### POST /api/v1/shortcuts/{uuid}/submit

**Authentication:** API KEY (Required)

Submit a draft or rejected shortcut for review.

#### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `uuid` | `string` | Yes | Shortcut UUID |

#### Request Body

No body required.

#### Response

**Status Code:** `200 OK`

**Response Schema:**

```json
{
  "message": "string"
}
```

**Response Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `message` | `string` | Success message |

**Example Response:**

```json
{
  "message": "Shortcut queued for review and scanning"
}
```

**Example Request:**

```bash
curl -X POST "https://shortcuty.app/api/v1/shortcuts/550e8400-e29b-41d4-a716-446655440000/submit" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

---

### POST /api/v1/shortcuts/{uuid}/update

**Authentication:** API KEY (Required)

Update an existing shortcut. For drafts, updates are applied immediately. For approved shortcuts, creates an update record for admin review.

#### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `uuid` | `string` | Yes | Shortcut UUID |

#### Request Body

**Content-Type:** `application/json`

**Request Schema:**

```json
{
  "name": "string (optional)",
  "description": "string (optional)",
  "sharing_url": "string (optional)",
  "category": "string (optional)",
  "requires_ios26_ai": "boolean (optional)",
  "updater_type": "string (optional)",
  "new_version": "string (optional)",
  "new_sharing_url": "string (optional)",
  "changelog": "string (optional)"
}
```

**Request Fields:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | `string` | No | Shortcut name |
| `description` | `string` | No | Description (max 500,000 chars) |
| `sharing_url` | `string` | No | New sharing URL (must be iCloud URL) |
| `category` | `string` | No | Category name |
| `requires_ios26_ai` | `boolean` | No | Whether shortcut requires iOS 26 AI features |
| `updater_type` | `string` | No | One of `"shortcuty"`, `"third_party"`, or `"none"` |
| `new_version` | `string` | No | Version string (e.g., `"2.0"`) |
| `new_sharing_url` | `string` | No | Sharing URL for new version (required if `new_version` provided) |
| `changelog` | `string` | No | Changelog text (max 10,000 chars) |

#### Response

**Status Code:** `200 OK` (Draft - Immediate Update) or `201 Created` (Update Submitted for Review)

**Response Schema (Draft Update):**

```json
{
  "message": "string"
}
```

**Response Schema (Update Submitted):**

```json
{
  "message": "string",
  "update_id": "integer"
}
```

**Response Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `message` | `string` | Success message |
| `update_id` | `integer` | Update ID (only present when update is submitted for review) |

**Example Response (Draft - Immediate Update):**

```json
{
  "message": "Draft shortcut updated"
}
```

**Example Response (Update Submitted for Review):**

```json
{
  "message": "Update queued for processing and scan",
  "update_id": 123
}
```

**Example Request:**

```bash
curl -X POST "https://shortcuty.app/api/v1/shortcuts/550e8400-e29b-41d4-a716-446655440000/update" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Updated Shortcut Name",
    "description": "New description"
  }'
```

**Notes:**

- Draft shortcuts are updated immediately
- Approved shortcuts create update records for admin review
- If URL or description changes, update requires scanning and admin review

---

### POST /api/v1/upload/screenshot

**Authentication:** API KEY (Required)

Upload a screenshot for a shortcut.

#### Request Body

**Content-Type:** `multipart/form-data`

**Form Data:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `file` | `file` | Yes | Image file (PNG, JPG, JPEG, GIF, SVG, WEBP) |
| `shortcut_id` | `string` | Yes | Shortcut UUID |

#### Response

**Status Code:** `200 OK`

**Response Schema:**

```json
{
  "screenshot": {
    "id": "integer",
    "filename": "string",
    "url": "string (URL)",
    "uploaded_at": "string (ISO 8601)"
  }
}
```

**Response Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `screenshot` | `object` | Screenshot object |
| `screenshot.id` | `integer` | Screenshot ID |
| `screenshot.filename` | `string` | Screenshot filename |
| `screenshot.url` | `string` | Full URL to the screenshot |
| `screenshot.uploaded_at` | `string` | Upload timestamp in ISO 8601 format (UTC) |

**Example Response:**

```json
{
  "screenshot": {
    "id": 789,
    "filename": "security/2025/12/3ffd4817-bff5-4cdd-b6a2-7be82a61441c.png",
    "url": "https://shortcuty.app/uploads/security/2025/12/3ffd4817-bff5-4cdd-b6a2-7be82a61441c.png",
    "uploaded_at": "2025-12-01T10:30:00Z"
  }
}
```

**Example Request:**

```bash
curl -X POST "https://shortcuty.app/api/v1/upload/screenshot" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -F "file=@screenshot.png" \
  -F "shortcut_id=550e8400-e29b-41d4-a716-446655440000"
```

**Notes:**

- Only the shortcut author can upload screenshots
- File content is validated for security
- Supported formats: PNG, JPG, JPEG, GIF, SVG, WEBP

---

## Error Codes

Common HTTP status codes returned by the API.

| Status Code | Description |
|-------------|-------------|
| `200 OK` | Request successful |
| `201 Created` | Resource created successfully |
| `400 Bad Request` | Invalid request data, missing required fields, or validation errors |
| `401 Unauthorized` | Missing or invalid API key |
| `403 Forbidden` | Permission denied (not the resource owner) |
| `404 Not Found` | Resource not found or user doesn't have permission to view |
| `409 Conflict` | Resource conflict (e.g., trying to create a second API key) |
| `500 Internal Server Error` | Server error |

---

## Important Notes

- **Shortcut identifiers must be UUIDs** - When using shortcut identifiers in endpoints, use the UUID format
- **Users can only access/modify their own shortcuts** - No admin privileges via API key
- **Draft shortcuts can be updated immediately** - Approved shortcuts require admin review for updates
- **All timestamps are in ISO 8601 format (UTC)** - All date/time fields use ISO 8601 format in UTC timezone
- **Screenshot and profile picture URLs are always full URLs** - Not relative paths
- **Sharing URLs must be valid iCloud shortcut sharing URLs** - Format: `https://www.icloud.com/shortcuts/...`

