# 🚀 PROJECT SPECIFICATION: Dynamic Central Configuration Manager (DCCM)

**Date:** November 19, 2025
**Technology Base:** Python / NiceGUI / Static Web Server
**Architecture Pattern:** Two-Tier Management & Retrieval

## 1. Executive Summary

The **Dynamic Central Configuration Manager (DCCM)** is a simplified, unified configuration platform designed to eliminate manual configuration errors and enhance governance. Its core value lies in its **Template-First** approach: uploading a JSON or YAML schema automatically generates a user-friendly form.

The system utilizes a two-tier architecture for optimal performance:
1.  **Management Tier (NiceGUI):** Handles complex validation and write operations.
2.  **Retrieval Tier (Company Web Server):** Handles simple, high-speed read operations via static file serving using the company's standard web server solution (e.g., Apache).

---

## 2. Architecture Overview: Two-Tier Model

The system is split into two functionally distinct, loosely coupled components that interact via a **Shared Filesystem**.

### 2.1 Component Separation

* **Management Tier (Write):** A single Python application running **NiceGUI**. This tier is accessed by administrators and editors and hosts the complex logic (RBAC, Validation, Form Generation).
* **Retrieval Tier (Read):** A standard, high-performance web server provided by the company (e.g., Apache). This tier provides applications with high-speed, **read-only** access to the configuration files via standard HTTP GET requests.

### 2.2 Filesystem Separation (Critical Security Requirement)

The system requires **three separate filesystems** with different access controls:

| Filesystem | Purpose | Access | Contents |
| :--- | :--- | :--- | :--- |
| **Management Filesystem** | Template & Metadata Storage | Management Tier only | Templates (`.yaml`/`.json`), Metadata (`.meta.json`) |
| **Retrieval Filesystem** | Configuration Output | Management Tier (write), Retrieval Tier (read), Consuming Apps (read via HTTP) | Final configuration files only (`.json`/`.yaml`) |
| **Audit Filesystem** | Audit Trail & History | Config Administrators only | Complete change history, version snapshots |

**Critical Constraints:**
* Templates and metadata **must never** be accessible via the Retrieval Tier
* The Retrieval Filesystem must **only** contain final configuration output files
* Audit trail must be on a completely separate filesystem with admin-only access
* No cross-contamination between filesystems

### 2.3 Simplified Internal Architecture (Management Flow)

The Management Tier ensures data integrity through a sequential, modular process upon form submission:

1.  **Identity & Authorization Module:** Retrieves the user's identity from the **`SSO_USERNAME`** environment variable and checks permissions against the saved Owner/Delegated Access List.
2.  **Data Integrity Validation Module:** Performs type checking, schema validation (`min`/`max`/`regex`), and resolves external **`lookup_file`** data.
3.  **Persistence & Output Module:** Runs the **Collision Detection** check and writes the final, validated configuration file to the **Retrieval Filesystem**.

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
* **Metadata Content**: For each template/configuration, store:
  * List of Owner usernames (SSO_USERNAME values)
  * List of Authorized User (Editor) usernames or Support Unit references
  * Original uploader and upload timestamp
  * Last modified by and timestamp
* **Metadata Storage Implementation**: Use separate `.meta.json` files alongside each template
  * For template `my-service`, store metadata in `my-service.meta.json`
  * Store in the **Management Filesystem** alongside templates
  * **Critical**: Never store in Retrieval Filesystem - metadata must not be accessible via HTTP
* **Critical Constraint**: Metadata must **never** appear in configuration files served to consuming applications via the Retrieval Tier

### 3.4 Audit Trail & History

The system must maintain a complete audit trail for compliance and troubleshooting purposes:

* **Audit Storage Location**:
  * Must be stored on the **Audit Filesystem** (see section 2.2)
  * Completely separate from both Management Filesystem and Retrieval Filesystem
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
* **Restore Capability**:
  * Config Administrators must be able to restore any previous version of a template
  * Config Administrators must be able to restore any previous version of a configuration
  * Restore operation itself must be logged in the audit trail
  * Restore UI must show diff between current and selected version before confirming
* **Access Control for Audit Trail**:
  * Regular users (Owners, Editors): No access to audit trail
  * Config Administrators: Read access to audit trail, restore capability
  * Audit trail queries should support filtering by: template name, username, date range

---

## 4. Template Upload & Validation

* **Template Upload:** Users upload a JSON or YAML template file that defines the configuration schema and form structure.
* **Template Validation:** Before accepting the template, the system must perform comprehensive validation:
  * Verify valid JSON or YAML syntax
  * Check that all schema keywords are recognized (`min`, `max`, `regex`, `options`, `lookup_file`, `separator`, `multi_select`, etc.)
  * Validate that `lookup_file` references point to existing, readable files
  * Validate that `lookup_file` paths are absolute and do not contain path traversal sequences (e.g., `..`)
  * Display **all validation errors** to the user in a clear, actionable format
  * Reject invalid templates and prevent form generation until errors are resolved
