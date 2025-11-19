Here is the final, complete project specification and architecture overview for the **Dynamic Central Configuration Manager (DCCM)**, incorporating all requirements and constraints discussed.

This document is tailored for IT Managers, emphasizing governance, simplicity, and efficiency.

---

# 🚀 FINAL PROJECT SPECIFICATION: Dynamic Central Configuration Manager (DCCM)

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

## 3. Functional Scope & Access Control

### 3.1 Owner-Centric Role-Based Access Control (RBAC)

The system uses an efficient, owner-centric model based on the user's **`SSO_USERNAME`**.

| Role | Privilege Level | Granting Mechanism |
| :--- | :--- | :--- |
| **Owner** | **Update/Modify** | Automatically assigned to the initial uploader of the configuration. **Must be saved as metadata.** |
| **Authorized User (Editor)** | **Update/Modify** | Granted via Owner delegation (Support Unit lookup or Manual List). |
| **Config Administrator** | **Full Control (Update + Delete)** | Explicitly assigned role; required for **removal/deletion** of any configuration. |

**Constraint Validation:**
* The system **must block the save operation** if the Owner attempts to delegate access to a **Support Unit** of which they are a member.

### 3.2 Configuration File Naming & Retrieval

* **File Naming:** Files are saved using a **free text name** input provided by the Authorized User.
* **File Format:** The user specifies the desired output format (JSON or YAML). The file extension is automatically added based on the selected format.
* **Collision Handling:** The NiceGUI application runs a check against the Shared Filesystem. If a file with the same name exists, the user is prompted to confirm **overwrite**.
* **Retrieval:** Consuming applications fetch the file directly by name from the Static Web Server: `http://[config-host]/[free-text-name].[json|yaml]`.

---

## 4. Implementation Logic: The Dynamic Form Builder

The core logic is the recursive parser that allows configuration structure and validation to be defined entirely within the JSON or YAML template.

### 4.1 Advanced Validation Strategy

The form generation and validation rely on two strategies:

1.  **Type Inference:** Automatically renders basic input fields (e.g., Number Input for an integer value) based on the template's initial value.
2.  **Explicit Schema Rules:** The parser looks for specific keywords to enable advanced input features and validation:

| Schema Key | Functionality / NiceGUI Component | Purpose |
| :--- | :--- | :--- |
| `min` / `max` / `regex` | **Range & Pattern Checking** | Enforces strict data format and boundary rules. |
| `options` | **Predefined Dropdown** | Renders a standard dropdown from an explicit list of allowed values. |
| `lookup_file` | **Searchable Dropdown** | Reads an **external local test file** line-by-line to populate the input field options, supporting dynamic data lists. |

---

## 5. Technology Stack

| Component | Technology | Rationale |
| :--- | :--- | :--- |
| **Management UI/Logic** | **NiceGUI / Python** | Single-language stack for rapid, dynamic UI generation and complex logic handling. |
| **Retrieval Server** | **Company Standard Web Server** | Highly efficient, high-performance static file serving using existing company infrastructure (e.g., Apache); simplest solution for external API access. |
| **Identity Source** | `SSO_USERNAME` (Env Var) | Leverages existing identity infrastructure without complex integration. |
| **Persistence** | Shared Filesystem | Simplest method for file persistence and direct consumption by the retrieval tier. |
| **Data Format** | **JSON and YAML** | Both formats supported for template input and configuration output; user selects preferred format. |

That's a great request. To demonstrate the power of the **Dynamic Form Builder**, I'll provide an example of a **YAML template** that defines both a standard configuration structure *and* the advanced validation rules discussed.

The example simulates a configuration file for a **Microservice Feature Toggle System**.

-----

# 📝 DCCM Configuration Template Example

This template is written in **YAML** (often preferred over JSON for readability) and defines the schema for a single application's feature flags.

## 1\. The Configuration Template (Input)

```yaml
# FILE: my-service_prod_template.yaml

# ----------------------------------------------------
# 1. Basic Type Inference & Range/Pattern Validation
# ----------------------------------------------------
service_timeout_ms:
  value: 3000
  type_check: 'integer' # Internal flag for type hint
  min: 1000
  max: 15000
  # NOTE: The 'value: 3000' is used for Type Inference (integer)
  # The 'min/max' is used for Explicit Rules Validation

debug_log_level:
  value: INFO
  regex: '^(INFO|WARN|DEBUG|ERROR)$'
  # Ensures the input string matches one of these specific keywords

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
database_instance:
  lookup_file: /etc/config/available_databases.txt
  # The form will read this file (line-by-line) to populate the dropdown.

# ----------------------------------------------------
# 4. Owner & Support Unit Delegation (Metadata)
# ----------------------------------------------------
# These fields are NOT rendered in the user form, but are required 
# by the NiceGUI application logic before saving the final config.
owner:
  value: user_sso_name
support_units:
  lookup_file: /etc/config/support_groups.txt
  # Populates a multi-select dropdown for delegation
```

## 2\. Dynamic UI Generation and Validation

When an Administrator uploads the template above, the **NiceGUI Management Tier** renders the following form and enforces the stated rules:

| Template Key | UI Component Rendered | Validation Strategy Applied |
| :--- | :--- | :--- |
| `service_timeout_ms` | **Number Input Field** | **Inference:** Forces integer input.<br>**Explicit:** Input must be between 1,000 and 15,000. |
| `debug_log_level` | **Text Input Field** | **Explicit (Regex):** Input string must be `INFO`, `WARN`, `DEBUG`, or `ERROR`. |
| `region_deployment` | **Standard Dropdown** | **Explicit (Options):** User can only select from the three predefined regions. |
| `database_instance` | **Searchable Dropdown** | **Explicit (Lookup):** Populated dynamically from the contents of the `/etc/config/available_databases.txt` file. |

## 3\. The Final Saved Configuration (Output)

Once the user fills out the generated form and saves it, the DCCM writes a clean, validated configuration file to the Retrieval Tier in the user's selected format (JSON or YAML).

**Example JSON Output:**
```json
{
  "service_timeout_ms": 5000,
  "debug_log_level": "WARN",
  "region_deployment": "EU-CENTRAL-1",
  "database_instance": "DB-Production-Alpha"
}
```

**Example YAML Output:**
```yaml
service_timeout_ms: 5000
debug_log_level: WARN
region_deployment: EU-CENTRAL-1
database_instance: DB-Production-Alpha
```

*(The owner and support unit information is saved as separate metadata associated with this file, not in the final configuration payload.)
