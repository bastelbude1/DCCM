# 🚀 PROJECT SPECIFICATION: Dynamic Central Configuration Manager (DCCM)

**Date:** November 19, 2025
**Technology Base:** Python / NiceGUI / Static Web Server
**Architecture Pattern:** Two-Tier Management & Retrieval

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Architecture Overview: Two-Tier Model](#2-architecture-overview-two-tier-model)
   - 2.1 [Component Separation](#21-component-separation)
   - 2.2 [Storage Architecture (Filesystem Separation)](#22-storage-architecture-filesystem-separation)
   - 2.3 [Architecture Diagram](#23-architecture-diagram)
   - 2.4 [Data Flow](#24-data-flow)
   - 2.5 [Internal Architecture (Management Flow)](#25-internal-architecture-management-flow)
   - 2.6 [System Bootstrap & Initial Access](#26-system-bootstrap--initial-access)
   - 2.7 [Multi-Environment Architecture](#27-multi-environment-architecture)
     - 2.7.1 [Environment Registry](#271-environment-registry)
     - 2.7.2 [Environment Selection (Mandatory)](#272-environment-selection-mandatory)
     - 2.7.3 [Per-Environment Configuration Storage](#273-per-environment-configuration-storage)
     - 2.7.4 [Per-Environment Operations](#274-per-environment-operations)
3. [Access Control](#3-access-control)
   - 3.1 [Owner-Centric Role-Based Access Control (RBAC)](#31-owner-centric-role-based-access-control-rbac)
   - 3.2 [Permission Matrix](#32-permission-matrix)
   - 3.3 [Metadata Storage](#33-metadata-storage)
   - 3.4 [Audit Trail & History](#34-audit-trail--history)
4. [Template Upload & Validation](#4-template-upload--validation)
   - 4.1 [Template Update Workflow](#41-template-update-workflow)
5. [Configuration Creation & Management Workflow](#5-configuration-creation--management-workflow)
   - 5.1 [Template Upload](#51-template-upload)
   - 5.2 [Configuration Generation/Editing](#52-configuration-generationediting)
   - 5.3 [Configuration File Naming & Retrieval](#53-configuration-file-naming--retrieval)
   - 5.4 [User Operations](#54-user-operations)
   - 5.5 [Error Handling During Configuration Creation/Editing](#55-error-handling-during-configuration-creationediting)
   - 5.6 [Configuration Drift Detection & Invalid Value Handling](#56-configuration-drift-detection--invalid-value-handling)
     - 5.6.1 [Immediate Feedback at Edit Time](#561-immediate-feedback-at-edit-time)
     - 5.6.2 [Background Validation Service](#562-background-validation-service)
     - 5.6.3 [Persistent Failure Tracking](#563-persistent-failure-tracking)
     - 5.6.4 [Owner Notification Email](#564-owner-notification-email)
     - 5.6.5 [Config Administrator Impact Analysis](#565-config-administrator-impact-analysis)
     - 5.6.6 [Validation History & Audit Trail](#566-validation-history--audit-trail)
     - 5.6.7 [Config Administrator Escalation](#567-config-administrator-escalation)
     - 5.6.8 [Implementation Considerations](#568-implementation-considerations)
   - 5.7 [Multi-Environment Management](#57-multi-environment-management)
     - 5.7.1 [Environment Selection UI (Mandatory)](#571-environment-selection-ui-mandatory)
     - 5.7.2 [Copy/Promote Configuration Between Environments](#572-copypromote-configuration-between-environments)
     - 5.7.3 [Template List with Multi-Environment Status](#573-template-list-with-multi-environment-status)
6. [Implementation Logic: The Dynamic Form Builder](#6-implementation-logic-the-dynamic-form-builder)
   - 6.1 [Form Field Types & Validation Strategy](#61-form-field-types--validation-strategy)
   - 6.2 [Working with Grouped Parameters (Best Practices)](#62-working-with-grouped-parameters-best-practices)
     - 6.2.1 [Design Philosophy](#621-design-philosophy)
     - 6.2.2 [Recommended Naming Conventions](#622-recommended-naming-conventions)
     - 6.2.3 [Consuming Application Pattern](#623-consuming-application-pattern)
     - 6.2.4 [When NOT to Use DCCM](#624-when-not-to-use-dccm)
     - 6.2.5 [Benefits of This Approach](#625-benefits-of-this-approach)
7. [Technology Stack](#7-technology-stack)
8. [Configuration Template Example](#8-configuration-template-example)
   - 8.1 [The Configuration Template (Input)](#81-the-configuration-template-input)
   - 8.2 [Dynamic UI Generation and Validation](#82-dynamic-ui-generation-and-validation)
   - 8.3 [The Final Saved Configuration (Output)](#83-the-final-saved-configuration-output)
9. [UI Mockups & User Interface Guidelines](#9-ui-mockups--user-interface-guidelines)
   - 9.1 [Screen 1: Template List (Main Dashboard)](#91-screen-1-template-list-main-dashboard)
   - 9.2 [Screen 2: Upload Template](#92-screen-2-upload-template)
   - 9.3 [Screen 3: Generated Configuration Form (Dynamic Form Builder)](#93-screen-3-generated-configuration-form-dynamic-form-builder)
   - 9.4 [Screen 4: Manage Access (Owners & Editors)](#94-screen-4-manage-access-owners--editors)
   - 9.5 [Screen 5: Validation Errors](#95-screen-5-validation-errors)
   - 9.6 [Screen 6: Audit Trail (Config Administrators Only)](#96-screen-6-audit-trail-config-administrators-only)
   - 9.7 [UI Design Guidelines](#97-ui-design-guidelines)
10. [Critical Implementation Considerations](#10-critical-implementation-considerations)
    - 10.1 [The Three-Layer Assembly Process](#101-the-three-layer-assembly-process)
    - 10.2 [Concurrency & Race Conditions](#102-concurrency--race-conditions)
    - 10.3 [Additional Implementation Considerations](#103-additional-implementation-considerations)
11. [Implementation Effort & Timeline](#11-implementation-effort--timeline)
    - 11.1 [Effort Estimation Matrix](#111-effort-estimation-matrix)
    - 11.2 [The 4-Week Implementation Plan](#112-the-4-week-implementation-plan)
    - 11.3 [Why This Fits in 4 Weeks (The Accelerators)](#113-why-this-fits-in-4-weeks-the-accelerators)
    - 11.4 [Risk Factors (Where the Timeline Will Slip)](#114-risk-factors-where-the-timeline-will-slip)

---

## 1. Executive Summary

The **Dynamic Central Configuration Manager (DCCM)** is a self-service configuration platform where teams manage configs centrally and applications consume them via HTTP - eliminating configuration drift across infrastructure.

### Key Benefits

**Centralized Configuration, Decentralized Consumption:**
- Single source of truth eliminates configuration drift - no copying files across servers to keep them in sync

**Template-First Approach:**
- Upload a JSON/YAML schema and automatically get a validated form - no manual form building required

**Generic & Extensible Design:**
- Fits any use case without code changes - just upload a new template for database configs, service settings, feature flags, etc.

**Low Barrier to Entry:**
- Anyone can create configurations and become automatic owner - self-service without pre-authorization bottlenecks

**Built-in Validation & Error Prevention:**
- Type checking, range validation, regex patterns, and dropdown constraints prevent bad configurations before deployment

**Complete Audit Trail & Compliance:**
- Every change logged with full history and restore capability - compliance-ready for regulated industries

**Version Control & Rollback:**
- One-click rollback to any previous version provides production safety net without panic or manual recovery

**Team Collaboration:**
- Multiple owners and delegated editors enable shared responsibility without permission conflicts

**Security by Design:**
- Three-volume isolation, metadata protection, and SSO integration ensure enterprise-grade security

### Architecture

The system utilizes a two-tier architecture for optimal performance:
1.  **Management Tier (NiceGUI):** Handles complex validation and write operations at the central location.
2.  **Retrieval Tier (Company Web Server):** Handles simple, high-speed read operations via static file serving, enabling decentralized consumption by any application with HTTP access.

---

## 2. Architecture Overview: Two-Tier Model

The system is split into two functionally distinct, loosely coupled components that interact via **separate storage volumes**.

### 2.1 Component Separation

* **Management Tier (Write):** A single Python application running **NiceGUI**. This tier is accessed by administrators and editors and hosts the complex logic (RBAC, Validation, Form Generation).
* **Retrieval Tier (Read):** A standard, high-performance web server provided by the company (e.g., Apache). This tier provides applications with high-speed, **read-only** access to the configuration files via standard HTTP GET requests.

### 2.2 Storage Architecture (Filesystem Separation)

The system relies on **three distinct storage volumes (Mount Points)**. While they may reside on the same physical NAS/Storage Appliance, they must be mounted with distinct permissions.

| Volume | Path (Example) | App Permission | Web Svr Permission | Contents |
| :--- | :--- | :--- | :--- | :--- |
| **Management Vol** | `/mnt/dccm/mgmt` | Read/Write | None | Templates (`.yaml`/`.json`), Metadata (`.meta.json`), Environment registry, Lookup registry, Validation scripts registry |
| **Retrieval Vol** | `/mnt/dccm/public` | Read/Write | Read Only | Final `.json` / `.yaml` configs organized by environment subdirectories |
| **Audit Vol** | `/mnt/dccm/audit` | Append Only | None | History logs, validation history |

**Retrieval Volume Structure (Environment Subdirectories):**
```
/mnt/dccm/public/
├── dev/
│   ├── my-service.json
│   ├── api-gateway.json
│   └── database-config.json
├── test/
│   ├── my-service.json
│   ├── api-gateway.json
│   └── database-config.json
├── prod/
│   ├── my-service.json
│   ├── api-gateway.json
│   └── database-config.json
└── staging/      # Additional environments configured by admins
    ├── my-service.json
    └── api-gateway.json
```

**Management Volume Structure:**
```
/mnt/dccm/mgmt/
├── templates/
│   ├── my-service.yaml             # ONE template for ALL environments
│   ├── my-service.meta.json        # Tracks all environment configs
│   ├── api-gateway.yaml
│   └── api-gateway.meta.json
├── registries/
│   ├── environments.json           # System-wide environment list
│   ├── lookup_registry.json        # Lookup file mappings
│   └── validation_scripts.json     # Validation script mappings
├── lookups/
│   ├── support_groups.txt
│   ├── available_instances.txt
│   └── aws_regions.txt
└── .locks/
    ├── my-service_dev.lock         # Per-environment locks
    ├── my-service_test.lock
    └── my-service_prod.lock
```

**Critical Constraints:**
* Templates and metadata **must never** be accessible via the Retrieval Tier
* The Retrieval Volume must **only** contain final configuration output files organized by environment
* Audit trail must be on a completely separate volume with admin-only access
* No cross-contamination between volumes
* **App Permission "Append Only"** for Audit Volume ensures immutable audit trail
* Each environment has its own subdirectory in Retrieval Volume for clean separation
* **One template, multiple environment-specific configs**: A single template generates separate configuration files for each environment

### 2.3 Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          DCCM ARCHITECTURE                              │
└─────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────┐
│  USERS                                                                   │
│  ├── Owners (upload templates, manage configs)                           │
│  ├── Editors (update config values only)                                 │
│  └── Config Administrators (full control + audit access)                 │
└────────┬─────────────────────────────────────────────────────────────────┘
         │ SSO_USERNAME via environment variable
         ▼
┌──────────────────────────────────────────────────────────────────────────┐
│  MANAGEMENT TIER (NiceGUI/Python Application)                            │
│  ┌────────────────────────────────────────────────────────────────────┐  │
│  │ 1. Identity & Authorization Module                                 │  │
│  │    - Read SSO_USERNAME from environment                            │  │
│  │    - Check permissions against .meta.json                          │  │
│  └────────────────────────────────────────────────────────────────────┘  │
│  ┌────────────────────────────────────────────────────────────────────┐  │
│  │ 2. Dynamic Form Builder                                            │  │
│  │    - Parse template (YAML/JSON)                                    │  │
│  │    - Generate form fields (7 types)                                │  │
│  │    - Render searchable dropdowns from lookup_file                  │  │
│  └────────────────────────────────────────────────────────────────────┘  │
│  ┌────────────────────────────────────────────────────────────────────┐  │
│  │ 3. Data Validation Module                                          │  │
│  │    - Type checking (number, boolean, text)                         │  │
│  │    - Range validation (min/max)                                    │  │
│  │    - Pattern validation (regex)                                    │  │
│  └────────────────────────────────────────────────────────────────────┘  │
│  ┌────────────────────────────────────────────────────────────────────┐  │
│  │ 4. Persistence & Output Module                                     │  │
│  │    - Collision detection (owner-based)                             │  │
│  │    - Write to Retrieval Volume                                     │  │
│  │    - Log to Audit Volume                                           │  │
│  └────────────────────────────────────────────────────────────────────┘  │
└──────┬────────────┬────────────┬─────────────────────────────────────────┘
       │            │            │
       ▼            ▼            ▼
   ┌────────┐  ┌────────┐  ┌────────┐
   │ MGMT   │  │RETRIEV │  │ AUDIT  │  ◄── THREE SEPARATE FILESYSTEMS
   │ FS     │  │AL FS   │  │ FS     │
   └────────┘  └────────┘  └────────┘
   │           │           │
   │ Templates │ Config    │ History &
   │ .meta.json│ Output    │ Snapshots
               │ (.json/   │ (admin
               │  .yaml)   │  only)
               │           │
               │           │
               └───────────┤
                           ▼
                  ┌──────────────────┐
                  │  RETRIEVAL TIER  │
                  │ (Company Web Svr)│
                  └──────────────────┘
                           │
                           │ HTTP GET
                           ▼
                  ┌──────────────────┐
                  │  CONSUMING APPS  │
                  │  curl/scripts/   │
                  │  applications    │
                  └──────────────────┘
```

### 2.4 Data Flow

**Template Upload Flow:**
```
Owner → Upload Template → Validate → Store in Management FS
                                   → Create .meta.json
                                   → Log in Audit FS
```

**Configuration Creation Flow:**
```
User → Select Template → **SELECT ENVIRONMENT (MANDATORY)** → Generate Form
     → Fill Values → Validate → Write to Retrieval FS/{env}/ → Log in Audit FS
```

**Configuration Retrieval Flow:**
```
Dev App   → HTTP GET /dev/my-service.json   → Retrieval Tier → Return JSON/YAML
Test App  → HTTP GET /test/my-service.json  → Retrieval Tier → Return JSON/YAML
Prod App  → HTTP GET /prod/my-service.json  → Retrieval Tier → Return JSON/YAML
```

### 2.5 Internal Architecture (Management Flow)

The Management Tier ensures data integrity through a sequential, modular process upon form submission:

1.  **Identity & Authorization Module:** Retrieves the user's identity from the **`SSO_USERNAME`** environment variable and checks permissions against the saved Owner/Delegated Access List.
2.  **Data Integrity Validation Module:** Performs type checking, schema validation (`min`/`max`/`regex`), and resolves external **`lookup_file`** data.
3.  **Persistence & Output Module:** Runs the **Collision Detection** check and writes the final, validated configuration file to the **Retrieval Volume**.

### 2.6 System Bootstrap & Initial Access

To initialize the system from a fresh install (Day 0):

* **Initial Admin:** The environment variable **`DCCM_INITIAL_ADMIN`** must be set (e.g., `DCCM_INITIAL_ADMIN=alice.smith`).
* **First Login:** When this user logs in, they are automatically granted Config Administrator privileges.
* **Subsequent Admins:** The Initial Admin adds other administrators via the UI.

**Bootstrap Process:**
1. On application startup, check if any Config Administrators exist in the system
2. If no administrators exist AND `DCCM_INITIAL_ADMIN` is set:
   - Create the first Config Administrator with the specified SSO username
   - Log bootstrap action to Audit Volume: `"Initial admin created: alice.smith at 2025-11-19T10:00:00Z"`
3. If no administrators exist AND `DCCM_INITIAL_ADMIN` is NOT set:
   - Application refuses to start with error: `"FATAL: No administrators exist and DCCM_INITIAL_ADMIN not set"`
4. After successful bootstrap, `DCCM_INITIAL_ADMIN` can be removed from environment (optional)

**Validation:**
- If `DCCM_INITIAL_ADMIN` contains invalid characters (not a valid SSO username format), application refuses to start
- Bootstrap action is idempotent: If admin already exists, skip creation

### 2.7 Multi-Environment Architecture

**Core Principle:** ONE template, MULTIPLE environment-specific configurations.

**Problem Solved:** Enterprise systems need separate configurations for development, testing, and production environments. Creating separate templates (`my-service-dev`, `my-service-test`, `my-service-prod`) leads to duplication and drift.

**DCCM Solution:** A single template (e.g., `my-service.yaml`) generates multiple environment-specific configuration files (e.g., `/dev/my-service.json`, `/test/my-service.json`, `/prod/my-service.json`).

#### 2.7.1 Environment Registry

Config Administrators manage a **system-wide environment registry** that defines available environments for ALL templates.

**Registry Location:** `/mnt/dccm/mgmt/registries/environments.json`

**Example Environment Registry:**
```json
{
  "environments": [
    {
      "name": "dev",
      "display_name": "Development",
      "description": "Development environment for testing new features",
      "order": 1
    },
    {
      "name": "test",
      "display_name": "Testing",
      "description": "Integration testing environment",
      "order": 2
    },
    {
      "name": "staging",
      "display_name": "Staging",
      "description": "Pre-production staging environment",
      "order": 3
    },
    {
      "name": "prod",
      "display_name": "Production",
      "description": "Live production environment",
      "order": 4
    }
  ]
}
```

**Registry Management:**
- Config Administrators can add, remove, or reorder environments
- Environment `name` must be web-safe (lowercase alphanumeric, hyphens, underscores only)
- Environment `name` is used in URLs and file paths (`/mnt/dccm/public/{name}/`)
- Environment `display_name` is shown in UI
- Environment `order` determines display order in UI (ascending)
- Changes to registry affect ALL templates immediately

**Default Bootstrap Environments:**
- On first system startup, create default environments: `dev`, `test`, `prod`
- Admins can customize after bootstrap

#### 2.7.2 Environment Selection (Mandatory)

When creating or editing a configuration, users **MUST** select an environment. There is no default value, and the selection cannot be skipped.

**Critical Requirements:**
- Environment is a **required field** (form validation prevents save without selection)
- No default environment (user must explicitly choose)
- Environment is stored in **metadata**, NOT in template or configuration file
- Environment cannot be changed after creation (must copy to different environment instead)
- Each environment gets its own configuration file in its subdirectory

**UI Behavior:**
- Radio button list showing all available environments
- Visual warning for production environment: "⚠ You are configuring the PRODUCTION environment"
- Environment selection is first field in form (cannot be missed)

#### 2.7.3 Per-Environment Configuration Storage

**File Location Pattern:**
```
/mnt/dccm/public/{environment}/{template-name}.{format}
```

**Examples:**
- Dev config: `/mnt/dccm/public/dev/my-service.json`
- Test config: `/mnt/dccm/public/test/my-service.json`
- Prod config: `/mnt/dccm/public/prod/my-service.json`

**Retrieval URLs:**
- Dev: `http://config-host/dev/my-service.json`
- Test: `http://config-host/test/my-service.json`
- Prod: `http://config-host/prod/my-service.json`

**Web Server Configuration:**
- DocumentRoot: `/mnt/dccm/public/`
- No special configuration needed - subdirectories work automatically
- Standard static file serving

#### 2.7.4 Per-Environment Operations

**Independent Locking:**
- Editing dev config does not lock test or prod
- Lock files: `my-service_dev.lock`, `my-service_test.lock`, `my-service_prod.lock`

**Independent Validation:**
- Background validation runs separately for each environment
- Dev validation failure does not trigger prod notifications
- Each environment has its own validation history

**Independent Audit:**
- All audit log entries include environment field
- Query audit trail by environment: "Show all prod changes"
- Separate drift tracking per environment

---

## 3. Access Control

### 3.1 Owner-Centric Role-Based Access Control (RBAC)

The system uses an efficient, owner-centric model based on the user's **`SSO_USERNAME`**.

| Role | Privilege Level | Granting Mechanism |
| :--- | :--- | :--- |
| **Owner** | **Full Template & Configuration Control** | Initial uploader is automatically assigned as Owner. **Additional Owners can be added** by existing Owners. Multiple Owners are supported. **Must be saved as metadata.** |
| **Authorized User (Editor)** | **Update Configuration Values Only** | Granted via Owner delegation (Support Unit lookup or Manual List). Can modify configuration values through the form but **cannot upload or modify templates**. |
| **Config Administrator** | **Full Control (Update + Delete)** | Explicitly assigned role; can delete any template or configuration. |

### 3.2 Permission Matrix

| Operation | Owner | Authorized User (Editor) | Config Administrator |
| :--- | :---: | :---: | :---: |
| **Upload Template** | ✓ | ✗ | ✓ |
| **Modify Template** | ✓ | ✗ | ✓ |
| **Update Configuration Values** | ✓ | ✓ | ✓ |
| **Delete Template/Configuration** | ✗ | ✗ | ✓ |
| **Add Additional Owners** | ✓ | ✗ | ✓ |
| **Delegate to Editors** | ✓ | ✗ | ✓ |

### 3.3 Metadata Storage

Access control metadata (Owners, Editors, delegations) must be stored separately from configuration files:

* **Storage Requirement**: Metadata must persist across system restarts and be queryable for authorization checks
* **Metadata Content**: For each template, store:
  * List of Owner usernames (SSO_USERNAME values)
  * List of Authorized User (Editor) usernames or Support Unit references
  * Original uploader and upload timestamp
  * **Per-environment configuration metadata**: Created by, created at, last modified by, last modified at, locked by
* **Metadata Storage Implementation**: Use separate `.meta.json` files alongside each template
  * For template `my-service`, store metadata in `my-service.meta.json`
  * Store in the **Management Volume** alongside templates: `/mnt/dccm/mgmt/templates/my-service.meta.json`
  * **Critical**: Never store in Retrieval Volume - metadata must not be accessible via HTTP
* **Critical Constraint**: Metadata must **never** appear in configuration files served to consuming applications via the Retrieval Tier

**Metadata Structure (Multi-Environment):**

```json
{
  "template_name": "my-service",
  "owners": ["alice.smith", "charlie.admin"],
  "editors": ["bob.jones"],
  "created_by": "alice.smith",
  "created_at": "2025-11-20T10:00:00Z",
  "template_last_modified_by": "alice.smith",
  "template_last_modified_at": "2025-11-21T09:00:00Z",
  "environments": {
    "dev": {
      "created_by": "alice.smith",
      "created_at": "2025-11-20T10:00:00Z",
      "last_modified_by": "bob.jones",
      "last_modified_at": "2025-11-21T14:30:00Z",
      "locked_by": null,
      "locked_at": null
    },
    "test": {
      "created_by": "alice.smith",
      "created_at": "2025-11-20T10:15:00Z",
      "last_modified_by": "alice.smith",
      "last_modified_at": "2025-11-20T10:15:00Z",
      "locked_by": null,
      "locked_at": null
    },
    "prod": {
      "created_by": "charlie.admin",
      "created_at": "2025-11-20T11:00:00Z",
      "last_modified_by": "charlie.admin",
      "last_modified_at": "2025-11-20T11:00:00Z",
      "locked_by": "charlie.admin",
      "locked_at": "2025-11-21T16:00:00Z"
    }
  }
}
```

**Key Points:**
- **One metadata file per template** (not per environment)
- `owners` and `editors` apply to **all environments** (template-level permission)
- `environments` object tracks each environment's configuration metadata
- If environment config doesn't exist yet, it won't have an entry in `environments` object
- `locked_by` and `locked_at` enable per-environment exclusive locking
- Template modifications (`template_last_modified_*`) are separate from config modifications (per-environment `last_modified_*`)

### 3.4 Audit Trail & History

The system must maintain a complete audit trail for compliance and troubleshooting purposes:

* **Audit Storage Location**:
  * Must be stored on the **Audit Volume** (see section 2.2)
  * Completely separate from both Management Volume and Retrieval Volume
  * Only **Config Administrators** can access audit trail data
  * Not accessible to regular users or via HTTP
* **Template Change History**:
  * Every template upload/update must create a versioned history entry
  * Store: full template content, uploader username, timestamp, change description (optional)
  * Retain indefinitely (or per company retention policy)
* **Configuration Value Change History**:
  * Every configuration update must log the complete before/after state
  * Store: full configuration content (before and after), username, timestamp, changed fields
  * Track all value modifications, not just the latest version
* **Audit Trail Content Requirements**:
  * **Who**: SSO_USERNAME of the user making the change
  * **What**: Complete snapshot of template/configuration before and after change
  * **When**: Precise timestamp (ISO 8601 format recommended)
  * **Which**: Template/configuration name and affected fields
  * **Where** (NEW): Environment name (for configuration changes): `dev`, `test`, `prod`, etc.
* **Restore Capability**:
  * Config Administrators must be able to restore any previous version of a template
  * Config Administrators must be able to restore any previous version of a configuration
  * Restore operation itself must be logged in the audit trail
  * Restore UI must show diff between current and selected version before confirming
* **Access Control for Audit Trail**:
  * Regular users (Owners, Editors): No access to audit trail
  * Config Administrators: Read access to audit trail, restore capability
  * Audit trail queries should support filtering by: template name, username, date range, **environment**

**Audit Log Entry Example (Multi-Environment):**

```json
{
  "timestamp": "2025-11-21T14:30:15Z",
  "action": "configuration_updated",
  "template_name": "my-service",
  "environment": "prod",
  "user": "charlie.admin",
  "changes": {
    "service_timeout_ms": {
      "old": 3000,
      "new": 5000
    }
  },
  "config_before": {
    "service_timeout_ms": 3000,
    "debug_log_level": "INFO"
  },
  "config_after": {
    "service_timeout_ms": 5000,
    "debug_log_level": "INFO"
  }
}
```

**Additional Audit Actions for Multi-Environment:**

- `configuration_created`: User created new environment config (includes `environment` field)
- `configuration_updated`: User modified environment config (includes `environment` field)
- `configuration_copied`: User copied config from one environment to another (includes `source_environment` and `destination_environment`)
- `template_updated`: User modified template (no `environment` field - affects all environments)

---

## 4. Template Upload & Validation

* **Template Upload:** Users upload a JSON or YAML template file that defines the configuration schema and form structure.
  * **Important:** Uploading a template does NOT create any configuration files
  * Template defines the schema; configurations are created separately for each environment
  * After upload, owners must explicitly create configurations for desired environments (dev, test, prod, etc.)
* **Template Validation:** Before accepting the template, the system must perform comprehensive validation:
  * Verify valid JSON or YAML syntax
  * Check that all schema keywords are recognized (`min`, `max`, `regex`, `options`, `lookup_file`, `separator`, `multi_select`, `validate_script`, etc.)
  * Validate that `lookup_file` keywords exist in the lookup registry (see Lookup File Constraints below)
  * Validate that registered lookup files are readable and correctly formatted
  * Validate that `validate_script` keywords exist in the validation scripts registry
  * Validate that registered validation scripts are executable
  * Display **all validation errors** to the user in a clear, actionable format
  * Reject invalid templates and prevent form generation until errors are resolved
* **Lookup File Constraints (Predefined Keywords Only):**
  * **Keyword-Based System:** Template authors do NOT specify file paths. Instead, they use predefined keywords that map to admin-managed files.
  * **Example Valid Template Usage:**
    ```yaml
    support_teams:
      lookup_file: "support_teams"  # Keyword, not path

    file_instance:
      lookup_file: "instances"  # Keyword, not path
    ```
  * **Predefined Lookup Keywords:**
    - Config Administrators define a fixed set of available lookup keywords
    - Each keyword maps to a file in `/mnt/dccm/mgmt/lookups/`
    - Example mapping configuration (`/mnt/dccm/mgmt/lookup_registry.json`):
      ```json
      {
        "support_teams": "/mnt/dccm/mgmt/lookups/support_groups.txt",
        "instances": "/mnt/dccm/mgmt/lookups/available_instances.txt",
        "regions": "/mnt/dccm/mgmt/lookups/aws_regions.txt",
        "environments": "/mnt/dccm/mgmt/lookups/environments.txt"
      }
      ```
  * **Validation:**
    - Template validation checks if the `lookup_file` keyword exists in the registry
    - Reject templates with unrecognized keywords: "Error: 'unknown_keyword' is not a valid lookup. Available: support_teams, instances, regions, environments"
    - Validation ensures the mapped file exists and is readable
  * **Benefits of Keyword Approach:**
    - **Guaranteed file existence** - Admins ensure files exist before adding to registry
    - **Clear ownership** - Config Administrators are responsible for maintaining lookup files
    - **Format enforcement** - Admins ensure all files follow the correct format
    - **Security** - No path traversal risks, no arbitrary file access
    - **Simplified UX** - Template authors see dropdown of available keywords, no filesystem knowledge required
  * **Security:**
    - The Management Application must have **Read** access to files in `/mnt/dccm/mgmt/lookups/`
    - The Retrieval Web Server must **NOT** have access to these files (Management Volume is not accessible to web server)
    - The lookup registry file is only readable by the Management Application
  * **File Permissions:**
    - Files must be readable by the Management Application process user (e.g., `dccm-app`)
    - Recommended permissions: `644` (owner: root, group: dccm-app)
    - Files owned by root or system administrator, not modifiable by application
  * **Administrative Responsibility:**
    - Config Administrators add new lookup keywords via UI or configuration file
    - Admins ensure files are updated regularly (e.g., when new support teams are created)
    - System can provide UI to view/edit lookup file contents (Config Administrators only)

* **Validation Script Constraints (Predefined Keywords Only):**
  * **Keyword-Based System:** Template authors do NOT specify script paths. Instead, they use predefined keywords that map to admin-managed validation scripts.
  * **Example Valid Template Usage:**
    ```yaml
    assigned_user:
      label: "Assigned User"
      description: "Enter SSO username of person responsible"
      validate_script: "phone"  # Keyword that maps to admin-managed script

    budget_code:
      label: "Budget Code"
      description: "Enter active budget code for billing"
      validate_script: "budget_code"  # Keyword that maps to admin-managed script
    ```
  * **Predefined Validation Script Keywords:**
    - Config Administrators define a fixed set of available validation script keywords
    - Each keyword maps to an executable script/binary on the DCCM server
    - Example mapping configuration (`/mnt/dccm/mgmt/validation_scripts.json`):
      ```json
      {
        "phone": {
          "path": "/usr/local/bin/phone",
          "description": "Validate username exists in corporate phonebook",
          "timeout_seconds": 5
        },
        "budget_code": {
          "path": "/usr/local/bin/validate_budget_code.sh",
          "description": "Check if budget code is active in finance system",
          "timeout_seconds": 3
        },
        "project_code": {
          "path": "/usr/local/bin/validate_project.py",
          "description": "Verify project code exists in PMO system",
          "timeout_seconds": 5
        }
      }
      ```
  * **Script Execution Protocol:**
    - DCCM executes the script with user input as first argument: `/usr/local/bin/phone alice.smith`
    - **Exit Code 0**: Input is **valid** - validation passes
    - **Exit Code Non-Zero**: Input is **invalid** - validation fails
    - **stderr Output**: Optional error message to display to user (e.g., "User not found in phonebook")
    - **stdout Output**: Ignored by DCCM (scripts can output diagnostic info to stdout)
    - **Timeout**: Script must complete within configured timeout or validation fails
  * **Validation Timing:**
    - **Real-time validation (recommended)**: Trigger validation on field blur (when user leaves field)
    - **Pre-save validation (required)**: Always re-validate all fields before saving configuration
    - **Error Display**: Show validation errors inline below the field
  * **Example Error Messages:**
    - Script exits 0: Field appears valid (green checkmark or no error)
    - Script exits 1 with stderr "User not found": Display "❌ User not found"
    - Script exits 2 with stderr "Budget code expired": Display "❌ Budget code expired"
    - Script timeout: Display "❌ Validation timeout - contact administrator"
    - Script not found: Display "❌ Validation unavailable - contact administrator"
  * **Security:**
    - The Management Application must have **Execute** permission on validation scripts
    - Scripts must be owned by root or system administrator, not modifiable by application
    - Scripts run with DCCM application user privileges (not root)
    - Input sanitization: Escape/validate user input before passing to script to prevent command injection
    - Never use shell expansion - pass arguments directly to script (e.g., `subprocess.run([script_path, user_input])`)
  * **Script Permissions:**
    - Scripts must be executable by the Management Application process user (e.g., `dccm-app`)
    - Recommended permissions: `755` (owner: root, executable by all)
    - Scripts should be in protected directory (e.g., `/usr/local/bin/`, not user-writable paths)
  * **Benefits of Keyword Approach:**
    - **Reuse existing infrastructure** - leverage scripts that already exist in the organization
    - **Flexible validation** - any check that can be scripted (LDAP, database, API calls)
    - **No DCCM code changes** - add new validation types without modifying application
    - **Security** - no path injection risks, controlled script execution
    - **Simplified UX** - template authors see dropdown of available validators
  * **Administrative Responsibility:**
    - Config Administrators add new validation script keywords via configuration file
    - Admins ensure scripts are installed, executable, and return correct exit codes
    - Admins set appropriate timeouts based on script performance
    - System should provide UI to test validation scripts (Config Administrators only)

* **Environment Registry (System-Wide Configuration):**
  * **Purpose:** Defines available environments for ALL templates (dev, test, staging, prod, etc.)
  * **Registry Location:** `/mnt/dccm/mgmt/registries/environments.json`
  * **Structure:** See Section 2.7.1 for complete details
  * **Management:** Config Administrators can add, remove, or reorder environments
  * **Validation:**
    - Environment names must be web-safe (lowercase alphanumeric, hyphens, underscores only)
    - Environment names must be unique
    - At least one environment must be defined
  * **Bootstrap:** On first startup, create default environments: `dev`, `test`, `prod`
  * **Impact:** Changes to environment registry affect all templates immediately
    - Adding environment: New environment appears in dropdown for all templates
    - Removing environment: Confirm no configurations exist for that environment before removal
    - Renaming environment: Not supported (must create new and migrate configs manually)

* **Error Handling During Template Validation:**
  * **Invalid JSON/YAML syntax**: Display syntax error with line number
  * **Unrecognized schema keyword**: List all unrecognized keywords
  * **Invalid `lookup_file` keyword**: Report which lookup keywords are not registered
    - Example: "Error: 'unknown_list' is not a valid lookup keyword. Available: support_teams, instances, regions, environments"
  * **Missing lookup file**: If registered keyword maps to non-existent file, report error
    - Example: "Error: Lookup 'instances' is registered but file '/mnt/dccm/mgmt/lookups/available_instances.txt' does not exist or is not readable"
  * **Invalid `validate_script` keyword**: Report which validation script keywords are not registered
    - Example: "Error: 'unknown_validator' is not a valid validation script. Available: phone, budget_code, project_code"
  * **Missing validation script**: If registered keyword maps to non-existent or non-executable script, report error
    - Example: "Error: Validation script 'phone' is registered but '/usr/local/bin/phone' does not exist or is not executable"
  * **Multiple errors**: Display all errors at once (not just the first error)
* **Template Naming & Collision Handling:** Templates are saved with a user-provided name. **The template name becomes the configuration filename.**
  * Maximum length: **50 characters**
  * Allowed characters: **Web-safe characters only** (alphanumeric, hyphens, underscores: `a-z`, `A-Z`, `0-9`, `-`, `_`)
  * No spaces or special characters that require URL encoding (avoids `%20` in curl requests)
  * The system must validate input and reject non-compliant names
  * If a template with the same name exists:
    * **Any Owner or Config Administrator**: Can overwrite the existing template
    * **Authorized Users (Editors)**: Cannot upload or overwrite templates
    * **Others**: Must choose a different name

### 4.1 Template Update Workflow

When an Owner or Config Administrator updates an existing template:

* **Impact on Existing Configurations**:
  * Existing configuration files remain unchanged until manually updated by a user
  * The system does **not** automatically migrate or re-validate existing configurations
  * Users will see the new template schema when they next edit the configuration
* **Breaking Changes**:
  * If the updated template removes fields or changes validation rules, users may encounter errors when updating existing configurations
  * The system should display clear validation errors indicating which fields no longer conform to the template
  * **Recommendation to Owner**: Test template changes with a different template name before overwriting production templates
* **Audit Trail**:
  * Every template update is logged in the audit trail (see section 3.4)
  * Config Administrators can restore previous template versions if needed
  * Template version history is maintained in the audit trail, not in the main application

---

## 5. Configuration Creation & Management Workflow

**Important Distinction**: Uploading a template does NOT automatically create a configuration file. These are two separate operations:

### 5.1 Template Upload
* Owner or Config Administrator uploads a template (schema definition)
* Template is validated and stored
* **No configuration file is created yet**

### 5.2 Configuration Generation/Editing
* After a template exists, users can create or edit the configuration:
  * **Create New Configuration**: User fills out the form generated from template for the first time
  * **Edit Existing Configuration**: User modifies values in existing configuration
* The form is dynamically generated based on the current template (read from Management Volume)
* Upon save, the validated configuration file is written to the **Retrieval Volume** for consumption by applications

### 5.3 Configuration File Naming & Retrieval

* **Configuration Naming (Multi-Environment):** The configuration filename is **derived from the template name** - users cannot choose a different name.
  * Template name: `my-service`
  * Dev config: `/mnt/dccm/public/dev/my-service.json`
  * Test config: `/mnt/dccm/public/test/my-service.json`
  * Prod config: `/mnt/dccm/public/prod/my-service.json`
  * Environment is determined by **mandatory user selection during creation**, NOT by filename
* **File Format:** The user specifies the desired output format (JSON or YAML). The file extension is automatically added based on the selected format.
* **Retrieval (Environment-Specific URLs):** Consuming applications fetch the file from environment-specific subdirectories:
  * Dev application: `curl http://config-host/dev/my-service.json`
  * Test application: `curl http://config-host/test/my-service.json`
  * Prod application: `curl http://config-host/prod/my-service.json`
* **Application Configuration:**
  * Each application environment must be configured with the correct base URL
  * Dev app: `CONFIG_BASE_URL=http://config-host/dev/`
  * Test app: `CONFIG_BASE_URL=http://config-host/test/`
  * Prod app: `CONFIG_BASE_URL=http://config-host/prod/`
  * Applications append template name to base URL: `${CONFIG_BASE_URL}my-service.json`

### 5.4 User Operations

The UI must support these distinct operations:

1. **Upload Template** (Owners/Admins only): Upload/update template schema (affects all environments)
2. **Create Configuration for Environment** (Owners/Editors): Select environment (mandatory), fill out form to create initial configuration file for that environment
3. **Edit Configuration for Environment** (Owners/Editors): Select environment, modify existing configuration values for that specific environment
4. **Copy Configuration Between Environments** (Owners/Editors): Copy config from source environment to destination environment (see Section 5.7.2)
5. **View Template** (Owners/Editors): View the current template schema (read-only for Editors)
6. **Delete Template** (Admins only): Remove template and ALL associated configurations across ALL environments
7. **Delete Environment Configuration** (Admins only): Remove configuration for specific environment only

### 5.5 Error Handling During Configuration Creation/Editing

* **`lookup_file` disappeared**: If a lookup keyword's mapped file no longer exists when creating/updating a configuration, display error and prevent saving
  - Example: "Error: Cannot load lookup 'instances' - admin must restore the file"
* **Lookup keyword removed from registry**: If a template uses a keyword that was removed from the registry, display error
  - Example: "Error: Lookup 'deprecated_list' is no longer available. Contact Config Administrator."
* **Validation failure**: Display specific field validation errors (e.g., "service_timeout_ms must be between 1000 and 15000")
* **Permission denied**: If user lacks permission to update configuration, display clear error message indicating they need Owner or Editor access
* **No template exists**: If user tries to create/edit configuration but template was deleted, display error and prevent operation

### 5.6 Configuration Drift Detection & Invalid Value Handling

**Problem:** Configuration values can become invalid over time due to changes in lookup files, validation scripts, or template updates. For example, a support team is renamed, a budget code expires, or a user leaves the company.

**Solution:** Multi-layered drift detection and notification system that distinguishes between legitimate failures and transient issues.

#### 5.6.1 Immediate Feedback at Edit Time

When an Owner or Editor opens a configuration with now-invalid values:

**UI Behavior:**
```
┌────────────────────────────────────────────────────────────────────┐
│ Edit Configuration: my-service                   User: alice.smith │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  ⚠ VALIDATION NOTICE                                               │
│  1 field contains a value that is no longer valid according to     │
│  the current template. You must update it before saving.           │
│  [View Details]                                                    │
│                                                                    │
│  ────────────────────────────────────────────────────────────────  │
│                                                                    │
│  Support Team                                                      │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ ❌ Current value: "DevOps-Legacy" (NO LONGER AVAILABLE)      │  │
│  │                                                              │  │
│  │ Please select new value:                                     │  │
│  │ ● DevOps-Platform                                            │  │
│  │ ○ DevOps-Infrastructure                                      │  │
│  │ ○ DevOps-Security                                            │  │
│  │                                                              │  │
│  │ 💡 Hint: "DevOps-Platform" is the successor to               │  │
│  │    "DevOps-Legacy" (renamed on 2025-10-15)                   │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  Service Timeout (ms)                                              │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ 5000                                                         │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  [Cancel]                                          [Save]          │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

**Requirements:**
- Clearly indicate which field(s) have invalid values
- Show current invalid value vs available options
- Display migration hint if Config Admin provided one (e.g., "DevOps-Legacy → DevOps-Platform")
- Prevent saving until all invalid fields are updated
- Load the configuration in edit mode (don't block access)
- Other valid fields remain editable

#### 5.6.2 Background Validation Service

**Purpose:** Proactively detect configurations with invalid values before owners try to edit them.

**Implementation:**
- Run daily validation scan (recommended: 02:00 AM)
- Validate ALL configurations against their current templates
- **Per-Environment Validation:** Validate each environment's configuration separately
  - Dev config validated independently from test and prod
  - Each environment has its own validation history
  - Failures in one environment don't affect notifications for other environments
- Track validation history to distinguish persistent failures from transient issues
- Send notifications only for persistent failures (3+ consecutive days)

**Validation Result Types:**

| Status | Meaning | Action |
| :--- | :--- | :--- |
| `success` | Value passes validation | No action |
| `failed` | Value legitimately invalid (e.g., not in lookup list, script returns non-zero) | Track failure, notify if persistent |
| `unavailable` | Validation cannot run (script down, file missing, timeout) | Log issue, do NOT count as failure, do NOT notify owners |

**Critical Distinction:**
- **Failed validation** = Data problem (value no longer valid) → Notify owner
- **Unavailable validation** = System problem (script timeout, file missing) → Alert admin, NOT owner

#### 5.6.3 Persistent Failure Tracking

**Algorithm:**
```
For each configuration:
  1. Run validation against current template
  2. If validation returns 'unavailable':
     - Log to audit trail: "Validation unavailable for config X, field Y"
     - Do NOT increment failure counter
     - Do NOT send notification
  3. If validation returns 'failed':
     - Record failure with timestamp and failed fields
     - Check consecutive days failed
     - If >= 3 consecutive days AND no 'unavailable' in last 3 days:
       - Send notification if not already notified this week
  4. If validation returns 'success':
     - Reset failure counter
```

**False Positive Prevention:**
- **3-day threshold:** Prevents alerts for temporary issues
- **Unavailable reset:** If validation was unavailable recently, restart the 3-day clock
- **Weekly frequency:** After initial alert, notify weekly (not daily)
- **Escalation:** After 4 notifications (1 month), escalate to Config Administrator

#### 5.6.4 Owner Notification Email

**Trigger Conditions:**
- Configuration has failed validation for 3+ consecutive days
- No 'unavailable' status in last 3 days (ensures legitimate failure)
- At least 7 days since last notification (weekly frequency)
- Fewer than 4 notifications already sent (then escalate to admin)

**Email Template:**
```
Subject: [DCCM] Action Required: Configuration "my-service" (prod) has invalid values

Hi Alice Smith, Charlie Admin (Owners of "my-service"),

Your configuration "my-service" in the PRODUCTION environment has values
that are no longer valid according to the current template. This has been
detected for 3 consecutive days.

INVALID FIELDS:
┌────────────────────────────────────────────────────────────────┐
│ Field: support_team                                            │
│ Current Value: "DevOps-Legacy"                                 │
│ Issue: Value not in approved list                              │
│ Available Options: DevOps-Platform, DevOps-Infrastructure, ... │
│                                                                │
│ Suggested Fix: Update to "DevOps-Platform"                     │
│ (DevOps-Legacy was renamed to DevOps-Platform on 2025-10-15)   │
└────────────────────────────────────────────────────────────────┘

NEXT STEPS:
1. Click here to edit configuration: https://dccm.company.com/edit/my-service?env=prod
2. Update the invalid field(s)
3. Save the configuration

This notification will repeat weekly until resolved.

Environment: prod
Last successful validation: 2025-11-17 (4 days ago)
Configuration last modified: 2025-10-01 by alice.smith

NOTE: Other environments (dev, test) may have different values and are
validated separately. This notification is specific to the prod environment.

---
Questions? Contact Config Administrators or reply to this email.
```

**Notification Frequency Policy:**
- **Initial delay:** 3 days of consecutive failure
- **Frequency:** Weekly after initial notification
- **Maximum:** 4 notifications (4 weeks total)
- **Escalation:** After 4 notifications, alert Config Administrator about stale configuration

#### 5.6.5 Config Administrator Impact Analysis

Before removing entries from lookup files or changing validation rules, show impact:

**Lookup File Editor UI:**
```
┌────────────────────────────────────────────────────────────────────┐
│ Lookup File Editor: support_teams                                  │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  Current Values:                                                   │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ ✓ DevOps-Platform               (Used in 45 configs)         │  │
│  │ ✓ DevOps-Infrastructure         (Used in 23 configs)         │  │
│  │ ⚠ DevOps-Legacy                 (Used in 12 configs)         │  │
│  │                                                              │  │
│  │   ⚠ WARNING: Removing "DevOps-Legacy" will cause           │  │
│  │      validation failures in 12 active configurations.       │  │
│  │                                                              │  │
│  │   Affected configs:                                          │  │
│  │   - my-service (alice.smith)                                 │  │
│  │   - api-gateway (bob.jones)                                  │  │
│  │   - payment-processor (charlie.admin)                        │  │
│  │   ... and 9 more                                             │  │
│  │                                                              │  │
│  │   Migration Hint (shown to owners):                          │  │
│  │   ┌────────────────────────────────────────────────────────┐ │  │
│  │   │ "DevOps-Legacy" was renamed to "DevOps-Platform"       │ │  │
│  │   │ on 2025-10-15                                          │ │  │
│  │   └────────────────────────────────────────────────────────┘ │  │
│  │                                                              │  │
│  │   [✓] Notify all affected owners before removal             │  │
│  │                                                              │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  [Cancel]                           [Remove with Notification]     │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

**Requirements:**
- Before removing lookup value, scan all configs using that lookup
- Show count of affected configurations
- List affected config names and owners
- Allow admin to add migration hint that will be shown to owners
- Send immediate notification to affected owners (don't wait 3 days)

**Pre-removal Notification Email:**
```
Subject: [DCCM] Configuration Update Required: "DevOps-Legacy" being removed

Hi Alice Smith,

The Config Administrator is removing "DevOps-Legacy" from the support_teams
lookup list. Your configuration "my-service" currently uses this value.

IMPACT:
- Your configuration will become invalid after 2025-11-25 (7 days)
- You will not be able to edit it until you update the support_team field

RECOMMENDED ACTION:
Update "DevOps-Legacy" to "DevOps-Platform" (renamed on 2025-10-15)

Click here to edit now: https://dccm.company.com/edit/my-service

Questions? Reply to this email or contact the Config Administrator.
```

#### 5.6.6 Validation History & Audit Trail

**Storage Requirements:**
- Store validation results in Audit Volume: `/mnt/dccm/audit/validation_history/`
- Track for each configuration:
  - Validation timestamp
  - Validation status (success, failed, unavailable)
  - Failed fields and reasons
  - Notification sent (yes/no, timestamp)

**Log Entry Format (Per-Environment):**
```json
{
  "timestamp": "2025-11-20T02:00:15Z",
  "config_name": "my-service",
  "environment": "prod",
  "validation_status": "failed",
  "failed_fields": [
    {
      "field": "support_team",
      "current_value": "DevOps-Legacy",
      "reason": "Value not in approved list",
      "validation_type": "lookup_file",
      "lookup_keyword": "support_teams"
    }
  ],
  "notification_sent": false,
  "consecutive_failures": 1
}
```

**Key Point:** Each environment's validation history is tracked separately. Dev failure doesn't increment prod's consecutive_failures counter.

#### 5.6.7 Config Administrator Escalation

After 4 weeks of notifications without resolution, escalate to Config Administrator:

**Escalation Email:**
```
Subject: [DCCM Admin] Stale Configuration: "my-service" (prod) invalid for 28 days

Configuration "my-service" in the PRODUCTION environment has had invalid
values for 28 days despite 4 owner notifications.

DETAILS:
- Template: my-service
- Environment: prod
- Owners: alice.smith, charlie.admin
- Invalid Field: support_team
- Current Value: "DevOps-Legacy" (not in approved list)
- Last Modified: 2025-10-01 by alice.smith
- First Detected: 2025-10-30
- Notifications Sent: 4 (last: 2025-11-24)

ENVIRONMENT STATUS:
- dev: Valid (last validated: 2025-11-25)
- test: Valid (last validated: 2025-11-25)
- prod: INVALID (consecutive failures: 28 days)

RECOMMENDED ACTIONS:
1. Contact owners directly to resolve
2. If configuration is no longer used, delete it
3. If value should be re-added to lookup, update lookup file
4. If urgent, fix configuration on behalf of owners (logged in audit trail)
5. Consider copying from dev or test if those are valid

View configuration: https://dccm.company.com/admin/view/my-service?env=prod
Validation history: https://dccm.company.com/admin/audit/my-service?env=prod
```

**Admin Actions:**
- View configuration (read-only or editable by admin)
- Delete configuration if abandoned
- Force-fix configuration (logged as "Fixed by admin on behalf of owner")
- Re-add value to lookup if removal was premature

#### 5.6.8 Implementation Considerations

**Validation Script Timeouts:**
- If validation script times out, mark as 'unavailable' (not 'failed')
- Log timeout to audit trail for admin troubleshooting
- Do not notify owners about script timeouts
- Alert Config Administrator if same script times out repeatedly

**Lookup File Missing:**
- If lookup file is temporarily missing, mark as 'unavailable'
- Do not notify owners
- Alert Config Administrator immediately

**Performance:**
- Background validation should run during low-usage hours (02:00 AM recommended)
- Batch validation requests to avoid overwhelming external systems
- Rate-limit validation script calls (e.g., max 10/second for phone script)
- Cache lookup file contents for duration of validation run

**Migration Hints Storage:**
When Config Administrator removes a lookup value, store migration hint in metadata:
```json
{
  "lookup_keyword": "support_teams",
  "deprecated_values": {
    "DevOps-Legacy": {
      "removed_date": "2025-11-25",
      "removed_by": "admin.user",
      "migration_hint": "DevOps-Legacy was renamed to DevOps-Platform on 2025-10-15",
      "suggested_replacement": "DevOps-Platform"
    }
  }
}
```

This hint is displayed to owners when they encounter the invalid value.

### 5.7 Multi-Environment Management

This section describes operations specific to managing configurations across multiple environments.

#### 5.7.1 Environment Selection UI (Mandatory)

When creating or editing a configuration, environment selection is the **first required field** in the form.

**UI Requirements:**
```
┌────────────────────────────────────────────────────────────────────┐
│ Create Configuration: my-service                 User: alice.smith │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  Environment * (Required - Select Target Environment)              │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                                                              │  │
│  │  ○ dev         Development                                   │  │
│  │                Development environment for testing           │  │
│  │                                                              │  │
│  │  ○ test        Testing                                       │  │
│  │                Integration testing environment               │  │
│  │                                                              │  │
│  │  ○ staging     Staging                                       │  │
│  │                Pre-production staging environment            │  │
│  │                                                              │  │
│  │  ○ prod        Production                                    │  │
│  │                Live production environment                   │  │
│  │                                                              │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  [Continue to Configuration Form]                                  │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

**Validation Rules:**
- Environment field is **required** - form cannot proceed without selection
- No default value - user must explicitly click a radio button
- Visual warning if `prod` is selected: "⚠ You are configuring the PRODUCTION environment"
- Once created, environment cannot be changed (must copy to different environment instead)

**When Editing Existing Configuration:**
- Environment is pre-selected and **read-only** (displayed at top of form)
- User cannot change environment during edit
- To move config to different environment, must use Copy operation (see 5.7.2)

#### 5.7.2 Copy/Promote Configuration Between Environments

Owners and Editors can copy configurations from one environment to another (e.g., promote dev → test → prod).

**Copy Operation UI:**
```
┌────────────────────────────────────────────────────────────────────┐
│ Copy Configuration: my-service                   User: alice.smith │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  Source Environment: *                                             │
│  ● dev       (Last modified: 2025-11-21 14:30 by bob.jones)        │
│  ○ test      (Last modified: 2025-11-20 10:15 by alice.smith)      │
│  ○ staging   (Not yet configured)                                  │
│  ○ prod      (Last modified: 2025-11-20 11:00 by charlie.admin)    │
│                                                                    │
│  Destination Environment: *                                        │
│  ○ dev                                                             │
│  ● test      (Last modified: 2025-11-20 10:15 by alice.smith)      │
│  ○ staging   (Not yet configured)                                  │
│  ○ prod      (Last modified: 2025-11-20 11:00 by charlie.admin)    │
│                                                                    │
│  ⚠ WARNING: This will OVERWRITE the existing test configuration    │
│                                                                    │
│  Current values in test will be replaced with values from dev.     │
│  This action is logged in the audit trail.                         │
│                                                                    │
│  [Cancel]                              [Copy dev → test]           │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

**Copy Operation Behavior:**
- Select source environment (which config to copy FROM)
- Select destination environment (which config to copy TO)
- If destination already has a config, show warning about overwrite
- If destination is empty, show info: "This will create a new configuration in test"
- Copy operation creates audit log entry with both source and destination
- User who performs copy becomes the `last_modified_by` for destination environment

**Audit Log Entry for Copy:**
```json
{
  "timestamp": "2025-11-21T15:00:00Z",
  "action": "configuration_copied",
  "template_name": "my-service",
  "source_environment": "dev",
  "destination_environment": "test",
  "user": "alice.smith",
  "overwrite": true
}
```

**Access Control:**
- Owners and Editors can copy between ANY environments
- Copy operation respects template-level permissions (must be Owner or Editor)
- No environment-specific copy restrictions (trusted users)

#### 5.7.3 Template List with Multi-Environment Status

The main dashboard shows status of each environment for each template.

**Template List UI:**
```
┌────────────────────────────────────────────────────────────────────┐
│ DCCM - Configuration Manager              User: alice.smith        │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  Templates                                    [+ Upload Template]  │
│  ────────────────────────────────────────────────────────────────  │
│                                                                    │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ my-service                                      Owner        │  │
│  │ Production microservice configuration                        │  │
│  │                                                              │  │
│  │ Environments:                                                │  │
│  │   ✓ dev    Last modified: 2025-11-21 14:30 (bob.jones)      │  │
│  │   ✓ test   Last modified: 2025-11-20 10:15 (alice.smith)    │  │
│  │   ⚠ staging  Not yet configured                             │  │
│  │   ✓ prod   Last modified: 2025-11-20 11:00 (charlie.admin)  │  │
│  │            🔒 Currently locked by charlie.admin               │  │
│  │                                                              │  │
│  │ [Edit dev] [Edit test] [Create staging] [Edit prod]         │  │
│  │ [Copy Between Environments] [Manage Access]                 │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ api-gateway                                     Owner        │  │
│  │ API Gateway configuration                                    │  │
│  │                                                              │  │
│  │ Environments:                                                │  │
│  │   ✓ dev    Last modified: 2025-11-19 16:00 (alice.smith)    │  │
│  │   ⚠ test     Not yet configured                             │  │
│  │   ⚠ staging  Not yet configured                             │  │
│  │   ✓ prod   Last modified: 2025-11-18 09:00 (charlie.admin)  │  │
│  │                                                              │  │
│  │ [Edit dev] [Create test] [Create staging] [Edit prod]       │  │
│  │ [Copy Between Environments] [Manage Access]                 │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

**Status Indicators:**
- ✓ Environment configured (show last modified info)
- ⚠ Not yet configured
- 🔒 Currently locked by user (editing in progress)

**Action Buttons:**
- **Edit {env}**: Opens edit form for that environment (if config exists)
- **Create {env}**: Opens create form for that environment (if config doesn't exist)
- **Copy Between Environments**: Opens copy UI
- **Manage Access**: Opens permission management (same permissions for all environments)

---

## 6. Implementation Logic: The Dynamic Form Builder

The core logic is the recursive parser that allows configuration structure and validation to be defined entirely within the JSON or YAML template.

### 6.1 Form Field Types & Validation Strategy

**Supported Field Types:**

The system supports the following input field types, determined by template configuration:

| Field Type | When Used | UI Component | Example Use Case |
| :--- | :--- | :--- | :--- |
| **Text Input** | Default (no keywords) or `type: 'string'` | Single-line text field | Service name, description |
| **Number Input** | `type: 'integer'` or `type: 'float'` | Numeric input with validation | Timeout, port number, retry count |
| **Text Area** | `type: 'textarea'` or `multiline: true` | Multi-line text input | Long descriptions, JSON snippets |
| **Dropdown (Standard)** | `options` provided (< 10 items) | Standard dropdown | Region selection (3-5 choices) |
| **Dropdown (Searchable)** | `options` provided (≥ 10 items) OR `lookup_file` | Searchable/autocomplete dropdown | Instance selection (hundreds of choices) |
| **Multi-Select** | `multi_select: true` with `options` or `lookup_file` | Multi-select dropdown | Multiple teams, tags |
| **Checkbox** | `type: 'boolean'` | Checkbox | Enable/disable feature flags |

**Field Rendering Logic:**

1. **Explicit type**: If `type` keyword is present, use it (e.g., `type: 'boolean'`)
2. **Type inference**: Infer from `value` (string → text, number → number input, boolean → checkbox)
3. **Dropdown mode**: If `options` or `lookup_file` present, render as dropdown
   - **Searchable Dropdown:** Renders if `options` count is **≥ 10** OR if `lookup_file` is used
   - **Standard Dropdown:** Renders if `options` count is **< 10** (9 items or fewer)
4. **Multi-Select mode**: If `multi_select: true` is set:
   - Allows selecting multiple values from dropdown
   - Output must be an **Array of Strings** in the configuration file
5. **Default**: Text input

**Multi-Select Output Format:**

When `multi_select: true` is used, the saved configuration must output an array:

*JSON Example:*
```json
{
  "support_teams": ["IT-Support", "DevOps-Team", "QA-Team"]
}
```

*YAML Example:*
```yaml
support_teams:
  - IT-Support
  - DevOps-Team
  - QA-Team
```

**Schema Keywords:**

| Schema Key | Functionality | Values |
| :--- | :--- | :--- |
| `type` | **Strict Data Typing** | `'string'` (default), `'integer'`, `'float'`, `'boolean'`, `'textarea'` |
| `label` | **Form Field Label** | Any string. Defaults to field key name if not specified. |
| `description` | **Help Text** | Any string. Displayed below or next to field. |
| `value` | **Default/Initial Value** | Any value matching the field type. Used for type inference if `type` not specified. |
| `multiline` | **Multi-line Text** | `true` or `false` (default). When `true`, renders a text area instead of single-line text input. |
| `min` / `max` | **Range Validation** | Integer/Float values. Only applies if `type` is `'integer'` or `'float'`. |
| `regex` | **Pattern Validation** | Regular expression string. Only applies to string fields. |
| `options` | **Predefined Dropdown** | Array of strings. Renders dropdown. Searchable if ≥ 10 items. |
| `lookup_file` | **External File Dropdown** | Predefined keyword (e.g., `"support_teams"`, `"instances"`). Maps to admin-managed file. Always renders as searchable dropdown. |
| `separator` | **File Column Delimiter** | Single character (e.g., `;`, `,`, `\t`). Required if `lookup_file` has multiple columns. |
| `column` | **Column Index** | Integer (0-indexed). Specifies which column to use from delimited file. Default: `0`. |
| `multi_select` | **Enable Multi-Select** | `true` or `false` (default). Allows selecting multiple values with `lookup_file` or `options`. |
| `validate_script` | **External Validation Script** | Predefined keyword (e.g., `"phone"`, `"budget_code"`). Maps to admin-managed validation script. DCCM executes script with user input as argument. Exit code 0 = valid, non-zero = invalid with optional error message on stderr. |

**Note:** All keywords are optional. The absolute minimum valid field definition is just a field key with no keywords - it renders as a free text input.

**Important Limitation - No Hierarchical/Nested Configurations:**
- The system supports **flat** configuration structures only
- All fields must be at the top level of the template
- **No nested objects** or hierarchical structures are supported
- Example of what is NOT supported:
  ```yaml
  database:           # ✗ NOT SUPPORTED
    host: localhost
    port: 5432
    credentials:
      username: admin
      password: secret
  ```
- All configuration fields must be direct properties:
  ```yaml
  database_host: localhost      # ✓ SUPPORTED
  database_port: 5432
  database_username: admin
  database_password: secret
  ```

### 6.2 Working with Grouped Parameters (Best Practices)

**Problem:** How do you handle repeating groups of related parameters (e.g., multiple firewall rules, database connections, backup schedules)?

**Answer:** Use **flat structure with naming conventions**. The consuming application parses and groups the fields.

#### 6.2.1 Design Philosophy

**DCCM is for CONFIGURATION, not DATA:**
- ✅ **5-10 grouped parameters:** Perfect for DCCM with naming convention
- ⚠️ **20-50 items:** Consider if DCCM is the right tool
- ❌ **100+ dynamic items:** Use database or specialized config management tool

**Why Flat Structure:**
- Maintains DCCM's simplicity (no dynamic UI complexity)
- Clear scope boundary
- No implementation bloat
- Works for 80% of use cases

#### 6.2.2 Delimiter Convention for Grouped Parameters

**IMPORTANT: Use Double Colon `::` as the Delimiter**

When creating grouped parameters, **always use double colon `::` as the delimiter** between the group identifier (prefix) and the field name.

**Why Double Colon?**
- ✅ **Explicit grouping signal** - When you see `::`, you immediately know it's a grouped parameter
- ✅ **Unambiguous parsing** - Clear boundary between prefix and field name
- ✅ **Preserves underscores** - Single underscores remain available for normal field names (e.g., `max_retries`, `db_host`)
- ✅ **Visual distinction** - Easy to distinguish `rule1::name` (grouped) from `database_host` (standalone)

**Comparison:**
```yaml
# ❌ UNCLEAR: Is this "primary" + "db_host" or "primary_db" + "host"?
primary_db_host:

# ✅ CLEAR: This is the "host" field of the "primary_db" group
primary_db::host:
```

**Pattern 1: Numbered Groups (Fixed Count)**

Use when you have a **known, fixed number** of groups:

**Template Example - Firewall Rules:**
```yaml
# Rule 1
rule1::name:
  label: "Rule 1: Name"
  description: "Descriptive name for this firewall rule"

rule1::source:
  label: "Rule 1: Source IP/CIDR"
  regex: "^\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}(/\\d{1,2})?$"

rule1::port:
  label: "Rule 1: Port"
  type: integer
  min: 1
  max: 65535

rule1::action:
  label: "Rule 1: Action"
  options: ["ALLOW", "DENY"]

# Rule 2
rule2::name:
  label: "Rule 2: Name"

rule2::source:
  label: "Rule 2: Source IP/CIDR"
  regex: "^\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}(/\\d{1,2})?$"

rule2::port:
  label: "Rule 2: Port"
  type: integer
  min: 1
  max: 65535

rule2::action:
  label: "Rule 2: Action"
  options: ["ALLOW", "DENY"]

# Rule 3
rule3::name:
  label: "Rule 3: Name"

rule3::source:
  label: "Rule 3: Source IP/CIDR"
  regex: "^\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}(/\\d{1,2})?$"

rule3::port:
  label: "Rule 3: Port"
  type: integer
  min: 1
  max: 65535

rule3::action:
  label: "Rule 3: Action"
  options: ["ALLOW", "DENY"]
```

**Output Configuration:**
```json
{
  "rule1::name": "Allow HTTPS",
  "rule1::source": "10.0.0.0/8",
  "rule1::port": 443,
  "rule1::action": "ALLOW",
  "rule2::name": "Block SSH from Internet",
  "rule2::source": "0.0.0.0/0",
  "rule2::port": 22,
  "rule2::action": "DENY",
  "rule3::name": "Allow HTTP from VPN",
  "rule3::source": "192.168.0.0/16",
  "rule3::port": 80,
  "rule3::action": "ALLOW"
}
```

**Pattern 2: Named Groups (Semantic)**

Use when groups have **distinct purposes**:

**Template Example - Database Connections:**
```yaml
primary_db::host:
  label: "Primary Database: Host"

primary_db::port:
  label: "Primary Database: Port"
  type: integer
  value: 5432

primary_db::name:
  label: "Primary Database: Database Name"

secondary_db::host:
  label: "Secondary Database: Host"

secondary_db::port:
  label: "Secondary Database: Port"
  type: integer
  value: 5432

secondary_db::name:
  label: "Secondary Database: Database Name"

failover_db::host:
  label: "Failover Database: Host"

failover_db::port:
  label: "Failover Database: Port"
  type: integer
  value: 5432

failover_db::name:
  label: "Failover Database: Database Name"
```

**Output Configuration:**
```json
{
  "primary_db::host": "db-primary.example.com",
  "primary_db::port": 5432,
  "primary_db::name": "production",
  "secondary_db::host": "db-secondary.example.com",
  "secondary_db::port": 5432,
  "secondary_db::name": "production",
  "failover_db::host": "db-failover.example.com",
  "failover_db::port": 5432,
  "failover_db::name": "production"
}
```

**Pattern 3: Prefix Convention**

Use for **hierarchical-like organization** while staying flat:

```yaml
app::service_name:
  label: "Application: Service Name"

app::service_port:
  label: "Application: Service Port"
  type: integer

app::timeout_ms:
  label: "Application: Timeout (ms)"
  type: integer

logging::level:
  label: "Logging: Level"
  options: ["DEBUG", "INFO", "WARN", "ERROR"]

logging::max_size_mb:
  label: "Logging: Max File Size (MB)"
  type: integer

monitoring::enabled:
  label: "Monitoring: Enabled"
  type: boolean

monitoring::endpoint:
  label: "Monitoring: Endpoint URL"
```

#### 6.2.3 Consuming Application Pattern

**Python Example - Parsing Numbered Groups:**

```python
import re
import json

# Load configuration from DCCM
with open('/mnt/dccm/public/prod/firewall-rules.json') as f:
    config = json.load(f)

# Parse grouped parameters using double colon delimiter
def parse_grouped_fields(config, pattern):
    """Parse fields matching pattern into grouped structure."""
    groups = {}

    for key, value in config.items():
        match = re.match(pattern, key)
        if match:
            group_id = match.group(1)  # e.g., "1", "2", "3"
            field_name = match.group(2)  # e.g., "name", "source", "port", "action"

            if group_id not in groups:
                groups[group_id] = {}
            groups[group_id][field_name] = value

    return groups

# Parse firewall rules using :: delimiter
rules = parse_grouped_fields(config, pattern=r'rule(\d+)::(\w+)')

# Result:
# rules = {
#   '1': {'name': 'Allow HTTPS', 'source': '10.0.0.0/8', 'port': 443, 'action': 'ALLOW'},
#   '2': {'name': 'Block SSH from Internet', 'source': '0.0.0.0/0', 'port': 22, 'action': 'DENY'},
#   '3': {'name': 'Allow HTTP from VPN', 'source': '192.168.0.0/16', 'port': 80, 'action': 'ALLOW'}
# }

# Use the grouped rules
for rule_id, rule in sorted(rules.items()):
    print(f"Applying firewall rule {rule_id}: {rule['action']} {rule['name']}")
    print(f"  Source: {rule['source']}, Port: {rule['port']}")
    # Apply rule to firewall...
```

**Bash Example - Parsing Named Groups:**

```bash
#!/bin/bash

# Load config from DCCM
CONFIG_FILE="/mnt/dccm/public/prod/database-connections.json"

# Extract primary database settings (using :: delimiter in field names)
PRIMARY_HOST=$(jq -r '.["primary_db::host"]' "$CONFIG_FILE")
PRIMARY_PORT=$(jq -r '.["primary_db::port"]' "$CONFIG_FILE")
PRIMARY_NAME=$(jq -r '.["primary_db::name"]' "$CONFIG_FILE")

# Extract secondary database settings
SECONDARY_HOST=$(jq -r '.["secondary_db::host"]' "$CONFIG_FILE")
SECONDARY_PORT=$(jq -r '.["secondary_db::port"]' "$CONFIG_FILE")
SECONDARY_NAME=$(jq -r '.["secondary_db::name"]' "$CONFIG_FILE")

# Connect to primary, fallback to secondary
if ! psql -h "$PRIMARY_HOST" -p "$PRIMARY_PORT" -d "$PRIMARY_NAME" -c "SELECT 1" > /dev/null 2>&1; then
    echo "Primary database down, using secondary"
    psql -h "$SECONDARY_HOST" -p "$SECONDARY_PORT" -d "$SECONDARY_NAME"
else
    psql -h "$PRIMARY_HOST" -p "$PRIMARY_PORT" -d "$PRIMARY_NAME"
fi
```

#### 6.2.4 When NOT to Use DCCM

Consider alternative tools when:

| Scenario | Why DCCM is NOT a Good Fit | Recommended Alternative |
|:---|:---|:---|
| **50+ dynamic list items** | Too many template fields to manage | Database with UI, or API-driven config |
| **Deeply nested hierarchies** | Flat structure becomes unwieldy | Native YAML/JSON config files |
| **Complex data structures** | Requires arrays of objects, nested maps | Terraform state, Ansible inventory, Consul KV |
| **User-driven dynamic lists** | Users need to add/remove items themselves | Application with database backend |
| **Schema changes frequently** | Updating template for each schema change is tedious | Schema-less storage (MongoDB, JSON files) |

**Examples of Poor Fits:**
- ❌ User management (100s of users with multiple attributes each)
- ❌ Product catalog (dynamic inventory)
- ❌ Monitoring targets (dozens of servers with changing IPs)
- ❌ CI/CD pipeline definitions (complex nested stages)

**Examples of Good Fits:**
- ✅ Application configuration (5-10 service parameters)
- ✅ Infrastructure settings (3-5 database connections)
- ✅ Feature flags (10-20 boolean toggles)
- ✅ Integration endpoints (2-3 API configurations)
- ✅ Environment-specific settings (5-10 parameters per env)

#### 6.2.5 Benefits of This Approach

**Maintains DCCM Simplicity:**
- No dynamic form rows (add/remove buttons)
- No nested validation complexity
- No array/object rendering logic
- Simple, predictable UI

**Clear Responsibility Separation:**
- **DCCM**: Generates validated flat configuration
- **Application**: Parses and groups fields based on naming convention
- Each component does what it's best at

**Works for Majority of Use Cases:**
- 80% of configuration needs have 5-15 grouped parameters
- Remaining 20% should use specialized tools anyway

**Prevents Scope Creep:**
- Clear boundary: config vs data management
- Keeps implementation focused and maintainable
- Avoids feature bloat

---

## 7. Technology Stack

| Component | Technology | Rationale |
| :--- | :--- | :--- |
| **Management UI/Logic** | **NiceGUI / Python** | Single-language stack for rapid, dynamic UI generation and complex logic handling. |
| **Validation Engine** | **Pydantic** | Dynamic model generation for robust, one-pass schema validation. Eliminates hand-coded validators. |
| **Retrieval Server** | **Company Standard Web Server** | Highly efficient, high-performance static file serving using existing company infrastructure (e.g., Apache); simplest solution for external API access. |
| **Identity Source** | `SSO_USERNAME` (Env Var) | Leverages existing identity infrastructure without complex integration. |
| **Persistence** | Three Storage Volumes | Management, Retrieval, and Audit volumes with distinct permissions for security isolation. |
| **Template Format** | **JSON or YAML** | The parser must auto-detect and support both formats for template definition. Templates can be uploaded in either format. |
| **Data Format** | **JSON and YAML** | Both formats supported for configuration output; user selects preferred format when saving. |

---

## 8. Configuration Template Example

The following example demonstrates the Dynamic Form Builder capabilities using a microservice configuration template.

**Note on Template Format:**
- Templates can be uploaded in either **JSON** or **YAML** format
- The system must auto-detect the format based on file extension (`.json`, `.yaml`, `.yml`) or content
- The examples below use YAML for readability, but all templates can equivalently be written in JSON
- Both formats produce identical form rendering and validation behavior

### 8.1 The Configuration Template (Input)

```yaml
# FILE: my-service_prod_template.yaml

# ----------------------------------------------------
# 0. Absolute Minimum - Just Field Key (Free Text)
# ----------------------------------------------------
service_name:
  # No keywords at all - valid!
  # Renders as: text input, label "service_name", no validation, default value ""

# ----------------------------------------------------
# 0b. Free Text with Custom Label
# ----------------------------------------------------
service_description:
  label: "Service Description"
  description: "Brief description of this service configuration"
  value: ""
  # Free text input with better UX

# ----------------------------------------------------
# 0c. External Validation via Script
# ----------------------------------------------------
assigned_user:
  label: "Assigned User"
  description: "SSO username of person responsible for this service"
  validate_script: "phone"  # Keyword that maps to /usr/local/bin/phone
  # When user enters "alice.smith" and leaves field:
  # DCCM executes: /usr/local/bin/phone alice.smith
  # Exit code 0 → validation passes, show green checkmark
  # Exit code 1 → validation fails, show error from stderr
  # Prevents saving until validation passes

# ----------------------------------------------------
# 1. Optional Type Inference & Range/Pattern Validation
# ----------------------------------------------------
service_timeout_ms:
  label: "Service Timeout (milliseconds)"
  description: "Maximum time to wait for service response"
  value: 3000
  type: 'integer'
  min: 1000
  max: 15000

debug_log_level:
  label: "Debug Log Level"
  description: "Logging verbosity level for debugging"
  value: INFO
  regex: '^(INFO|WARN|DEBUG|ERROR)$'

# ----------------------------------------------------
# 2. Predefined Strings (Standard Dropdown)
# ----------------------------------------------------
# Use case: Few choices (< 10) - standard dropdown
region_deployment:
  label: "Deployment Region"
  description: "AWS region where this service will be deployed"
  options:
    - US-EAST-1
    - EU-CENTRAL-1
    - ASIA-SOUTHEAST-2
  # Only 3 options - rendered as standard dropdown

# ----------------------------------------------------
# 2b. Boolean / Checkbox
# ----------------------------------------------------
enable_debug_mode:
  label: "Enable Debug Mode"
  description: "Enable verbose logging for troubleshooting"
  type: boolean
  value: false
  # Rendered as checkbox

# ----------------------------------------------------
# 2c. Multi-line Text Area
# ----------------------------------------------------
custom_config:
  label: "Custom Configuration"
  description: "Additional JSON configuration (optional)"
  type: textarea
  multiline: true
  value: ""
  # Rendered as multi-line text area

# ----------------------------------------------------
# 3. External File Lookup (Searchable Dropdown)
# ----------------------------------------------------
# Use case: Hundreds of instances - searchable dropdown is essential
file_instance:
  label: "Instance Selection"
  description: "Select the target instance from available instances"
  lookup_file: "instances"  # Keyword that maps to admin-managed file
  # Config Admin has registered "instances" → /mnt/dccm/mgmt/lookups/available_instances.txt
  # File contains hundreds of lines - automatically rendered as SEARCHABLE dropdown
  # User can type to filter/search through options

# ----------------------------------------------------
# 4. Multi-Select with Delimited File
# ----------------------------------------------------
support_teams:
  label: "Support Teams"
  description: "Select one or more support teams for this configuration"
  lookup_file: "support_teams"  # Keyword that maps to admin-managed file
  separator: ";"
  column: 0  # Use first column (SU_NAME), column index is 0-based
  multi_select: true
  # Config Admin has registered "support_teams" → /mnt/dccm/mgmt/lookups/support_groups.txt
  # File format example:
  # IT-Support;john.doe
  # DevOps-Team;jane.smith
  # Column 0 (IT-Support, DevOps-Team) shown in dropdown, column 1 (john.doe, jane.smith) ignored
```

### 8.2 Dynamic UI Generation and Validation

When an Administrator uploads the template above, the **NiceGUI Management Tier** renders the following form and enforces the stated rules:

| Template Key | UI Component Rendered | Validation Strategy Applied |
| :--- | :--- | :--- |
| `assigned_user` | **Text Input Field** | **External Script Validation:** Calls `/usr/local/bin/phone alice.smith` on field blur. Exit code 0 = valid, non-zero = invalid with error message. |
| `service_timeout_ms` | **Number Input Field** | **Inference:** Forces integer input.<br>**Explicit:** Input must be between 1,000 and 15,000. |
| `debug_log_level` | **Text Input Field** | **Explicit (Regex):** Input string must be `INFO`, `WARN`, `DEBUG`, or `ERROR`. |
| `region_deployment` | **Standard Dropdown** | **Explicit (Options):** User can only select from the three predefined regions. |
| `file_instance` | **Searchable Dropdown (Single)** | **Explicit (Lookup):** Populated dynamically from local .txt file. |
| `support_teams` | **Multi-Select Dropdown** | **Explicit (Lookup + Multi):** Reads delimited file with `;` separator, displays first column only, allows multiple selections. |

### 8.3 The Final Saved Configuration (Output)

Once the user fills out the generated form and saves it, the DCCM writes a clean, validated configuration file to the Retrieval Tier in the user's selected format (JSON or YAML).

**Example JSON Output:**
```json
{
  "service_description": "Production microservice configuration",
  "assigned_user": "alice.smith",
  "service_timeout_ms": 5000,
  "debug_log_level": "WARN",
  "region_deployment": "EU-CENTRAL-1",
  "file_instance": "instance-prod-01",
  "support_teams": ["IT-Support", "DevOps-Team"],
  "enable_debug_mode": true
}
```

**Example YAML Output:**
```yaml
service_description: Production microservice configuration
assigned_user: alice.smith
service_timeout_ms: 5000
debug_log_level: WARN
region_deployment: EU-CENTRAL-1
file_instance: instance-prod-01
support_teams:
  - IT-Support
  - DevOps-Team
enable_debug_mode: true
```

**Note on Multi-Select Fields:**
- Fields with `multi_select: true` (like `support_teams`) are **always** output as arrays
- Even if only one value is selected, it must be an array: `["IT-Support"]`
- Empty selection results in empty array: `[]`

**Note:** Owner and access control metadata (such as Owner list, delegated Editors) are stored separately from the configuration output and are not included in the files served to consuming applications.

---

## 9. UI Mockups & User Interface Guidelines

### 9.1 Screen 1: Template List (Main Dashboard)

```
┌────────────────────────────────────────────────────────────────────┐
│ DCCM - Configuration Manager              User: alice.smith        │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  Templates                                    [+ Upload Template]  │
│  ────────────────────────────────────────────────────────────────  │
│                                                                    │
│  Search: [_________________]  Filter: [All ▼]  Sort: [Name ▼]      │
│                                                                    │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ my-service                                      Owner        │  │
│  │ Production microservice configuration                        │  │
│  │ Last modified: 2025-11-19 14:30 by alice.smith               │  │
│  │ [Edit Config] [View Template] [Manage Access]                │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ database-config                                 Editor       │  │
│  │ Database connection settings                                 │  │
│  │ Last modified: 2025-11-18 09:15 by bob.jones                 │  │
│  │ [Edit Config] [View Template]                                │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ api-gateway                                     Owner        │  │
│  │ API Gateway routing configuration                            │  │
│  │ Last modified: 2025-11-15 16:45 by alice.smith               │  │
│  │ [Edit Config] [View Template] [Manage Access]                │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

### 9.2 Screen 2: Upload Template

```
┌────────────────────────────────────────────────────────────────────┐
│ Upload New Template                              User: alice.smith │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  Template Name *                                                   │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ my-service                                                   │  │
│  └──────────────────────────────────────────────────────────────┘  │
│  Max 50 characters, web-safe only (a-z, 0-9, -, _)                 │
│                                                                    │
│  Template File *                                                   │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ [Choose File] my-service-template.yaml                       │  │
│  └──────────────────────────────────────────────────────────────┘  │
│  Accepts .yaml, .yml, .json                                        │
│                                                                    │
│  Description (Optional)                                            │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ Production microservice configuration template               │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│     A template named "my-service" already exists.                  │
│     You are the Owner. Uploading will overwrite the existing       │
│     template. Existing configurations will use the new schema.     │
│                                                                    │
│  [Cancel]                              [Validate & Upload]         │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

### 9.3 Screen 3: Generated Configuration Form (Dynamic Form Builder)

```
┌────────────────────────────────────────────────────────────────────┐
│ Edit Configuration: my-service                   User: alice.smith │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  Template: my-service (v2, updated 2025-11-19)                     │
│  Output Format: ● JSON  ○ YAML                                     │
│                                                                    │
│  ────────────────────────────────────────────────────────────────  │
│                                                                    │
│  Service Name                                                      │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ prod-api-service                                             │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  Service Description                                               │
│  Brief description of this service configuration                   │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ Production microservice configuration                        │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  Service Timeout (milliseconds)                                    │
│  Maximum time to wait for service response                         │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ 5000                                        [−]  [+]         │  │
│  └──────────────────────────────────────────────────────────────┘  │
│  Must be between 1000 and 15000                                    │
│                                                                    │
│  Debug Log Level                                                   │
│  Logging verbosity level for debugging                             │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ WARN                                         [▼]             │  │
│  └──────────────────────────────────────────────────────────────┘  │
│  Matches pattern: INFO, WARN, DEBUG, or ERROR                      │
│                                                                    │
│  Deployment Region                                                 │
│  AWS region where this service will be deployed                    │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ EU-CENTRAL-1                                 [▼]             │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  Instance Selection                                                │
│  Select the target instance from available instances               │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ instance-prod-01                             [▼] [🔍]        │ │
│  │ ┌──────────────────────────────────────────────────────────┐ │  │
│  │ │ instance-prod-01                                         │ │  │
│  │ │ instance-prod-02                    ← Searchable!        │ │  │
│  │ │ instance-prod-03                                         │ │  │
│  │ │ ... (100+ instances)                                     │ │  │
│  │ └──────────────────────────────────────────────────────────┘ │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  Support Teams                                                     │
│  Select one or more support teams                                  │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ [×] IT-Support    [×] DevOps-Team    [ ] QA-Team             │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  Enable Debug Mode                                                 │
│  Enable verbose logging for troubleshooting                        │
│  [✓] Enabled                                                       │
│                                                                    │
│  ────────────────────────────────────────────────────────────────  │
│                                                                    │
│  [Cancel]                              [Validate]  [Save Config]   │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

### 9.4 Screen 4: Manage Access (Owners & Editors)

```
┌────────────────────────────────────────────────────────────────────┐
│ Manage Access: my-service                        User: alice.smith │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  Owners (Full Control)                                             │
│  ────────────────────────────────────────────────────────────────  │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ alice.smith (You)                          [Remove] [×]      │  │
│  │ Initial uploader - 2025-11-15 10:00                          │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ charlie.admin                               [Remove]         │  │
│  │ Added by alice.smith - 2025-11-18 14:30                      │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  Add Owner                                                         │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ Enter SSO username...                        [Add Owner]     │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  Authorized Users (Editors - Can Update Values Only)               │
│  ────────────────────────────────────────────────────────────────  │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ bob.jones                                   [Remove]         │  │
│  │ Via Support Unit: DevOps-Team                                │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ diana.dev                                   [Remove]         │  │
│  │ Direct delegation by alice.smith                             │  │
│  └──────────────────────────────────────────────────────────────┘  │ 
│                                                                    │
│  Add Editor                                                        │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ ● By Username  ○ By Support Unit                             │  │
│  │ Enter SSO username...                        [Add Editor]    │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  [Close]                                                [Save]     │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

### 9.5 Screen 5: Validation Errors

```
┌────────────────────────────────────────────────────────────────────┐
│ Template Validation Failed                      User: alice.smith  │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  ❌ Template "my-service-v2.yaml" contains errors:                │
│                                                                    │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ Line 15: Invalid JSON syntax                                 │  │
│  │   Expected ',' or '}' after property value                   │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ Line 23: Unrecognized keyword 'max_length'                   │  │
│  │   Did you mean 'max'?                                        │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  ┌──────────────────────────────────────────────────────────────┐  │
  │  │ Line 34: Invalid lookup_file keyword                         │  │
  │  │   'unknown_list' is not registered. Available: support_teams,│  │
  │  │   instances, regions, environments                           │  │
  │  │   instances, regions, environments                           │  │
  │  │ Line 34: Invalid lookup_file keyword                         │  │
  │  │ Line 45: Lookup file not accessible                          │  │
  │  │   'instances' maps to file that does not exist or cannot be  │  │
  │  │   read. Contact Config Administrator.                        │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ Line 45: Invalid file path (path traversal detected)         │  │
│  │   ../../etc/passwd contains '..' - rejected                  │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  Please fix these errors and upload again.                         │
│                                                                    │
│  [Back to Upload]                                                  │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

### 9.6 Screen 6: Audit Trail (Config Administrators Only)

```
┌────────────────────────────────────────────────────────────────────┐
│ Audit Trail: my-service                     User: charlie.admin    │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  Filter: [Username ▼] [Date Range ▼]                [Export CSV]   │
│                                                                    │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ 2025-11-19 14:30:15                                          │  │
│  │ Configuration Updated by alice.smith                         │  │
│  │ Changed fields: service_timeout_ms (3000 → 5000)             │  │
│  │ [View Full Diff] [Restore This Version]                      │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ 2025-11-18 09:15:42                                          │  │
│  │ Template Updated by alice.smith                              │  │
│  │ Added new field: enable_debug_mode                           │  │
│  │ [View Full Diff] [Restore This Version]                      │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ 2025-11-15 10:00:00                                          │  │
│  │ Template Created by alice.smith                              │  │
│  │ Initial version                                              │  │
│  │ [View Template] [Restore This Version]                       │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  [Close]                                                           │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

### 9.7 UI Design Guidelines

#### General Principles
- **Simplicity**: Clear, uncluttered interface with focus on core tasks
- **Responsive**: Forms adapt to content (more fields = scrollable)
- **Validation Feedback**: Real-time validation with clear error messages
- **Progressive Disclosure**: Show advanced options only when needed

#### Field Rendering Rules
- **Text Input**: Single line, full width by default
- **Number Input**: With +/- controls, display min/max constraints
- **Text Area**: Multi-line, resizable, minimum 4 rows
- **Dropdown (Standard)**: Native select element for < 10 items
- **Dropdown (Searchable)**: Autocomplete/typeahead for ≥ 10 items or lookup_file
- **Multi-Select**: Checkbox tags that can be added/removed
- **Checkbox**: Single checkbox with label inline

#### Color Coding (Suggested)
- **Info/Help Text**: Gray (#666)
- **Success/Valid**: Green (#28a745)
- **Warning**: Orange (#ffc107)
- **Error**: Red (#dc3545)
- **Primary Actions**: Blue (#007bff)

---

## 10. Critical Implementation Considerations

### 10.1 The Three-Layer Assembly Process

The DCCM application uses a **three-file merge strategy** to render the configuration edit screen. Understanding this assembly process is critical for correct implementation.

When a user visits `/edit/my-service`, the system reads and merges three distinct files from disk:

| Input File | Role | UI Analogy | Purpose |
| :--- | :--- | :--- | :--- |
| **Template** (`.yaml`/`.json`) | **The Architect** | Builds the walls, windows, and doors | Defines **structure** and validation rules |
| **Config** (`.json`/`.yaml`) | **The Decorator** | Paints the walls and places the furniture | Provides **data values** for form fields |
| **Metadata** (`.meta.json`) | **The Security Guard** | Decides who gets the keys | Controls **access** and permissions |

#### 10.1.1 Layer 1: Template → Builds the Structure

**File:** `management_fs/my-service.yaml`

**Purpose:** Defines what UI components to create and their validation rules.

**Example Input:**
```yaml
service_timeout:
  type: number
  min: 1000
  max: 15000
  label: "Service Timeout (ms)"
```

**UI Effect:**
- System creates a Number Input field
- Sets validation: must be between 1000-15000
- Displays label "Service Timeout (ms)"

**Without this file:** The screen is blank - no form fields exist.

#### 10.1.2 Layer 2: Config → Fills the Data

**File:** `retrieval_fs/my-service.json`

**Purpose:** Provides the current values to pre-fill into form fields.

**Example Input:**
```json
{
  "service_timeout": 5000
}
```

**UI Effect:**
- The Number Input field (created by Layer 1) is pre-filled with value `5000`

**Without this file:** Form fields are empty (or show default values from template).

#### 10.1.3 Layer 3: Metadata → Controls State & Access

**File:** `management_fs/my-service.meta.json`

**Purpose:** Determines what the user can do with the form.

**Example Input:**
```json
{
  "owners": ["alice"],
  "editors": ["bob"]
}
```

**UI Effect based on SSO_USERNAME:**
- **If user is Alice (Owner):**
  - All inputs are editable
  - "Save" button is visible and enabled
  - "Delete" button is visible
  - "Manage Access" button is visible
- **If user is Bob (Editor):**
  - All inputs are editable
  - "Save" button is visible and enabled
  - "Delete" button is hidden
  - "Manage Access" button is hidden
- **If user is Eve (No access):**
  - Show "Access Denied" error page OR
  - Show read-only view (inputs grayed out, no save button)

**Without this file:** System cannot determine permissions - should show error.

#### 10.1.4 The Assembly Algorithm

When a user opens `/edit/my-service`, the backend executes this sequence:

**Step 1: Load Metadata & Check Permissions**
```python
# Load management_fs/my-service.meta.json
metadata = load_metadata("my-service")

# Check if user has access
user = os.environ["SSO_USERNAME"]
if user not in (metadata["owners"] + metadata["editors"]):
    return "Access Denied"

# Set permission flags
is_owner = user in metadata["owners"]
can_delete = is_owner
can_manage_access = is_owner
```

**Step 2: Load Template & Build Form Structure**
```python
# Load management_fs/my-service.yaml
template = load_template("my-service")

# Create UI elements
form_fields = {}
for field_key, field_schema in template.items():
    ui_element = create_ui_element(field_schema)  # Creates Number, Text, Dropdown, etc.
    form_fields[field_key] = ui_element
```

**Step 3: Load Config & Populate Values**
```python
# Load retrieval_fs/my-service.json (if exists)
if config_exists("my-service"):
    config_data = load_config("my-service")

    # Populate form fields with existing values
    for field_key, value in config_data.items():
        if field_key in form_fields:
            form_fields[field_key].value = value
        else:
            # Orphan data - field exists in config but not in template
            preserved_data[field_key] = value  # Store for later
```

**Step 4: Handle Orphan Data (Strategy B - Preserve)**
- If config file contains fields that are NOT in the current template, preserve them in a hidden variable
- When user saves, merge these orphan fields back into the output
- This prevents data loss when templates are updated

**Step 5: Render UI**
```python
# Render form with appropriate buttons based on permissions
render_form(form_fields)

if can_delete:
    render_delete_button()

if can_manage_access:
    render_manage_access_button()

render_save_button()  # All editors and owners can save
```

#### 10.1.5 Orphan Data Handling (Preserve with Feedback)

**Problem:** A field exists in the configuration file but has been removed from the template.

**Example Scenario:**
1. Template version 1 has: `service_name`, `service_timeout`, `debug_mode`
2. Config file contains: `{"service_name": "api", "service_timeout": 5000, "debug_mode": true}`
3. Owner updates template to version 2, removing `debug_mode`
4. User opens the configuration - what happens to `debug_mode`?

**Solution: Preserve Orphan Data with User Notification**

**Detection:**
- On load (Step 3 of assembly), identify keys in the config file that are missing from the current template
- Count orphan fields: `orphan_count = len(preserved_data)`

**Action:**
- Store orphan data in hidden state: `preserved_data = {"debug_mode": true}`
- When user saves, merge: `final_config = {form_data} + {preserved_data}`
- Result: `debug_mode` is preserved even though it's not visible in the form

**UI Feedback (Required):**

Display a **non-blocking warning banner** at the top of the configuration form:

```
┌────────────────────────────────────────────────────────────────────┐
│ Edit Configuration: my-service                   User: alice.smith │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  ⚠ 3 fields detected in file that are not in the current template │
│     They will be preserved upon save. [View Hidden Fields]         │
│                                                                    │
│  ────────────────────────────────────────────────────────────────  │
│  ... (visible form fields)                                         │
└────────────────────────────────────────────────────────────────────┘
```

**Banner Requirements:**
- Must be visible immediately when form loads (if orphan data exists)
- Non-blocking: User can still edit and save the form
- Shows count of orphan fields
- Styling: Warning color (orange/yellow background)
- **Optional:** "View Hidden Fields" button for Config Administrators to see raw JSON of preserved data

**Rationale:**
- Prevents accidental data loss during template evolution
- **User awareness:** Shifts from silent preservation to informed preservation
- Allows rollback: If owner reverts to template v1, `debug_mode` reappears
- Warning: Orphan data accumulates - recommend periodic cleanup by Config Administrators

### 10.2 Concurrency & Race Conditions

#### 10.2.1 The "Last Write Wins" Problem

**Scenario (Single Environment):**
1. **09:00 AM:** Alice opens `/edit/my-service?env=prod` (config has `timeout: 3000`)
2. **09:00 AM:** Bob opens `/edit/my-service?env=prod` (config has `timeout: 3000`)
3. **09:05 AM:** Alice changes `timeout` to `5000` and saves
4. **09:06 AM:** Bob changes `timeout` to `8000` and saves (he never saw Alice's change)
5. **Result:** Final config has `timeout: 8000` - Alice's change is **silently lost**

**Problem:** Without concurrency control, the last person to click "Save" wins, and intermediate changes are lost without warning.

**Multi-Environment Consideration:** This problem occurs PER ENVIRONMENT. Alice editing prod doesn't conflict with Bob editing dev.

#### 10.2.2 Required Solution: Exclusive Locking

To prevent data loss, the system **must** implement exclusive locking:

**Mechanism: File-Based Exclusive Lock**

When a user opens a configuration for editing:

1. **Acquire Lock (Per-Environment):**
   ```python
   # Lock is PER ENVIRONMENT - not template-wide
   lock_file = f"management_fs/.locks/{template_name}_{environment}.lock"
   # Example: management_fs/.locks/my-service_prod.lock

   # Try to create lock file with current user and timestamp
   if lock_exists(lock_file):
       lock_info = read_lock(lock_file)
       return f"Configuration ({environment}) is locked by {lock_info['user']} since {lock_info['timestamp']}"

   create_lock(lock_file, {"user": SSO_USERNAME, "timestamp": now(), "environment": environment})
   ```

2. **Display Lock Status:**
   - Show banner: "You have exclusive edit access to the PROD environment. Lock acquired at 09:00 AM."
   - Other users see: "The PROD configuration is locked by Alice (editing since 09:00 AM). Try again later."
   - **Important:** Dev and test configs remain editable by other users

3. **Release Lock:**
   - On Save: Release lock automatically
   - On Cancel: Release lock
   - On Session Timeout: Auto-release after 15 minutes of inactivity
   - On Browser Close: Best-effort release (use `beforeunload` event)

4. **Lock Timeout & Override:**
   - Config Administrators can forcefully break locks if user's session crashed
   - Log lock breaks in audit trail (include environment in log entry)

**Implementation Notes:**

**Lock File Naming Convention (Per-Environment):**
- Store in Management Volume: `management_fs/.locks/{template-name}_{environment}.lock`
- Do NOT store in Retrieval Volume (locks are internal metadata)
- Examples:
  - `/mnt/dccm/mgmt/.locks/my-service_dev.lock`
  - `/mnt/dccm/mgmt/.locks/my-service_test.lock`
  - `/mnt/dccm/mgmt/.locks/my-service_prod.lock`

**Lock File Content:**
```json
{
  "locked_by": "alice.smith",
  "locked_at": "2025-11-19T09:00:15Z",
  "session_id": "abc123",
  "environment": "prod"
}
```

**Key Behavior:**
- Alice editing `my-service` (prod) does NOT block Bob from editing `my-service` (dev)
- Lock is environment-specific, not template-specific
- Each environment can be edited independently and concurrently by different users

**Lock Cleanup:**
- Implement background job to release locks older than 15 minutes
- When user logs in, check for stale locks by this user and auto-release

**Alternative: Optimistic Locking (NOT RECOMMENDED for this spec)**

The user requested **exclusive locking** to ensure only one person can modify at a time. Optimistic locking (version checking on save) would allow conflicts, which violates the requirement.

#### 10.2.3 UI/UX for Locked Configurations

**Read-Only View for Locked Configs:**

When a user tries to open a locked configuration:

```
┌────────────────────────────────────────────────────────────────────┐
│ Configuration: my-service (READ-ONLY)          User: bob.jones     │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  ⚠ This configuration is currently locked for editing              │
│     Locked by: alice.smith                                         │
│     Since: 2025-11-19 09:00:15                                     │
│                                                                    │
│  You can view the current values below, but cannot make changes.   │
│                                                                    │
│  [Refresh to Check Lock Status]                                    │
│                                                                    │
│  ────────────────────────────────────────────────────────────────  │
│                                                                    │
│  Service Timeout (ms)                                              │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ 5000                                              [disabled] │  │
│  └──────────────────────────────────────────────────────────────┘  │
│  ... (all fields shown as disabled/read-only)                      │
│                                                                    │
│  [Close]                                                           │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

**Active Edit View (Lock Holder):**

```
┌────────────────────────────────────────────────────────────────────┐
│ Edit Configuration: my-service                  User: alice.smith  │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  🔒 You have exclusive edit access                                 │
│     Lock acquired: 2025-11-19 09:00:15                             │
│     Auto-release in: 14 minutes (on save, cancel, or timeout)      │
│                                                                    │
│  ────────────────────────────────────────────────────────────────  │
│  ... (editable form fields)                                        │
│                                                                    │
│  [Cancel & Release Lock]                          [Save]           │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

### 10.3 Additional Implementation Considerations

#### 10.3.1 File System Failures

**Scenario:** The Management Volume becomes unavailable (NFS mount fails, disk full, permissions issue).

**Requirements:**
- Display clear error message: "System error: Unable to access template storage. Contact administrator."
- Log detailed error to system logs (not visible to user)
- Do NOT expose file system paths or internal structure to users
- Gracefully degrade: If templates can't be loaded, show list of configs that are already cached (if applicable)

#### 10.3.2 External Dependency Failures (lookup_file)

**Scenario:** A template references `lookup_file: "instances"` but the mapped file is missing or unreadable.

**Requirements:**
- Detect during form rendering (Step 2 of assembly)
- Show field-level error: "⚠ Data source unavailable: instance list cannot be loaded (contact Config Administrator)"
- Disable the affected field (render as disabled dropdown with error message)
- Do NOT allow saving - missing lookup data indicates system configuration issue
- Admin notification: Log to audit trail that lookup keyword's file is missing
- Error message should include lookup keyword for admin troubleshooting: "Lookup 'instances' mapped to '/mnt/dccm/mgmt/lookups/available_instances.txt' is not accessible"

#### 10.3.3 Template-Config Schema Mismatch

**Scenario:** Config file contains `service_timeout: "fast"` (string) but template defines `type: number`.

**Requirements:**
- Detect during value population (Step 3 of assembly)
- Attempt type coercion: Try to convert "5000" string → 5000 number
- If coercion fails, show validation error: "⚠ Invalid value for service_timeout: expected number, got string"
- Preserve original value in orphan data (don't lose it)
- Prevent save until user corrects the value

---

## 11. Implementation Effort & Timeline

### 11.1 Effort Estimation Matrix

**IMPORTANT:** This estimation reflects the **FULL specification** including all features added during refinement (multi-environment, drift detection, validation scripts, registries).

#### Core Components (Original Scope)

| Component | Complexity | Estimated Days | Notes |
| :--- | :--- | :--- | :--- |
| **Project Skeleton & Infrastructure** | Low | 2 | Setup Repo, CI/CD mock, Env vars, Logging. |
| **Dynamic Form Builder (The Core)** | High | 7 | Parsing YAML to NiceGUI elements. Binding state. |
| **Validation Logic (Basic)** | Medium | 3 | Regex, Min/Max, and Type checking. |
| **File I/O & Basic Lookup Parsing** | Low | 2 | Reading/Writing JSON/YAML, parsing `.txt` lists. |
| **Access Control (RBAC)** | Medium | 3 | Handling `meta.json` and SSO logic. |
| **UI Polish & UX (Basic)** | Medium | 3 | Making it look like the mockups, error toasts. |
| **Subtotal - Basic DCCM** | | **20 Days** | |

#### Extended Features (Added During Specification)

| Component | Complexity | Estimated Days | Notes |
| :--- | :--- | :--- | :--- |
| **Multi-Environment Architecture** | High | 7 | Environment registry, per-environment storage/locking/metadata, copy/promote UI, environment selection mandatory workflow. |
| **Lookup File Registry System** | Medium | 2 | Keyword-based system, registry management UI, validation, error handling. |
| **Validation Script Integration** | Medium | 3 | Script registry, execution with timeout, exit code handling, error display, security considerations. |
| **Configuration Drift Detection** | High | 8 | Background validation service, 3-day threshold tracking, consecutive failure logic, notification emails (4 iterations), admin impact analysis, validation history storage, escalation workflow, false positive prevention. |
| **Enhanced Per-Environment Audit** | Medium | 2 | Environment field in audit trail, per-environment history tracking. |
| **Multi-Environment UI Components** | Medium | 3 | Environment selector, copy/promote modal, multi-env status dashboard, per-env locking indicators. |
| **Subtotal - Extended Features** | | **25 Days** | |

#### Testing & Buffer

| Component | Complexity | Estimated Days | Notes |
| :--- | :--- | :--- | :--- |
| **Integration Testing** | N/A | 5 | End-to-end workflows, multi-environment scenarios, drift detection edge cases. |
| **Bug Fixes & Edge Cases** | N/A | 5 | File locking race conditions, validation script timeouts, concurrent environment edits. |
| **Subtotal - Testing** | | **10 Days** | |

#### Total Effort

| Scope | Total Days | Working Weeks | Calendar Weeks |
| :--- | :--- | :--- | :--- |
| **MVP (Core Only)** | 20 + 5 buffer = **25 Days** | 5 weeks | 5-6 weeks |
| **Full Specification (All Features)** | 20 + 25 + 10 = **55 Days** | 11 weeks | 12-14 weeks |

**Realistic Timeline for Full Implementation:** **12-14 weeks (3-3.5 months)**

### 11.2 Phased Implementation Approach

Given the full specification scope (55 days), a phased approach is recommended:

#### Phase 1: MVP Core (5-6 Weeks)

**Goal:** Basic DCCM functionality without extended features.

**Includes:**
- Template upload and validation
- Dynamic form generation
- Basic validation (regex, min/max, type checking)
- Basic lookup files (inline file paths, not registry)
- RBAC with owners/editors
- Single configuration per template (no environments)
- Save/edit configurations
- Basic audit trail

**Excludes:**
- Multi-environment architecture
- Drift detection system
- Validation scripts
- Lookup/script registries

**Timeline:** 25 days (5 weeks) + 1 week buffer = **6 weeks**

---

#### Phase 2: Multi-Environment Support (3-4 Weeks)

**Goal:** Enable dev/test/prod environment separation.

**Includes:**
- Environment registry
- Per-environment configuration storage
- Mandatory environment selection
- Copy/promote between environments
- Per-environment locking and metadata
- Per-environment audit trail
- Multi-environment UI components

**Timeline:** 7 days (multi-env) + 3 days (UI) + 2 days (audit) + 3 days buffer = **3 weeks**

---

#### Phase 3: Validation & Governance (3-4 Weeks)

**Goal:** Add drift detection and external validation.

**Includes:**
- Lookup file registry system
- Validation script integration
- Configuration drift detection (background service)
- Notification emails
- Admin impact analysis
- Validation history tracking
- Escalation workflows

**Timeline:** 2 days (lookup registry) + 3 days (validation scripts) + 8 days (drift detection) + 2 days buffer = **3 weeks**

---

#### Recommended Development Sequence

**Option A: MVP First (Fastest Time-to-Value)**
1. Phase 1 (6 weeks) → Deploy to production
2. Phase 2 (3 weeks) → Add multi-environment
3. Phase 3 (3 weeks) → Add drift detection

**Total:** 12 weeks with 3 production releases

**Option B: Full Feature Set (Single Release)**
1. All phases sequentially
2. Comprehensive testing
3. Single production release

**Total:** 12-14 weeks with 1 production release

---

#### Phase 1 Detailed: 6-Week MVP Plan

##### Week 1-2: The "Happy Path" Engine

* **Goal:** Upload a template, generate a form, save a config. (No auth, no validation yet).
* **Tasks:**
  1. Create the `TemplateParser` class: Reads YAML, returns a Python dictionary.
  2. Build the `FormGenerator`: A loop that iterates through the dictionary and spits out `ui.input` or `ui.select` elements.
  3. Implement the "Save" button: Dumps the form state to a JSON file in the `Retrieval` folder.
  4. Implement template upload UI and file storage.
* **Outcome:** A working prototype where you can upload templates, generate forms, and save data.

##### Week 3: Validation & Intelligence

* **Goal:** Stop users from entering bad data.
* **Tasks:**
  1. Implement `Validator`: Connect `min`, `max`, and `regex` from the template to the NiceGUI validation props.
  2. Implement `LookupHandler`: Write the function to read external `.txt` files and feed them into dropdowns.
  3. Add the logic for `multi_select` and `checkbox` types.
  4. Implement Pydantic dynamic model generation for validation.
* **Outcome:** The system is now "safe" to use but has no security.

##### Week 4: Governance & Security

* **Goal:** Lock it down.
* **Tasks:**
  1. Implement `AuthMiddleware`: Read `os.environ["SSO_USERNAME"]`.
  2. Implement `MetaStore`: Create the logic to write/read `.meta.json` files alongside templates.
  3. Apply the "Permission Matrix": Disable the "Save" button if the user isn't in the meta file.
  4. Implement the Audit Log writer (append-only JSON lines).
  5. Implement file-based exclusive locking.
* **Outcome:** The system has security and access control.

##### Week 5: Polish & Core Features

* **Goal:** Make it usable and robust.
* **Tasks:**
  1. **The "Edit" Mode:** Ensure that when loading an existing file, the form populates correctly.
  2. **Error Handling:** What if the `lookup_file` is missing? What if the YAML is broken? Add nice UI notifications (`ui.notify`).
  3. **UI Cleanup:** Add the headers, the dashboard list of templates, and search filtering.
  4. **Manage Access UI:** Owner/editor management screen.
* **Outcome:** Feature-complete MVP.

##### Week 6: Testing & Hardening

* **Goal:** Production readiness.
* **Tasks:**
  1. Integration testing (end-to-end workflows).
  2. Edge case testing (concurrent edits, missing files, malformed templates).
  3. Performance testing (100+ templates, large lookup files).
  4. Security review (path traversal, injection risks).
  5. Documentation (deployment guide, user manual).
* **Outcome:** Production-ready MVP (Phase 1 complete).

### 11.3 Why Phase 1 MVP Fits in 6 Weeks (The Accelerators)

1. **NiceGUI Data Binding:** In a traditional stack (React + FastAPI), syncing the form state is a huge task. In NiceGUI, it looks like this:

   ```python
   # This is why it's fast. Direct binding to a dict.
   config_data = {"timeout": 3000}
   ui.number(label="Timeout", value=3000).bind_value(config_data, 'timeout')
   ```

   This saves days of wiring up API endpoints.

2. **No Database:** You don't need to design tables, write SQL, or handle migrations. You are just dumping `dict` to `json`.

3. **No User Management:** You aren't building a "Forgot Password" flow or a "Registration" page. You rely entirely on the environment variable.

4. **Pydantic Dynamic Validation:** Generate validation models on-the-fly from templates instead of hand-coding validators.

   **The Problem (Traditional Approach):**
   - For each template, manually write validation code
   - Check each field type: `if field == 'timeout': validate_integer(value, min=1000, max=15000)`
   - Hundreds of lines of repetitive validation logic
   - Every template change requires code changes

   **The Pydantic Solution:**
   ```python
   from pydantic import BaseModel, Field, create_model

   # Read template once
   template = load_template("my-service")

   # Build Pydantic model dynamically
   field_definitions = {}
   for field_key, field_schema in template.items():
       field_type = infer_type(field_schema)  # int, str, bool
       constraints = {}

       if 'min' in field_schema:
           constraints['ge'] = field_schema['min']
       if 'max' in field_schema:
           constraints['le'] = field_schema['max']
       if 'regex' in field_schema:
           constraints['pattern'] = field_schema['regex']

       field_definitions[field_key] = (field_type, Field(**constraints))

   # Create model class at runtime
   DynamicConfigModel = create_model('DynamicConfigModel', **field_definitions)

   # Validate entire form payload in one line
   validated_data = DynamicConfigModel.model_validate(form_data)
   ```

   **Why This Accelerates Development:**
   - **Zero validation code per template** - validation is generated, not written
   - **Automatic error messages** - Pydantic produces clear validation errors
   - **Type safety** - Catches type mismatches before saving
   - **One-pass validation** - All fields validated simultaneously (no cascading checks)
   - **Eliminates bugs** - No risk of forgetting to validate a field

   **Time Saved:**
   - Manual validation: ~2 hours per template × 10 templates = 20 hours
   - Pydantic approach: ~4 hours to build the generator once = **16 hours saved**

   This is why dynamic model generation is critical for hitting the 4-week timeline.

### 11.4 Risk Factors (Where the Timeline Will Slip)

#### Phase 1 MVP Risks (Could extend 6 weeks to 8-10)

1. **"Perfect" Diffing:** Building a UI that shows a visual *diff* (Red/Green lines) between versions is complex. **MVP Mitigation:** Just show the "Before" and "After" JSON text side-by-side.
2. **Complex File Locking:** Trying to build a perfect database-grade locking mechanism on a file system. **MVP Mitigation:** Use Python's `fcntl` (Linux) or `msvcrt` (Windows) for simple file locks.
3. **Frontend Perfectionism:** Trying to make NiceGUI look exactly like a custom React app. **MVP Mitigation:** Accept the standard Material Design look that NiceGUI provides out of the box.
4. **Over-Engineering Template Validation:** Building a full JSON Schema validator. **MVP Mitigation:** Simple field-by-field validation checking for required keywords.
5. **Premature Optimization:** Worrying about performance before there's a problem. **MVP Mitigation:** Simple file reads/writes. Optimize only if real bottlenecks appear.

#### Phase 2 Multi-Environment Risks (Could extend 3 weeks to 5)

1. **Complex Copy Logic:** Trying to implement "merge" or "diff-based copy" between environments. **Mitigation:** Simple full-copy only. User manually handles conflicts.
2. **Environment Permissions:** Per-environment access control (dev owners vs prod owners). **Mitigation:** Template-level permissions apply to ALL environments.
3. **Cross-Environment Validation:** Validating consistency across environments. **Mitigation:** Skip for MVP. Each environment is independent.

#### Phase 3 Drift Detection Risks (Could extend 3 weeks to 5)

1. **Perfect Email System:** Building retry logic, templates, HTML emails. **Mitigation:** Simple text emails via `smtplib`. No retries.
2. **Complex Notification Rules:** Different rules for different fields, escalation chains. **Mitigation:** Fixed 3-day threshold, fixed 4-week escalation. No customization.
3. **Real-Time Drift Detection:** Checking on every page load. **Mitigation:** Daily background job only. No real-time checks.
4. **Validation Script Dependency Management:** Managing script versions, dependencies. **Mitigation:** Scripts are admin-managed binaries. No version tracking.

#### Overall Project Risks (Could extend 12 weeks to 16-18)

1. **Scope Creep:** Adding features not in specification during development.
   - **Mitigation:** Strict adherence to specification. New features go in Phase 4.

2. **Testing Underestimation:** Complex edge cases in multi-environment + drift detection.
   - **Mitigation:** Add 20% buffer to each phase for testing.

3. **Integration Complexity:** Background validation service + email system + external scripts.
   - **Mitigation:** Keep components loosely coupled. Use simple file-based communication.

4. **Production Issues:** Unforeseen problems in production (file permissions, network issues).
   - **Mitigation:** 2-week post-deployment stabilization period budgeted separately.

#### Conservative Timeline Estimates

| Scenario | Timeline | Notes |
| :--- | :--- | :--- |
| **Best Case** | 12 weeks | Experienced dev, no major blockers, tight scope control |
| **Realistic** | 14 weeks | Normal development pace with testing |
| **Conservative** | 16-18 weeks | Includes scope creep, production issues, rework |
| **Worst Case** | 20+ weeks | Multiple developers, unclear requirements, frequent changes |

**Recommendation for Practice Exam:** Target **Phase 1 MVP (6 weeks)** as the exam scope. Full specification is too large for a practice exam unless it's a multi-month project.

---