* **Error Handling During Template Validation:**
  * **Invalid JSON/YAML syntax**: Display syntax error with line number
  * **Unrecognized schema keyword**: List all unrecognized keywords
  * **Missing `lookup_file`**: Report which file paths do not exist or are not readable
  * **Invalid file path**: Reject paths with traversal sequences or non-absolute paths
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
* The form is dynamically generated based on the current template (read from Management Filesystem)
* Upon save, the validated configuration file is written to the **Retrieval Filesystem** for consumption by applications

### 5.3 Configuration File Naming & Retrieval

* **Configuration Naming:** The configuration filename is **derived from the template name** - users cannot choose a different name.
  * When a template is uploaded with name `my-service`, configurations generated from it will be named `my-service.[json|yaml]`
* **File Format:** The user specifies the desired output format (JSON or YAML). The file extension is automatically added based on the selected format.
* **Retrieval:** Consuming applications fetch the file directly by name from the Static Web Server: `http://[config-host]/[template-name].[json|yaml]`.

### 5.4 User Operations

The UI must support these distinct operations:

1. **Upload Template** (Owners/Admins only): Upload/update template schema
2. **Create Configuration** (Owners/Editors): Fill out form to create initial configuration file
3. **Edit Configuration** (Owners/Editors): Modify existing configuration values
4. **View Template** (Owners/Editors): View the current template schema (read-only for Editors)
5. **Delete Template/Configuration** (Admins only): Remove template and associated configuration

### 5.5 Error Handling During Configuration Creation/Editing

* **`lookup_file` disappeared**: If a `lookup_file` that existed during template validation no longer exists when creating/updating a configuration, display error and prevent saving until file is restored
* **Validation failure**: Display specific field validation errors (e.g., "service_timeout_ms must be between 1000 and 15000")
* **Permission denied**: If user lacks permission to update configuration, display clear error message indicating they need Owner or Editor access
* **No template exists**: If user tries to create/edit configuration but template was deleted, display error and prevent operation

---

## 6. Implementation Logic: The Dynamic Form Builder

The core logic is the recursive parser that allows configuration structure and validation to be defined entirely within the JSON or YAML template.

### 6.1 Form Field Types & Validation Strategy

**Supported Field Types:**

The system supports the following input field types, determined by template configuration:

| Field Type | When Used | UI Component | Example Use Case |
| :--- | :--- | :--- | :--- |
| **Text Input** | Default (no keywords) or `value` is string | Single-line text field | Service name, description |
| **Number Input** | `value` is numeric or `type_check: 'integer'` | Numeric input with validation | Timeout, port number, retry count |
| **Text Area** | `multiline: true` | Multi-line text input | Long descriptions, JSON snippets |
| **Dropdown (Standard)** | `options` provided (< 10 items) | Standard dropdown | Region selection (3-5 choices) |
| **Dropdown (Searchable)** | `options` provided (≥ 10 items) OR `lookup_file` | Searchable/autocomplete dropdown | Instance selection (hundreds of choices) |
| **Multi-Select** | `multi_select: true` with `options` or `lookup_file` | Multi-select dropdown | Multiple teams, tags |
| **Checkbox** | `type: 'boolean'` or `value` is boolean | Checkbox | Enable/disable feature flags |

**Field Type Selection Logic:**
1. **Explicit type**: If `type` keyword is present, use it (e.g., `type: 'boolean'`)
2. **Type inference**: Infer from `value` (string → text, number → number input, boolean → checkbox)
3. **Dropdown mode**: If `options` or `lookup_file` present, render as dropdown
   - If `options` has ≥ 10 items OR `lookup_file` is used → **searchable dropdown**
   - If `options` has < 10 items → **standard dropdown**
4. **Default**: Text input

**Schema Keywords:**

| Schema Key | Functionality / NiceGUI Component | Purpose |
| :--- | :--- | :--- |
| `type` | **Explicit Type Declaration** | Forces specific field type: `'text'`, `'number'`, `'boolean'`, `'textarea'`. Overrides type inference. |
| `label` | **Form Field Label** | Human-readable label displayed in the form. If omitted, the field key name is used as the label. |
| `description` | **Help Text** | Optional description/tooltip text displayed below or next to the field to guide users. |
| `value` | **Default/Initial Value** | Initial value for the field. Used for type inference if `type` not specified. |
| `multiline` | **Multi-line Text** | When `true`, renders a text area instead of single-line text input. |
| `min` / `max` / `regex` | **Range & Pattern Checking** | Enforces strict data format and boundary rules. |
| `options` | **Predefined Dropdown** | Renders dropdown from explicit list. Searchable if ≥ 10 items. |
| `lookup_file` | **Searchable Dropdown** | Reads **external local .txt file** to populate searchable dropdown. Ideal for hundreds of choices. |
| `separator` | **File Column Delimiter** | Used with `lookup_file`. Defines the delimiter character (e.g., `;`, `,`, `\t`). Must be used together with `column`. |
| `column` | **Column Index** | Used with `lookup_file` and `separator`. Specifies which column to use for dropdown values (0-indexed). Default: `0` (first column). |
| `multi_select` | **Enable Multi-Select** | Used with `lookup_file` or `options`. When `true`, allows selecting multiple values. |

