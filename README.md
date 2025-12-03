# Shortcuty API v1 Documentation

## Base URL

All v1 API endpoints are prefixed with `/api/v1`

## Authentication

Most v1 endpoints require API key authentication using the `Authorization` header:

```
Authorization: Bearer <your-api-key>
```

API keys are UUID v4 format strings. Get your API key from the `/api/auth/api-key` endpoint (requires user authentication).

**Note:** The `/categories` endpoint is public and does not require authentication.

---

## Endpoints

### Categories

#### GET `/api/v1/categories`

Get all available categories (public endpoint, no authentication required).

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

### Shortcuts

#### POST `/api/v1/shortcuts`

Create a new shortcut (draft) with optional immediate submission.

**Authentication:** Required

**Request Body:**

```json
{
  "sharing_url": "https://www.icloud.com/shortcuts/...",
  "description": "Shortcut description (optional, max 500,000 characters)",
  "category": "Productivity",
  "requires_ios26_ai": false,
  "updater_type": "none",
  "submit": false
}
```

**Fields:**

- `sharing_url` (required): iCloud sharing URL (must start with `https://www.icloud.com/shortcuts/` or `https://icloud.com/shortcuts/`)
- `description` (optional): Description (max 500,000 characters)
- `category` (optional): One of the preset categories
- `requires_ios26_ai` (optional): Boolean, default `false`
- `updater_type` (optional): One of `"shortcuty"`, `"third_party"`, or `"none"` (default: `"none"`)
- `submit` (optional): If `true`, immediately submits for review; if `false`, creates as draft (default: `false`)

**Response (201 Created):**

```json
{
  "message": "Draft shortcut created" | "Shortcut created and queued for review and scanning",
  "shortcut": {
    "uuid": "550e8400-e29b-41d4-a716-446655440000",
    "name": "Untitled Shortcut" | "Extracted Name",
    "description": "Shortcut description" | null,
    "sharing_url": "https://www.icloud.com/shortcuts/...",
    "icon_filename": "path/to/icon.png" | null,
    "version": "1.0" | null,
    "status": "draft" | "pending",
    "created_at": "2025-01-15T10:30:00Z",
    "updated_at": "2025-01-15T10:30:00Z",
    "category": "Productivity" | null,
    "downloads": 0,
    "likes_count": 0,
    "requires_ios26_ai": false,
    "updater_type": "none" | "shortcuty" | "third_party",
    "author": "username",
    "author_profile_picture": "https://shortcuty.app/uploads/profiles/2025/12/..." | null
  }
}
```

**Note:** The `name` field is extracted automatically during scanning and will initially be "Untitled Shortcut" for drafts.

---

#### GET `/api/v1/shortcuts/my`

Get current user's shortcuts.

**Authentication:** Required

**Query Parameters:**

- `page` (optional): Page number (default: 1)
- `per_page` (optional): Items per page (default: 20)

**Response (200 OK):**

```json
{
  "shortcuts": [
    {
      "uuid": "550e8400-e29b-41d4-a716-446655440000",
      "name": "My Shortcut",
      "description": "Description" | null,
      "sharing_url": "https://www.icloud.com/shortcuts/...",
      "icon_filename": "path/to/icon.png" | null,
      "version": "1.0" | null,
      "status": "draft" | "pending" | "approved" | "rejected",
      "created_at": "2025-01-15T10:30:00Z",
      "updated_at": "2025-01-15T10:30:00Z",
      "category": "Productivity" | null,
      "downloads": 0,
      "likes_count": 0,
      "requires_ios26_ai": false,
      "updater_type": "none" | "shortcuty" | "third_party",
      "author": "username",
      "author_profile_picture": "https://shortcuty.app/uploads/profiles/2025/12/..." | null,
      "rejection_reason": "Reason for rejection" | null
    }
  ],
  "total": 10,
  "pages": 1,
  "current_page": 1
}
```

---

#### GET `/api/v1/shortcuts/<shortcut_identifier>`

Get shortcut by UUID with screenshots.

**Authentication:** Required

**Parameters:**

- `shortcut_identifier`: Shortcut UUID

**Authorization:**

- Approved shortcuts: accessible to all authenticated users
- Draft/pending/rejected shortcuts: only accessible to the author

**Response (200 OK):**

