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

All API requests require authentication using a Bearer token. You can obtain your API key from [browse.shortcuty.app](https://browse.shortcuty.app) in your account settings. Include your API key in the `Authorization` header for every request. Once authenticated, you can start creating and managing shortcuts using the endpoints documented below.



## Authentication

All endpoints require API key authentication:

```

Authorization: Bearer <your_api_key>

```



## Endpoints



### Create Shortcut

**POST** `/api/v1/shortcuts`



Creates a new shortcut (draft or submitted).



**Request Body:**

```json

{

  "sharing_url": "https://www.icloud.com/shortcuts/...",  // Required

  "description": "Shortcut description",                  // Optional

  "category": "Productivity",                             // Optional

  "updater_type": "shortcuty",                           // Optional: "shortcuty" | "third_party" | "none" (default: "none")

  "requires_ios26_ai": false,                            // Optional (default: false)

  "auto_submit": false                                    // Optional: submit immediately for review (default: false)

}

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



**Validation:**

- `sharing_url` must be a valid iCloud sharing URL

- `category` must be one of the preset categories

- `name` is read-only (extracted from shortcut directly)




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



Updates a shortcut. Drafts are updated immediately; approved shortcuts require security review.



**Request Body:**

```json

{

  "description": "Updated description",     // Optional

  "sharing_url": "https://...",             // Optional

  "category": "Productivity",                // Optional

  "requires_ios26_ai": false,               // Optional

  "updater_type": "shortcuty",               // Optional

  "version": "1.2.0",                       // Optional

  "changelog": "What's new in this version"  // Optional

}

```



**Response (200/201):**

```json

{

  "message": "Draft shortcut updated"  // or "Update queued for processing and scan"

}

```



**Validation:**

- `name` is read-only (cannot be set)

- `version` must be provided

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



Returns version history for a shortcut.



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



## Response Fields



All shortcut responses include these fields:



| Field | Type | Description |

|-------|------|-------------|

| `uuid` | string | Unique identifier |

| `name` | string | Shortcut name (read-only, server-scanned) |

| `description` | string\|null | Description |

| `sharing_url` | string | iCloud sharing URL |

| `version` | string\|null | Version number |

| `status` | string | `draft` \| `pending` \| `approved` \| `rejected` |

| `category` | string\|null | Category name |

| `created_at` | string\|null | ISO 8601 timestamp |

| `updated_at` | string\|null | ISO 8601 timestamp |

| `requires_ios26_ai` | boolean | iOS 26 AI requirement flag |

| `updater_type` | string | `shortcuty` \| `third_party` \| `none` |

| `rejection_reason` | string | Only present if `status` is `"rejected"` |


---



## Validation Rules



### Field Naming

- Use `auto_submit`

- Use `version`



### Error Responses



**400 Bad Request:**
```json

{

  "error": "Unknown field(s): submit. Allowed fields: auto_submit, category, description, requires_ios26_ai, sharing_url, updater_type"

}

```



**401 Unauthorized:**

```json

{

  "error": "Invalid API key"

}

```



**404 Not Found:**

```json

{

  "error": "Shortcut not found"

}

```



---



## Categories



Valid categories (case-sensitive):

- Artificial Intelligence

- Education

- Entertainment

- Finance

- Food & Dining

- Gaming

- Health & Fitness

- Music

- News

- Photography

- Productivity

- Shopping

- Shortcut Extension

- Sideloading

- Social Media

- Sports

- Technology

- Travel

- Utilities

- Weather

---