---

## 7. Technology Stack

| Component | Technology | Rationale |
| :--- | :--- | :--- |
| **Management UI/Logic** | **NiceGUI / Python** | Single-language stack for rapid, dynamic UI generation and complex logic handling. |
| **Retrieval Server** | **Company Standard Web Server** | Highly efficient, high-performance static file serving using existing company infrastructure (e.g., Apache); simplest solution for external API access. |
| **Identity Source** | `SSO_USERNAME` (Env Var) | Leverages existing identity infrastructure without complex integration. |
| **Persistence** | Shared Filesystem | Simplest method for file persistence and direct consumption by the retrieval tier. |
| **Data Format** | **JSON and YAML** | Both formats supported for template input and configuration output; user selects preferred format. |

---

# Configuration Template Example

The following example demonstrates the Dynamic Form Builder capabilities using a microservice configuration template.

## 1\. The Configuration Template (Input)

```yaml
# FILE: my-service_prod_template.yaml

# ----------------------------------------------------
# 0. Default: Free Text (No Validation)
# ----------------------------------------------------
service_description:
  label: "Service Description"
  description: "Brief description of this service configuration"
  value: ""
  # No validation keywords = free text input
  # Field key: service_description, Form label: "Service Description"

# ----------------------------------------------------
# 1. Optional Type Inference & Range/Pattern Validation
# ----------------------------------------------------
service_timeout_ms:
  label: "Service Timeout (milliseconds)"
  description: "Maximum time to wait for service response"
  value: 3000
  type_check: 'integer'
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
  lookup_file: /etc/config/available_instances.txt
  # File contains hundreds of lines - automatically rendered as SEARCHABLE dropdown
  # User can type to filter/search through options

# ----------------------------------------------------
# 4. Multi-Select with Delimited File
# ----------------------------------------------------
support_teams:
  label: "Support Teams"
  description: "Select one or more support teams for this configuration"
  lookup_file: /etc/config/support_groups.txt
  separator: ";"
  column: 0  # Use first column (SU_NAME), column index is 0-based
  multi_select: true
  # File format example:
  # IT-Support;john.doe
  # DevOps-Team;jane.smith
  # Column 0 (IT-Support, DevOps-Team) shown in dropdown, column 1 (john.doe, jane.smith) ignored
```

## 2\. Dynamic UI Generation and Validation

When an Administrator uploads the template above, the **NiceGUI Management Tier** renders the following form and enforces the stated rules:

| Template Key | UI Component Rendered | Validation Strategy Applied |
| :--- | :--- | :--- |
| `service_timeout_ms` | **Number Input Field** | **Inference:** Forces integer input.<br>**Explicit:** Input must be between 1,000 and 15,000. |
| `debug_log_level` | **Text Input Field** | **Explicit (Regex):** Input string must be `INFO`, `WARN`, `DEBUG`, or `ERROR`. |
| `region_deployment` | **Standard Dropdown** | **Explicit (Options):** User can only select from the three predefined regions. |
| `file_instance` | **Searchable Dropdown (Single)** | **Explicit (Lookup):** Populated dynamically from local .txt file. |
| `support_teams` | **Multi-Select Dropdown** | **Explicit (Lookup + Multi):** Reads delimited file with `;` separator, displays first column only, allows multiple selections. |

## 3\. The Final Saved Configuration (Output)

Once the user fills out the generated form and saves it, the DCCM writes a clean, validated configuration file to the Retrieval Tier in the user's selected format (JSON or YAML).

**Example JSON Output:**
```json
{
  "service_description": "Production microservice configuration",
  "service_timeout_ms": 5000,
  "debug_log_level": "WARN",
  "region_deployment": "EU-CENTRAL-1",
  "file_instance": "instance-prod-01"
}
```

**Example YAML Output:**
```yaml
service_description: Production microservice configuration
service_timeout_ms: 5000
debug_log_level: WARN
region_deployment: EU-CENTRAL-1
file_instance: instance-prod-01
```

**Note:** Owner and access control metadata (such as Owner list, delegated Editors) are stored separately from the configuration output and are not included in the files served to consuming applications.