```json
{
  "shortcut": {
    "uuid": "123e4567-e89b-12d3-a456-426614174000",
    "name": "My Shortcut",
    "description": "Shortcut description",
    "sharing_url": "https://www.icloud.com/shortcuts/...",
    "author_id": 1,
    "author_username": "username",
    "author_profile_picture": "https://example.com/uploads/profile.jpg",
    "category": "productivity",
    "icon_filename": "icon.png",
    "status": "approved",
    "version": "1.0",
    "requires_ios26_ai": false,
    "updater_type": "shortcuty",
    "created_at": "2024-01-01T00:00:00Z",
    "updated_at": "2024-01-01T00:00:00Z"
  },
  "screenshots": [
    {
      "id": 1,
      "filename": "screenshot1.png",
      "url": "https://example.com/uploads/screenshots/screenshot1.png",
      "order": 1
    },
    {
      "id": 2,
      "filename": "screenshot2.png",
      "url": "https://example.com/uploads/screenshots/screenshot2.png",
      "order": 2
    }
  ],
  "latest_update": {
    "id": 1,
    "shortcut_id": 1,
    "changelog": "Updated version 1.1",
    "new_version": "1.1",
    "status": "approved",
    "approved_at": "2024-01-15T00:00:00Z"
  }
}
```

**Error Responses:**

- `404 Not Found`: Shortcut not found or not accessible
  ```json
  {
    "error": "Shortcut not found"
  }
  ```

---

#### GET `/api/v1/shortcuts/<shortcut_identifier>/history`

Get version history for a shortcut owned by the authenticated user.

**Authentication:** Required

**Parameters:**

- `shortcut_identifier`: Shortcut UUID

**Authorization:** Only the shortcut owner can view history

**Response (200 OK):**

```json
{
  "shortcut_name": "My Shortcut",
  "shortcut_uuid": "550e8400-e29b-41d4-a716-446655440000",
  "changelog": [
    {
      "version": "1.1",
      "changelog": "What's new in this version" | null,
      "date": "2025-01-15T10:30:00Z",
      "status": "approved" | "pending" | "rejected",
      "sharing_url": "https://www.icloud.com/shortcuts/..." | null,
      "changes": {
        "name": "New Name" | null,
        "description": "New Description" | null,
        "category": "Productivity" | null,
        "requires_ios26_ai": true | null,
        "updater_type": "shortcuty" | null,
        "new_version": "1.1" | null
      },
      "rejection_reason": "Reason for rejection" | null
    }
  ]
}
```

**Error Responses:**

- `404 Not Found`: Shortcut not found or not owned by user
```json
{
    "error": "Shortcut not found"
}
```

---

#### POST `/api/v1/shortcuts/<shortcut_identifier>/submit`

Submit a draft shortcut for review.

**Authentication:** Required

**Parameters:**

- `shortcut_identifier`: Shortcut UUID

**Authorization:** Only the shortcut author can submit

**Response (200 OK):**

```json
{
  "message": "Shortcut queued for review and scanning"
}
```

**Error Responses:**

- `400 Bad Request`: Shortcut is not a draft or rejected shortcut
  ```json
  {
    "error": "Shortcut is not a draft or rejected shortcut"
  }
  ```
- `403 Forbidden`: Permission denied
  ```json
  {
    "error": "Permission denied"
  }
  ```
- `404 Not Found`: Shortcut not found
  ```json
  {
    "error": "Shortcut not found"
  }
```

---

#### POST `/api/v1/shortcuts/<shortcut_identifier>/update`

Unified endpoint for updating shortcuts.

**Authentication:** Required

**Parameters:**

- `shortcut_identifier`: Shortcut UUID

**Authorization:** Only the shortcut author can update

**Request Body:**

```json
{
  "name": "New name",
  "description": "New description (max 500,000 characters)",
  "sharing_url": "https://www.icloud.com/shortcuts/...",
  "category": "Productivity",
  "requires_ios26_ai": false,
  "updater_type": "none",
  "new_version": "1.1",
  "changelog": "What's new in this version (max 10,000 characters, optional)"
}
```

**Fields:**

- All fields are optional, but at least one change must be provided
- `sharing_url`: Must be a valid iCloud sharing URL
- `category`: Must be one of the preset categories
- `updater_type`: One of `"shortcuty"`, `"third_party"`, or `"none"`
- `changelog`: Optional changelog text (max 10,000 characters)

**Behavior:**

- Draft shortcuts: Updates are applied immediately without admin review
- Pending shortcuts (first submission): Updates modify the submission directly
- Approved shortcuts: Creates an update record for admin review
- Rejected shortcuts: Changes status to pending and requires manual review

**Auto-approval:**

- If neither `sharing_url` nor `description` changed, the update is auto-approved
- If `sharing_url` or `description` changed, the update requires admin review and scanning

**Response (200 OK or 201 Created):**

