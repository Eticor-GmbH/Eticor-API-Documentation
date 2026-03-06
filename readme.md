# Eticor API Documentation

## Table of Contents

- [Introduction](#introduction)
  - [Authentication](#authentication)
  - [Pagination](#pagination)
  - [Language](#language)
  - [Multi-Tenancy](#multi-tenancy)
  - [Error Handling](#error-handling)
- [Public API](#public-api)
  - [GET /v2/public/delegations](#get-v2publicdelegations)
- [Legal API](#legal-api)
  - [GET /v2/tasks](#get-v2tasks)
  - [GET /v2/delegations](#get-v2delegations)
  - [POST /v2/delegations/multiple](#post-v2delegationsmultiple)
  - [PUT /v2/delegations/{delegationId}](#put-v2delegationsdelegationid)
  - [POST /v2/inspections](#post-v2inspections)
- [Enumerations](#enumerations)

---

## Introduction

This document describes the Eticor REST API endpoints used for managing delegations, tasks, and inspections. The API is split into two logical groups:

- **Public API** -- available under `/v2/public/`, intended for external or cross-system integrations.
- **Legal API** -- available directly under `/v2/`, intended for the Eticor web application front-end.

Interactive Swagger documentation is available at:

- [Public API Swagger](https://intapi.eticor-portal.com/swagger/index.html?urls.primaryName=Public+API)
- [Legal API Swagger](https://intapi.eticor-portal.com/swagger/index.html?urls.primaryName=Legal+API)

### Authentication

All endpoints require Bearer token authentication. Include the token in the `Authorization` header:

```
Authorization: Bearer <token>
```

### Pagination

Paginated endpoints return a `PageResult<T>` envelope:

```json
{
  "total": 142,
  "offset": 0,
  "pageSize": 50,
  "items": [ ... ]
}
```

| Field      | Type    | Description                                   |
| ---------- | ------- | --------------------------------------------- |
| `total`    | integer | Total number of records matching the filters. |
| `offset`   | integer | The zero-based offset used in the request.    |
| `pageSize` | integer | Number of items returned in this page.        |
| `items`    | array   | The result items for the current page.        |

Pagination parameters common to all paginated endpoints:

| Parameter    | Type    | Default | Description                           |
| ------------ | ------- | ------- | ------------------------------------- |
| `offset`     | integer | `0`     | Number of records to skip.            |
| `limit`      | integer | `50`    | Maximum number of records to return.  |
| `orderBy`    | string  | varies  | Property name to sort by.             |
| `descending` | boolean | `false` | Sort in descending order when `true`. |

### Language

The API supports several languages by using the language's two letter ISO code in the request header:

For example:

```
Accept-Language: de
```

returns the content in German.

### Multi-Tenancy

A client ID must be provided in the request headers in order for the API to know from which client to load data:

```
ClientID: 10000
```

### Error Handling

The API uses standard HTTP status codes:

| Code | Meaning                                                        |
| ---- | -------------------------------------------------------------- |
| 200  | Success.                                                       |
| 400  | Bad request -- validation failed. Body contains error details. |
| 401  | Unauthorized -- missing or invalid token.                      |
| 403  | Forbidden -- insufficient permissions.                         |
| 404  | Resource not found.                                            |
| 500  | Internal server error.                                         |

Validation errors (400) are returned by FluentValidation and follow this structure:

```json
{
  "errors": {
    "PropertyName": ["Error message."]
  }
}
```

---

## Public API

### GET /v2/public/delegations

Returns a paginated list of delegations. This is the primary endpoint for external systems to query delegation data.

**Route:** `GET /v2/public/delegations`

**Authentication:** Bearer token required.

**Authorization:** No additional policy beyond authentication. The service scopes results based on the authenticated user's role and org-unit access.

#### Query Parameters

| Parameter            | Type                | Required | Default     | Description                                                                                                             |
| -------------------- | ------------------- | -------- | ----------- | ----------------------------------------------------------------------------------------------------------------------- |
| `offset`             | integer             | No       | `0`         | Number of records to skip.                                                                                              |
| `limit`              | integer             | No       | `50`        | Maximum number of records to return.                                                                                    |
| `orderBy`            | string              | No       | `"duedate"` | Sort property. Allowed: `duedate`, `task`, `responsible`, `module`, `taskId`.                                           |
| `descending`         | boolean             | No       | `false`     | Sort direction.                                                                                                         |
| `delegationId`       | integer             | No       |             | Filter by a specific delegation ID.                                                                                     |
| `taskId`             | integer             | No       |             | Filter delegations by task ID.                                                                                          |
| `responsibleId`      | integer             | No       |             | Filter by the responsible employee ID.                                                                                  |
| `controllerId`       | integer             | No       |             | Filter by the controller employee ID.                                                                                   |
| `deputyId`           | integer             | No       |             | Filter by the deputy employee ID.                                                                                       |
| `employeeId`         | integer             | No       |             | Filter by employee ID across all roles (responsible, controller, deputy, delegation deputy).                            |
| `startDate`          | datetime            | No       |             | Filter delegations with a due date on or after this date.                                                               |
| `endDate`            | datetime            | No       |             | Filter delegations with a due date on or before this date.                                                              |
| `orgUnitPath`        | string              | No       |             | Hierarchy path to filter by org-unit subtree (e.g., `"/1/3/"`).                                                         |
| `orgUnitIds`         | integer[]           | No       |             | Filter by specific org-unit IDs (includes descendants).                                                                 |
| `sourcePath`         | string              | No       |             | Hierarchy path to filter by task source subtree.                                                                        |
| `roleId`             | integer             | No       |             | Filter by role ID.                                                                                                      |
| `taskPackageId`      | integer             | No       |             | Filter by task package ID.                                                                                              |
| `searchQuery`        | string              | No       |             | Free-text search. Numeric values search by task ID; text values search task text, description, and delegation comments. |
| `tagIds`             | string[]            | No       |             | Filter by tag IDs.                                                                                                      |
| `usedInTaskPackages` | boolean             | No       |             | `true`: only delegations from task packages. `false`: only delegations not from task packages.                          |
| `usedInRoles`        | boolean             | No       |             | `true`: only delegations used in roles. `false`: exclude role-based delegations.                                        |
| `riskFactor`         | integer             | No       |             | Filter by risk factor (1 = Normal, 2 = High, 3 = Very High).                                                            |
| `delegationDateType` | DelegationDateTypes | No       |             | Filter by date type: `Due`, `Dated`, `Permanent`.                                                                       |
| `delegationStatus`   | DelegationStatus[]  | No       |             | Filter by status: `PermanentOk`, `Green`, `Yellow`, `PermanentOverdue`, `Red`.                                          |

#### Response

`PageResult<DelegationModel>`

```json
{
  "total": 87,
  "offset": 0,
  "pageSize": 50,
  "items": [
    {
      "id": 1234,
      "taskId": 56,
      "orgUnitId": 12,
      "orgUnit": { "id": 12, "name": "Site A", "parents": [...] },
      "responsibleId": 99,
      "responsible": { "id": 99, "firstName": "Max", "lastName": "Mustermann", "deputy": { ... } },
      "controllerId": 100,
      "dueDate": "2026-06-15T00:00:00Z",
      "delegationStatus": "Green",
      "intervalType": "Regular",
      "interval": "Monthly",
      "intervalCount": 3,
      "warningPeriod": 14,
      "comment": "Check required",
      "module": "Eticor",
      "isDisabled": false,
      "task": {
        "id": 56,
        "text": "Fire safety inspection",
        "description": "Annual fire safety check.",
        "sources": [...],
        "status": "Added"
      },
      "taskPackages": [{ "id": 10, "name": "Safety Package" }],
      "roles": [{ "id": 5, "name": "Safety Officer" }]
    }
  ]
}
```

#### Service Behavior

- The service resolves the caller's user ID and admin status from the HTTP context.
- Admin users see all delegations for the customer; non-admin users see only delegations within their accessible org units.
- Only active, non-archived tasks are included (`isActive = true`, `isArchived = false`).
- When `searchQuery` is a numeric value, it searches by task ID. Text values use wildcard matching on task text, task description, task translations, and delegation comments.
- Results include related task packages and roles by default (except when fetched from the cockpit context).
- The `orderBy` options and their secondary sort keys:
  - `duedate`: primary by delegation status (descending), then by due date, then by task ID.
  - `task`: primary by task text, then by delegation status, then by due date.
  - `responsible`: primary by delegation status, then by last name, then by first name.
  - `module`: primary by module, then by delegation status, then by due date, then by task ID.
  - `taskId`: primary by delegation status, then by task ID, then by due date.

---

## Legal API

### GET /v2/tasks

Returns a paginated list of tasks. Tasks define responsibilities for employees.

**Route:** `GET /v2/tasks`

**Authentication:** Bearer token required.

**Authorization:** No additional policy beyond authentication. Results are scoped to tasks licensed for the current customer.

#### Query Parameters

| Parameter              | Type     | Required | Default | Description                                                                                    |
| ---------------------- | -------- | -------- | ------- | ---------------------------------------------------------------------------------------------- |
| `offset`               | integer  | No       | `0`     | Number of records to skip.                                                                     |
| `limit`                | integer  | No       | `50`    | Maximum number of records to return.                                                           |
| `orderBy`              | string   | No       |         | Property name to sort by.                                                                      |
| `descending`           | boolean  | No       | `false` | Sort direction.                                                                                |
| `searchQuery`          | string   | No       |         | Free-text search. Numeric values match by task ID; text values search task text and law title. |
| `extend`               | string[] | No       |         | Include related entities. See [Extend Options](#task-extend-options) below.                    |
| `exactMatch`           | boolean  | No       | `false` | When `true`, requires exact match on search query.                                             |
| `lawId`                | integer  | No       |         | Filter by law ID.                                                                              |
| `lawAreaId`            | integer  | No       |         | Filter by law area ID.                                                                         |
| `sourceId`             | integer  | No       |         | Filter by source ID.                                                                           |
| `delegationId`         | integer  | No       |         | Filter tasks that have a specific delegation.                                                  |
| `regionId`             | integer  | No       |         | Filter by region ID.                                                                           |
| `topDownRegionFilter`  | boolean  | No       | `false` | When `true`, applies top-down hierarchy region filtering.                                      |
| `isActive`             | boolean  | No       |         | Filter by active status.                                                                       |
| `isArchived`           | boolean  | No       |         | Filter by archived status.                                                                     |
| `isDelegated`          | boolean  | No       |         | `true`: only tasks with active delegations. `false`: only tasks without active delegations.    |
| `employeeId`           | integer  | No       |         | Filter by responsible employee ID.                                                             |
| `orgUnitId`            | integer  | No       |         | Filter tasks delegated to a specific org unit (includes descendants).                          |
| `siteId`               | integer  | No       |         | Filter by site ID.                                                                             |
| `internalRegulationId` | integer  | No       |         | Filter by internal regulation ID (includes descendants via breadcrumb).                        |
| `permitId`             | integer  | No       |         | Filter by permit ID (includes descendants via breadcrumb).                                     |
| `contractId`           | integer  | No       |         | Filter by contract ID (includes descendants via breadcrumb).                                   |
| `module`               | string   | No       | `"ALL"` | Filter by module: `Eticor`, `KVP`, `APPROVAL`, `SUPPLIER`, `AUDIT`, or `ALL`.                  |
| `excludeModule`        | string   | No       |         | Exclude a specific module from results.                                                        |
| `responsibleId`        | integer  | No       |         | Filter by responsible employee on delegations.                                                 |
| `roleId`               | integer  | No       |         | Filter tasks by role ID.                                                                       |
| `taskPackageId`        | integer  | No       |         | Filter tasks by task package ID.                                                               |
| `tagIds`               | string[] | No       |         | Filter by tag IDs.                                                                             |

#### Task Extend Options

The `extend` parameter accepts the following values (combinable):

| Value                | Description                                                                 |
| -------------------- | --------------------------------------------------------------------------- |
| `sources` / `laws`   | Include the task's legal source, law, law area, and region.                 |
| `delegations`        | Include associated delegations with responsible, org unit, and inspections. |
| `tags`               | Include task tags.                                                          |
| `sites`              | Include site-task associations and site comments.                           |
| `internalRegulation` | Include the linked internal regulation.                                     |
| `permit`             | Include the linked permit.                                                  |
| `contract`           | Include the linked contract.                                                |
| `roles`              | Include associated roles.                                                   |
| `taskPackages`       | Include associated task packages.                                           |

#### Response

`PageResult<TaskModel>`

```json
{
  "total": 320,
  "offset": 0,
  "pageSize": 50,
  "items": [
    {
      "id": 56,
      "text": "Fire safety inspection",
      "description": "Annual fire safety check.",
      "module": "Eticor",
      "isActive": true,
      "isArchived": false,
      "intervalType": "Regular",
      "interval": "Yearly",
      "intervalCount": 1,
      "warningPeriod": 30,
      "hint": "Refer to section 4.2",
      "riskFactor": 2,
      "hasDocuments": true,
      "sources": [
        {
          "id": 101,
          "law": {
            "id": 42,
            "title": "Fire Protection Act",
            "lawArea": { "id": 7, "name": "Occupational Safety" }
          }
        }
      ],
      "translations": [
        {
          "localeKey": "de-AT",
          "text": "Brandschutzinspektion",
          "description": "..."
        }
      ]
    }
  ]
}
```

#### Service Behavior

- The service retrieves tasks licensed for the current customer based on module licensing.
- Task translations and custom texts are always loaded.
- The `searchQuery` applies as: numeric values match by task ID (Eticor admins also match by `masterId`); text values use wildcard matching on task text and law title.
- When `module` is `"ALL"`, tasks from all licensed modules are included (Eticor, KVP, APPROVAL, SUPPLIER, AUDIT).
- Region filtering supports both top-down and bottom-up modes via `topDownRegionFilter`.
- Results undergo in-memory post-filtering for region-based constraints after the database query.

---

### GET /v2/delegations

Returns a paginated list of delegations. This endpoint is used by the Eticor front-end application.

**Route:** `GET /v2/delegations`

**Authentication:** Bearer token required.

**Authorization:** No additional policy beyond authentication. The service scopes results based on user role and org-unit access level.

#### Query Parameters

The query parameters are identical to [GET /v2/public/delegations](#query-parameters). Refer to that section for the full parameter table.

#### Response

`PageResult<DelegationModel>` -- same structure as [GET /v2/public/delegations](#response).

#### Differences from Public API

Both endpoints delegate to the same service method (`DelegationListService.GetDelegationsAsync`), so filtering, sorting, and pagination behavior is identical. The key differences are:

| Aspect            | GET /v2/public/delegations      | GET /v2/delegations     |
| ----------------- | ------------------------------- | ----------------------- |
| Route             | `/v2/public/delegations`        | `/v2/delegations`       |
| Controller        | `PublicAccessController`        | `DelegationsController` |
| Intended consumer | External systems / integrations | Eticor web application  |
| Swagger group     | Public API                      | Legal API               |

#### Service Behavior

Identical to [GET /v2/public/delegations -- Service Behavior](#service-behavior).

---

### POST /v2/delegations/multiple

Creates or deletes multiple delegations in a single batch operation. The request body maps task IDs to lists of delegation actions (create or delete).

**Route:** `POST /v2/delegations/multiple`

**Authentication:** Bearer token required.

**Authorization:** No additional policy attribute, but the service enforces that the current user has **write access** to each referenced org unit. Delegation creation also requires that the responsible employee is licensed for the target org unit.

#### Request Body

`CreateOrDeleteMultipleDelegationsModel`

```json
{
  "items": {
    "56": [
      {
        "value": true,
        "orgUnitId": 12,
        "responsibleId": 99,
        "startDate": "2026-04-01T00:00:00Z",
        "comment": "New assignment"
      },
      {
        "value": false,
        "orgUnitId": 12,
        "responsibleId": 100
      }
    ],
    "78": [
      {
        "value": true,
        "orgUnitId": 15,
        "responsibleId": 99
      }
    ]
  }
}
```

**Schema:**

| Field   | Type                                                        | Required | Description                                                           |
| ------- | ----------------------------------------------------------- | -------- | --------------------------------------------------------------------- |
| `items` | `object<int, CreateOrDeleteMultipleDelegationsItemModel[]>` | Yes      | Dictionary mapping task IDs (as keys) to lists of delegation actions. |

**CreateOrDeleteMultipleDelegationsItemModel:**

| Field           | Type     | Required | Default | Description                                                                |
| --------------- | -------- | -------- | ------- | -------------------------------------------------------------------------- |
| `value`         | boolean  | Yes      |         | `true` to create/activate the delegation, `false` to delete/deactivate it. |
| `orgUnitId`     | integer  | Yes      |         | The org unit for the delegation.                                           |
| `responsibleId` | integer  | Yes      |         | The responsible employee for the delegation.                               |
| `startDate`     | datetime | No       |         | Start date for the delegation (only used when `value` is `true`).          |
| `comment`       | string   | No       | `""`    | Comment attached to the delegation.                                        |

#### Validation Rules

The following rules are enforced by `CreateOrDeleteMultipleDelegationsModelValidator`:

| Rule                           | Description                                                                                                                                                    |
| ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `items` not null/empty         | The items dictionary must contain at least one entry.                                                                                                          |
| Task exists                    | Each task ID key must correspond to an existing task in the database.                                                                                          |
| Employee exists                | Each `responsibleId` must correspond to an existing employee.                                                                                                  |
| Org unit exists                | Each `orgUnitId` must correspond to an existing org unit.                                                                                                      |
| Write access (create)          | When `value` is `true`, the current user must have write access to the org unit, unless a delegation already exists for that responsible/org-unit combination. |
| Write access (delete)          | When `value` is `false`, the current user must have write access to the org unit.                                                                              |
| Start date not in past         | When `value` is `true` and `startDate` is provided, it must not be before today (UTC).                                                                         |
| Employee licensed for org unit | When `value` is `true`, the responsible employee must be licensed (have at least `None` access level) for the target org unit.                                 |

#### Response

`MultipleDelegationsCreatedModel`

```json
{
  "added": [
    {
      "id": 5001,
      "taskId": 56,
      "orgUnitId": 12,
      "responsibleId": 99,
      "dueDate": "2026-04-01T00:00:00Z",
      "delegationStatus": "Green",
      "isDisabled": false
    }
  ],
  "removed": [
    {
      "id": 4800,
      "taskId": 56,
      "orgUnitId": 12,
      "responsibleId": 100,
      "isDisabled": true
    }
  ]
}
```

| Field     | Type                | Description                                                                      |
| --------- | ------------------- | -------------------------------------------------------------------------------- |
| `added`   | `DelegationModel[]` | Delegations that were created or re-activated.                                   |
| `removed` | `DelegationModel[]` | Delegations that were deleted or deactivated (deactivated if inspections exist). |

#### Service Behavior

- The service processes each task/delegation pair independently.
- **Creating delegations (`value: true`):**
  - If a matching delegation already exists for the same task + org unit + responsible combination, it is re-activated instead of creating a duplicate.
  - New delegations are added to the database context and saved in a single transaction.
- **Deleting delegations (`value: false`):**
  - The delegation is fully removed if it has no inspections.
  - If the delegation has inspections, it is **deactivated** (`isDisabled = true`) instead of deleted to preserve audit history.

---

### PUT /v2/delegations/{delegationId}

Updates an existing delegation.

**Route:** `PUT /v2/delegations/{delegationId}`

**Authentication:** Bearer token required.

**Authorization:** `DelegationWritePolicy` -- the caller must have write-level access to the delegation.

#### Path Parameters

| Parameter      | Type    | Required | Description               |
| -------------- | ------- | -------- | ------------------------- |
| `delegationId` | integer | Yes      | The ID of the delegation. |

#### Request Body

`DelegationUpdateModel`

```json
{
  "id": 1234,
  "controllerId": 100,
  "comment": "Updated assignment details",
  "riskFactor": 2,
  "intervalType": "Regular",
  "interval": "Monthly",
  "intervalCount": 3,
  "warningPeriod": 14,
  "dueDate": null,
  "startDate": "2026-04-01T00:00:00Z",
  "isDisabled": false
}
```

**Schema:**

| Field           | Type         | Required | Description                                                                                                        |
| --------------- | ------------ | -------- | ------------------------------------------------------------------------------------------------------------------ |
| `id`            | integer      | Yes      | The delegation ID (must match the route parameter).                                                                |
| `controllerId`  | integer?     | No       | The controller employee ID. Set to `null` to remove the controller.                                                |
| `comment`       | string       | No       | Free-text comment for the delegation.                                                                              |
| `riskFactor`    | integer?     | No       | Risk factor: `1` (Normal), `2` (High), `3` (Very High). Clamped to [1, 3].                                         |
| `intervalType`  | IntervalType | Yes      | Delegation interval type: `Permanent`, `Single`, `Regular`, `Fixed`.                                               |
| `interval`      | Interval     | Yes      | Interval unit: `None`, `Daily`, `Weekly`, `Monthly`, `Yearly`.                                                     |
| `intervalCount` | integer      | Yes      | Number of interval units per cycle.                                                                                |
| `warningPeriod` | integer      | Yes      | Warning period in days before the due date.                                                                        |
| `dueDate`       | datetime?    | No       | Explicitly set the due date. If omitted and interval type changes to/from Permanent, the due date is recalculated. |
| `startDate`     | datetime?    | No       | Update the start date. If changed and no inspections exist, the due date is recalculated from the new start date.  |
| `isDisabled`    | boolean      | Yes      | Set to `true` to deactivate the delegation.                                                                        |

#### Validation Rules

The following rules are enforced by `DelegationUpdateModelValidator`:

| Field           | Rule                                                                                           |
| --------------- | ---------------------------------------------------------------------------------------------- |
| `id`            | Must match an existing delegation in the database.                                             |
| `controllerId`  | If provided, must match an active employee.                                                    |
| `intervalCount` | Must be >= 1 when `intervalType` is `Fixed` or `Regular`.                                      |
| `interval`      | Must be `None` when `intervalType` is `Permanent` or `Single`.                                 |
| `interval`      | Must be `Daily`, `Weekly`, `Monthly`, or `Yearly` when `intervalType` is `Fixed` or `Regular`. |
| `warningPeriod` | Must be >= 0.                                                                                  |
| `riskFactor`    | Must be between 1 and 3 (inclusive).                                                           |

#### Response

`DelegationModel` -- the updated delegation.

```json
{
  "id": 1234,
  "taskId": 56,
  "orgUnitId": 12,
  "responsibleId": 99,
  "controllerId": 100,
  "dueDate": "2026-07-01T00:00:00Z",
  "delegationStatus": "Green",
  "intervalType": "Regular",
  "interval": "Monthly",
  "intervalCount": 3,
  "warningPeriod": 14,
  "comment": "Updated assignment details",
  "riskFactor": 2,
  "isDisabled": false,
  "task": { ... }
}
```

#### Service Behavior

- The service loads the delegation with write-level access checking.
- The caller must have write access to the delegation's org unit to update fields like `controllerId`, `intervalType`, `interval`, `intervalCount`, `startDate`, `dueDate`, and `isDisabled`. Without org-unit write access, only `comment` and `warningPeriod` can be updated.
- **Due date recalculation:**
  - When `startDate` is changed and the delegation has no inspections, the due date is recalculated from the new start date.
  - When `intervalType` changes to or from `Permanent`, the due date is recalculated. Permanent delegations have no due date (`null`).
  - An explicit `dueDate` value in the request overrides automatic recalculation.
- The `riskFactor` value is clamped to the range [1, 3] regardless of the input.
- Changes that affect mail relevance (warning period, controller, interval, start date, disabled state) update `lastMailRelevantChange` to trigger notification processing.
- An event log entry is written for every update with the serialized model payload.

---

### POST /v2/inspections

Creates a new inspection for a delegation. An inspection records whether a delegated task has been checked/fulfilled.

**Route:** `POST /v2/inspections`

**Authentication:** Bearer token required.

**Authorization:** No additional policy attribute on the endpoint, but the service verifies that the caller has write-level access to the delegation's license. The validator enforces that the inspector is authorized (see validation rules).

#### Request Body

`InspectionPostModel`

```json
{
  "delegationId": 1234,
  "comment": "All safety measures verified and in order.",
  "inspectionDate": "2026-03-05T00:00:00Z",
  "inspectorId": 99,
  "isComplete": true,
  "inspectionType": "DueDate",
  "documents": [
    {
      "fileName": "checklist.pdf",
      "mimeType": "application/pdf",
      "bytes": "<base64-encoded content>"
    }
  ]
}
```

**Schema:**

| Field            | Type                | Required | Description                                                       |
| ---------------- | ------------------- | -------- | ----------------------------------------------------------------- |
| `delegationId`   | integer             | Yes      | The ID of the delegation this inspection belongs to.              |
| `comment`        | string              | Yes      | Inspection comment/notes. Must not be empty.                      |
| `inspectionDate` | datetime            | Yes      | The date the inspection was performed. Must not be in the future. |
| `inspectorId`    | integer             | Yes      | The employee ID of the inspector.                                 |
| `isComplete`     | boolean             | Yes      | Whether the task is marked as complete in this inspection.        |
| `inspectionType` | InspectionType      | Yes      | Type of inspection: `DueDate` or `FreeForm`.                      |
| `documents`      | DocumentPostModel[] | No       | Optional list of documents to attach to the inspection.           |

**DocumentPostModel:**

| Field      | Type   | Required | Description                  |
| ---------- | ------ | -------- | ---------------------------- |
| `fileName` | string | Yes      | Original file name.          |
| `mimeType` | string | Yes      | MIME type of the document.   |
| `bytes`    | string | Yes      | Base64-encoded file content. |

#### Validation Rules

The following rules are enforced by `InspectionPostModelValidator`:

| Field            | Rule                                                                                          |
| ---------------- | --------------------------------------------------------------------------------------------- |
| `inspectionDate` | Required. Must not be empty. Must not be in the future (max: tomorrow's date UTC).            |
| `delegationId`   | Must match an existing delegation.                                                            |
| `comment`        | Required. Must not be empty.                                                                  |
| `isComplete`     | Required. Must not be null.                                                                   |
| `inspectionType` | Must be a valid enum value (`DueDate` or `FreeForm`).                                         |
| `inspectorId`    | Must match an existing, non-deleted employee. Additionally, the inspector must be authorized: |

**Inspector authorization check** (applied when the caller is not an Admin):

The inspector must satisfy at least one of the following conditions:

1. The inspector is the **responsible** employee on the delegation.
2. The inspector is a **delegation deputy** on the delegation.
3. The inspector is the **controller** on the delegation.
4. The inspector is the **deputy of the responsible** employee.
5. The inspector is a **site admin** with a write license matching the delegation's org unit and module.

If none of these conditions are met, the validation fails with: _"Creation of an inspection entry not allowed!"_

#### Response

`InspectionModel`

```json
{
  "id": 8901,
  "delegationId": 1234,
  "taskId": 56,
  "orgUnitId": 12,
  "inspectorId": 99,
  "inspectorName": "Max Mustermann",
  "inspectionDate": "2026-03-05T00:00:00Z",
  "comment": "All safety measures verified and in order.",
  "isComplete": true,
  "createdDate": "2026-03-05T14:23:00Z",
  "module": "Eticor"
}
```

#### Service Behavior

- The service loads the delegation with all related entities (responsible, controller, delegation deputies, task, inspections).
- The delegation must not be disabled. Attempting to create an inspection for a disabled delegation results in an `400 - Bad Request` response.
- The inspector employee must exist and must not be marked as deleted.
- The `inspectorName` is derived from the HTTP context claims principal name if the inspector is the current user, otherwise from the employee's first and last name.
- **Due date handling for `DueDate` inspection type:**
  - For non-permanent delegations: the `inspectionDatePreset` is set to the current due date of the delegation.
  - For permanent delegations: the `inspectionDatePreset` is set to the inspection date itself.
  - If `isComplete` is `true`, the delegation's due date is recalculated based on the interval settings, using the latest complete inspection date.
  - The delegation's `lastInspectionDate` is updated to the inspection date.
- The delegation's `controlled` flag is reset to `0` to enable the controls workflow.
- Attached documents are saved via the document service after the inspection is persisted.
- The due date is always stored as date-only (time component stripped).

---

## Enumerations

### DelegationStatus

| Value              | Description                             |
| ------------------ | --------------------------------------- |
| `PermanentOk`      | Permanent delegation in good standing.  |
| `Green`            | Dated delegation within safe timeframe. |
| `Yellow`           | Dated delegation within warning period. |
| `PermanentOverdue` | Permanent delegation that is overdue.   |
| `Red`              | Dated delegation that is past due.      |

### DelegationDateTypes

| Value       | Description                                                      |
| ----------- | ---------------------------------------------------------------- |
| `Due`       | Delegations that are currently overdue (status >= 3).            |
| `Dated`     | Delegations with a date-based interval (Single, Regular, Fixed). |
| `Permanent` | Delegations with a permanent (ongoing) interval.                 |

### IntervalType

| Value       | Description                                    |
| ----------- | ---------------------------------------------- |
| `Permanent` | No recurring schedule; ongoing responsibility. |
| `Single`    | One-time delegation with a single due date.    |
| `Regular`   | Repeating delegation on a regular interval.    |
| `Fixed`     | Repeating delegation on a fixed interval.      |

### Interval

| Value     | Description                                        |
| --------- | -------------------------------------------------- |
| `None`    | No interval unit (used with Permanent and Single). |
| `Daily`   | Interval measured in days.                         |
| `Weekly`  | Interval measured in weeks.                        |
| `Monthly` | Interval measured in months.                       |
| `Yearly`  | Interval measured in years.                        |

### InspectionType

| Value      | Description                                         |
| ---------- | --------------------------------------------------- |
| `DueDate`  | Inspection tied to the delegation's due date cycle. |
| `FreeForm` | Ad-hoc inspection not tied to the due date.         |

### RiskFactor

| Value | Name      | Description    |
| ----- | --------- | -------------- |
| `1`   | Normal    | Standard risk. |
| `2`   | High      | Elevated risk. |
| `3`   | Very High | Critical risk. |
