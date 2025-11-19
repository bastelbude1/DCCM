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

### 2.2 Simplified Internal Architecture (Management Flow)

The Management Tier ensures data integrity through a sequential, modular process upon form submission:

1.  **Identity & Authorization Module:** Retrieves the user's identity from the **`SSO_USERNAME`** environment variable and checks permissions against the saved Owner/Delegated Access List.
2.  **Data Integrity Validation Module:** Performs type checking, schema validation (`min`/`max`/`regex`), and resolves external **`lookup_file`** data.
3.  **Persistence & Output Module:** Runs the **Collision Detection** check and writes the final, validated configuration file to the **Shared Filesystem**.

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
* **Implementation Decision**: The apprentice must choose an appropriate storage mechanism:
  * Option A: Separate `.meta.json` files alongside each template
  * Option B: Centralized metadata database or file
  * Option C: Extended file attributes (xattr) if filesystem supports it
* **Critical Constraint**: Metadata must **never** appear in configuration files served to consuming applications via the Retrieval Tier

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
* **Version History**:
  * The system does **not** maintain template version history
  * Overwriting a template permanently replaces the previous version
  * **Responsibility**: Owners should maintain external backups of templates if version history is needed

---

## 5. Configuration File Naming & Retrieval

* **Configuration Naming:** The configuration filename is **derived from the template name** - users cannot choose a different name.
  * When a template is uploaded with name `my-service`, all configurations generated from it will be named `my-service.[json|yaml]`
* **File Format:** The user specifies the desired output format (JSON or YAML). The file extension is automatically added based on the selected format.
* **Retrieval:** Consuming applications fetch the file directly by name from the Static Web Server: `http://[config-host]/[template-name].[json|yaml]`.
* **Error Handling During Configuration Creation:**
  * **`lookup_file` disappeared**: If a `lookup_file` that existed during template validation no longer exists when creating/updating a configuration, the system must display an error and prevent saving until the file is restored
  * **Validation failure**: Display specific field validation errors (e.g., "service_timeout_ms must be between 1000 and 15000")
  * **Permission denied**: If user lacks permission to update configuration, display clear error message indicating they need Owner or Editor access

---

## 6. Implementation Logic: The Dynamic Form Builder

The core logic is the recursive parser that allows configuration structure and validation to be defined entirely within the JSON or YAML template.

### 6.1 Validation Strategy

**Default Behavior: No Validation (Free Text)**
- By default, all fields render as free text input fields with **no validation**
- The Owner decides what validation (if any) to apply when creating the template

The form generation supports two optional validation strategies:

1.  **Type Inference:** Automatically renders basic input fields (e.g., Number Input for an integer value) based on the template's initial value.
2.  **Explicit Schema Rules (Optional):** The Owner can add specific keywords to enable validation and advanced input features:

| Schema Key | Functionality / NiceGUI Component | Purpose |
| :--- | :--- | :--- |
| `min` / `max` / `regex` | **Range & Pattern Checking** | Enforces strict data format and boundary rules. |
| `options` | **Predefined Dropdown** | Renders a standard dropdown from an explicit list of allowed values. |
| `lookup_file` | **Searchable Dropdown (Single or Multi-Select)** | Reads an **external local .txt file** to populate dropdown options. File format: one value per line, or delimited columns (first column used). |
| `separator` | **File Column Delimiter** | Used with `lookup_file`. Defines the delimiter character (e.g., `;`, `,`, `\t`). First column becomes dropdown values. |
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
  value: ""
  # No validation keywords = free text input

# ----------------------------------------------------
# 1. Optional Type Inference & Range/Pattern Validation
# ----------------------------------------------------
service_timeout_ms:
  value: 3000
  type_check: 'integer' # Internal flag for type hint
  min: 1000
  max: 15000
  # NOTE: The 'value: 3000' is used for Type Inference (integer)
  # The 'min/max' is used for Explicit Rules Validation (OPTIONAL)

debug_log_level:
  value: INFO
  regex: '^(INFO|WARN|DEBUG|ERROR)$'
  # Regex validation is OPTIONAL - Owner decides whether to enforce it

# ----------------------------------------------------
# 2. Predefined Strings (Dropdowns)
# ----------------------------------------------------
region_deployment:
  options:
    - US-EAST-1
    - EU-CENTRAL-1
    - ASIA-SOUTHEAST-2

# ----------------------------------------------------
# 3. External File Lookup (Searchable Dropdown)
# ----------------------------------------------------
file_instance:
  lookup_file: /etc/config/available_instances.txt
  # The form reads this local .txt file to populate the dropdown.
  # Format: One value per line, or multiple columns with separator (first column used)

# ----------------------------------------------------
# 4. Multi-Select with Delimited File
# ----------------------------------------------------
# Example: File contains "SU_NAME;LOGIN_NAME" format
# NOTE: This demonstrates multi-select capability with delimited files
# Use case: Selecting multiple support teams or user groups
support_teams:
  lookup_file: /etc/config/support_groups.txt
  separator: ";"
  multi_select: true
  # File format example:
  # IT-Support;john.doe
  # DevOps-Team;jane.smith
  # Only first column (SU_NAME) is shown in dropdown, multiple selections allowed
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