```json
{
  "message": "Draft shortcut updated" | "Update approved automatically (no URL or description change)" | "Update queued for processing and scan" | "Update submitted for manual review (shortcut was previously rejected)",
  "update_id": 123
}
```

**Note:** `update_id` is only present for non-draft updates (when status is not "draft").

**Error Responses:**

- `400 Bad Request`: No changes provided, invalid field values, or validation errors
  ```json
  {
    "error": "No changes provided"
  }
  ```
  ```json
  {
    "error": "Description must be 500,000 characters or less"
  }
  ```
  ```json
  {
    "error": "Changelog must be 10,000 characters or less"
  }
  ```
  ```json
  {
    "error": "Sharing URL must be an iCloud sharing URL (https://www.icloud.com/shortcuts/...)"
  }
  ```
  ```json
  {
    "error": "Invalid category. Must be one of: Artificial Intelligence, Education, ..."
  }
  ```
  ```json
  {
    "error": "Invalid updater_type. Must be one of: shortcuty, third_party, none"
  }
  ```
- `403 Forbidden`: Permission denied
  ```json
  {
    "error": "Permission denied"
  }
  ```
- `404 Not Found`: Shortcut not found
  ```json
  {
    "error": "Shortcut not found"
  }
  ```
- `500 Internal Server Error`: Server error
  ```json
  {
    "error": "Internal server error during shortcut update"
  }
  ```

---

### Screenshots

#### POST `/api/v1/upload/screenshot`

Upload a screenshot for a shortcut.

**Authentication:** Required

**Request:**

- Content-Type: `multipart/form-data`
- `file`: Image file (JPEG, PNG, GIF)
- `shortcut_id`: Shortcut UUID (form field)

**Limits:**

- Maximum 5 screenshots per shortcut
- File must be a valid image (validated by content, not just extension)

**Response (200 OK):**

```json
{
  "screenshot": {
    "id": 1,
    "filename": "screenshots/2025/12/550e8400-e29b-41d4-a716-446655440000.png",
    "uploaded_at": "2025-01-15T10:30:00Z",
    "url": "https://shortcuty.app/uploads/screenshots/2025/12/550e8400-e29b-41d4-a716-446655440000.png"
  }
}
```

**Error Responses:**

- `400 Bad Request`: No file provided, invalid file extension, file validation failed, or screenshot limit reached
  ```json
  {
    "error": "No file provided"
  }
  ```
  ```json
  {
    "error": "Shortcut ID required"
  }
  ```
  ```json
  {
    "error": "Invalid file extension"
  }
  ```
  ```json
  {
    "error": "File content validation failed"
  }
  ```
  ```json
  {
    "error": "Maximum of 5 screenshots allowed per shortcut"
  }
  ```
- `403 Forbidden`: Permission denied (not the shortcut author)
  ```json
  {
    "error": "Permission denied"
  }
  ```
- `404 Not Found`: Shortcut not found
  ```json
  {
    "error": "Shortcut not found"
  }
  ```

---

#### DELETE `/api/v1/shortcuts/<shortcut_identifier>/screenshots/<screenshot_id>`

Delete a screenshot for a shortcut.

**Authentication:** Required

**Parameters:**

- `shortcut_identifier`: Shortcut UUID
- `screenshot_id`: Screenshot ID (integer)

**Authorization:** Only the shortcut author can delete screenshots

**Response (200 OK):**

```json
{
  "message": "Screenshot deleted successfully"
}
```

**Error Responses:**

- `403 Forbidden`: Permission denied or screenshot doesn't belong to shortcut
  ```json
  {
    "error": "Permission denied"
  }
  ```
  ```json
  {
    "error": "Screenshot does not belong to this shortcut"
  }
  ```
- `404 Not Found`: Shortcut or screenshot not found
  ```json
  {
    "error": "Shortcut not found"
}
```
  ```json
  {
    "error": "Screenshot not found"
  }
  ```
- `500 Internal Server Error`: Failed to delete screenshot
  ```json
  {
    "error": "Failed to delete screenshot"
  }
  ```

---

## Response Format

### Success Responses

- `200 OK`: Request successful
- `201 Created`: Resource created successfully

### Error Responses

- `400 Bad Request`: Invalid request data or validation error
- `401 Unauthorized`: Authentication required or invalid API key
- `403 Forbidden`: Permission denied
- `404 Not Found`: Resource not found
- `500 Internal Server Error`: Server error

### Error Response Format

```json
{
  "error": "Error message description"
}
```

---

## Data Types and Field Descriptions

### Shortcut Object

- `uuid` (string): Unique identifier for the shortcut (UUID format)
- `name` (string): Shortcut name
- `description` (string | null): Shortcut description, can be null
- `sharing_url` (string): iCloud sharing URL
- `icon_filename` (string | null): Relative path to icon file, can be null
- `version` (string | null): Version string (e.g., "1.0"), can be null
- `status` (string): One of `"draft"`, `"pending"`, `"approved"`, `"rejected"`
- `created_at` (string): ISO 8601 formatted datetime
- `updated_at` (string): ISO 8601 formatted datetime
- `category` (string | null): Category name, can be null
- `downloads` (integer): Download count (default: 0)
- `likes_count` (integer): Number of likes
- `requires_ios26_ai` (boolean): Whether shortcut requires iOS 26 AI features
- `updater_type` (string): One of `"none"`, `"shortcuty"`, `"third_party"` (default: `"none"`)
- `author` (string): Author username
- `liked` (boolean | null): Whether the authenticated user has liked this shortcut (only present when user_id is provided)
- `rejection_reason` (string | null): Rejection reason, only present for rejected shortcuts when requested
- `author_profile_picture` (string | null): Full URL to author's profile picture, can be null

### Screenshot Object

- `id` (integer): Screenshot ID
- `filename` (string): Relative path to screenshot file
- `uploaded_at` (string): ISO 8601 formatted datetime
- `url` (string): Full URL to screenshot (only in v1 API responses)

### ShortcutUpdate Object

- `id` (integer): Update ID
- `shortcut_id` (integer): Shortcut ID
- `new_version` (string | null): New version string, can be null
- `new_sharing_url` (string | null): New sharing URL, can be null
- `changelog` (string | null): Changelog text, can be null
- `status` (string): One of `"pending"`, `"approved"`, `"rejected"` (default: `"pending"`)
- `rejection_reason` (string | null): Rejection reason, can be null
- `field_changes` (string | object | null): JSON string or object containing field changes, can be null
- `created_at` (string): ISO 8601 formatted datetime
- `approved_at` (string | null): ISO 8601 formatted datetime when approved, can be null

### Changelog Entry Object (in history response)

- `version` (string | null): Version string, can be null
- `changelog` (string | null): Changelog text, can be null
- `date` (string): ISO 8601 formatted datetime
- `status` (string): One of `"approved"`, `"pending"`, `"rejected"`
- `sharing_url` (string | null): Sharing URL for this version, can be null
- `changes` (object): Object containing field changes (excluding sharing_url fields)
- `rejection_reason` (string | null): Rejection reason, only present if status is "rejected"

---

## Notes

1. UUID-only identifiers: v1 API uses UUIDs for shortcuts, not numeric IDs
2. Full URLs: Upload URLs in responses are full URLs (e.g., `https://shortcuty.app/uploads/...`)
3. Draft shortcuts: Can be updated immediately without admin review
4. Auto-approval: Updates that don't change `sharing_url` or `description` are auto-approved
5. Scanning: Shortcuts and updates with URL/description changes are queued for security scanning
6. Status values: `draft`, `pending`, `approved`, `rejected`
7. Categories: Must match one of the preset categories exactly (case-sensitive)
8. Datetime format: All datetime fields use ISO 8601 format (e.g., `"2025-01-15T10:30:00Z"`)

---

## Example Usage

### Create a Draft Shortcut

```bash
curl -X POST https://shortcuty.app/api/v1/shortcuts \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "sharing_url": "https://www.icloud.com/shortcuts/abc123",
    "description": "My awesome shortcut",
    "category": "Productivity",
    "submit": false
  }'
```

### Submit Draft for Review

```bash
curl -X POST https://shortcuty.app/api/v1/shortcuts/UUID/submit \
  -H "Authorization: Bearer YOUR_API_KEY"
```

### Update Shortcut

```bash
curl -X POST https://shortcuty.app/api/v1/shortcuts/UUID/update \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "new_version": "1.1",
    "changelog": "Added new features",
    "description": "Updated description"
  }'
```

### Upload Screenshot

```bash
curl -X POST https://shortcuty.app/api/v1/upload/screenshot \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -F "file=@screenshot.png" \
  -F "shortcut_id=UUID"
```

---

## Command-Line Interface

A command-line interface for managing shortcuts via the Shortcuty Creator API v1 is available:

**[Shortcuty Creator API CLI](https://github.com/Shortcuty/Shortcuty-CLI)**

The CLI provides an easy way to interact with the API from your terminal, with features including:
- Create, update, and manage shortcuts
- Upload and delete screenshots
- View shortcut history and changelogs
- List and filter your shortcuts
- JSON output support for scripting
