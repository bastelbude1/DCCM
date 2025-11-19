# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

**IMPORTANT**: This repository contains a **specification document** for the Dynamic Central Configuration Manager (DCCM) project. Our role is as the **customer/requirements owner**, not the implementer.

The specification serves as the baseline for a developer's final practice exam. The developer will decide how to implement the requirements based on the specification provided in `project_specification.md`.

When working in this repository:
- Focus on **refining and clarifying requirements** from a customer perspective
- Do NOT implement code unless explicitly asked to create examples or prototypes
- Maintain the specification document with clear, testable requirements
- Think as an IT Manager defining "what" is needed, not "how" to build it

## Project Overview

**Dynamic Central Configuration Manager (DCCM)** is a two-tier configuration management system built with Python and NiceGUI. The system uses a template-first approach where uploading a JSON/YAML schema automatically generates a user-friendly validation form.

## Architecture

### Two-Tier Model

The system consists of two loosely coupled components with **three separate filesystems**:

**Filesystems:**
1. **Management Filesystem**: Templates + metadata (`.meta.json`) - Management Tier only
2. **Retrieval Filesystem**: Final config output - served via HTTP to consuming apps
3. **Audit Filesystem**: Complete change history - Config Administrators only

**Tiers:**
1. **Management Tier (NiceGUI/Python)**: Reads/writes Management Filesystem, writes to Retrieval Filesystem. Handles validation, RBAC, form generation.

2. **Retrieval Tier (Company Web Server)**: Serves files from Retrieval Filesystem only via HTTP GET at `http://[config-host]/[filename].[json|yaml]`.

### Management Flow (Sequential Processing)

When a configuration is submitted through the generated form:

1. **Identity & Authorization Module**: Retrieves `SSO_USERNAME` from environment and validates against Owner/Delegated Access List
2. **Data Integrity Validation Module**: Performs type checking, schema validation (min/max/regex), resolves `lookup_file` references
3. **Persistence & Output Module**: Runs collision detection and writes validated file to shared filesystem

## Core Functionality

### Dynamic Form Builder

The form generation engine is the heart of DCCM. It uses recursive parsing to convert YAML/JSON templates into validated input forms.

**Default Behavior: Free Text (No Validation)**
- By default, fields with no validation keywords render as free text inputs
- Owners decide what validation (if any) to apply in their templates
- Validation is completely optional

**Type Inference**: Automatically renders appropriate inputs based on initial values (e.g., number input for integers)

**Explicit Schema Rules (Optional)**: Recognized keywords that Owners can use to control form generation and validation:

- `label`: Human-readable label displayed in form (if omitted, field key name is used)
- `description`: Optional help text/tooltip displayed below or next to field
- `min` / `max` / `regex`: Range and pattern enforcement
- `options`: Renders standard dropdown from explicit list
- `lookup_file`: Renders searchable dropdown populated from external local .txt file. Supports one value per line, or multiple columns with delimiter.
- `separator`: Defines delimiter character for delimited files (e.g., `;`, `,`, `\t`). Used with `lookup_file` and `column`.
- `column`: Specifies which column to use for dropdown values (0-indexed). Default: `0` (first column). Used with `lookup_file` and `separator`.
- `multi_select`: When `true`, enables multi-select mode for `lookup_file` or `options` dropdowns.

### Owner-Centric RBAC

Access control is based on `SSO_USERNAME` environment variable:

- **Owner**: Initial uploader automatically assigned as Owner. **Multiple Owners supported** - existing Owners can add additional Owners. Full control over templates and configurations. Stored as metadata.
- **Authorized User (Editor)**: Granted access via Owner delegation (Support Unit lookup or manual list). **Can only update configuration values** through forms, cannot upload or modify templates.
- **Config Administrator**: Explicitly assigned role with full control including deletion of any template or configuration

### Template Upload & Validation

**Critical Distinction**: The system has two separate workflows - template upload does NOT create configuration files.

1. **Template Upload**: Owner uploads schema definition (no config file created)
2. **Configuration Create/Edit**: User fills out form to generate actual config file

**Template Upload Process:**
- Users upload JSON or YAML template files that define the schema
- **Template name becomes the configuration filename** (critical coupling)
- System performs comprehensive validation before accepting:
  - Valid JSON/YAML syntax
  - All schema keywords recognized (`min`, `max`, `regex`, `options`, `lookup_file`)
  - `lookup_file` references point to existing, readable files
  - Display ALL validation errors to user clearly
  - Reject invalid templates until errors resolved
- Template naming follows same constraints as configurations (50 char, web-safe)
- Template collision handling:
  - Any Owner or Config Administrator: Can overwrite existing template
  - Authorized Users (Editors): Cannot upload or overwrite templates at all
  - Others: Must choose different name

### Configuration Naming & Collision Handling

**Configuration Creation Process:**
- After form is generated from template, users fill it out and save validated configuration
- **Configuration filename is derived from template name** - no separate naming step
  - Template `my-service` generates configuration `my-service.[json|yaml]`
- Users select the desired output format (JSON or YAML)
- File extension is automatically added based on selected format
- Configuration update permissions:
  - Owners: Can update configuration values and modify template
  - Authorized Users (Editors): Can update configuration values only
  - Config Administrator: Full control
- Final files are served at predictable URLs: `http://[config-host]/[template-name].[json|yaml]`

## Template Structure Example

```yaml
# Default: Free text (no validation)
service_description:
  label: "Service Description"
  description: "Brief description of this service configuration"
  value: ""

# Type inference with optional explicit validation
service_timeout_ms:
  label: "Service Timeout (milliseconds)"
  description: "Maximum time to wait for service response"
  value: 3000
  type_check: 'integer'
  min: 1000
  max: 15000

# Optional regex validation
debug_log_level:
  label: "Debug Log Level"
  value: INFO
  regex: '^(INFO|WARN|DEBUG|ERROR)$'

# Predefined dropdown
region_deployment:
  label: "Deployment Region"
  options:
    - US-EAST-1
    - EU-CENTRAL-1
    - ASIA-SOUTHEAST-2

# External file lookup (single-select searchable dropdown)
file_instance:
  label: "Instance Selection"
  lookup_file: /etc/config/available_instances.txt

# External file lookup with multi-select and delimiter
support_teams:
  label: "Support Teams"
  description: "Select one or more support teams"
  lookup_file: /etc/config/support_groups.txt
  separator: ";"
  column: 0  # Use first column (0-indexed)
  multi_select: true
  # File: IT-Support;john.doe → shows "IT-Support" in dropdown

```

## Key Implementation Considerations

### Module Separation

Maintain clear separation between the three core modules (Identity/Authorization, Data Validation, Persistence). Each module should be independently testable.

### Form Generator Recursion

The form builder must handle nested structures recursively. Each field in the template (JSON or YAML) should be parsed to determine:
1. What UI component to render (type inference + explicit keywords)
2. What validation rules to apply
3. Whether to include in final output or treat as metadata

The same form can generate output in either JSON or YAML format based on user selection.

### Metadata Storage

Owner and delegation information must be stored separately from the output configuration files. This metadata is required for authorization checks but should never appear in the final JSON/YAML served to consuming applications.

### Audit Trail & History

Complete audit trail required for compliance:
- **Audit Filesystem**: Completely separate from Management and Retrieval filesystems, admin-only access
- **Template history**: Every upload/update logged with full content, user, timestamp
- **Configuration history**: Every value change logged with before/after state
- **Restore capability**: Config Administrators can restore any previous version
- **Access control**: Only Config Administrators can view/restore from audit trail
- Audit trail supports filtering by template name, username, date range

### Critical Filesystem Separation

- **Management Filesystem**: Templates and `.meta.json` files - never exposed via HTTP
- **Retrieval Filesystem**: Only final config output - served to consuming apps
- **Audit Filesystem**: Change history only - admin access only
- No cross-contamination between filesystems

### Lookup File Processing

When encountering `lookup_file` directives:
- Only local .txt files are supported
- Read the specified file line-by-line
- File format options:
  - **Simple**: One value per line (entire line becomes the option)
  - **Delimited**: Multiple columns with `separator` character
- Separator and column support:
  - Use `separator` keyword to define delimiter (e.g., `;`, `,`, `\t`)
  - Use `column` keyword to specify which column to use (0-indexed)
  - Default: `column: 0` (first column) if not specified
  - Parse each line, extract specified column
  - Example: `IT-Support;john.doe` with `separator: ";"` and `column: 0` → shows `IT-Support`
  - Example: `IT-Support;john.doe` with `separator: ";"` and `column: 1` → shows `john.doe`
- Multi-select support:
  - Use `multi_select: true` to enable multiple value selection
  - Works with both `lookup_file` and `options`
  - Output is array of selected values
- Handle file read errors gracefully
- Cache file contents if performance becomes an issue

### Environment-Based Identity

The system relies on `SSO_USERNAME` environment variable being set by upstream authentication (reverse proxy, SSO gateway). The application should fail gracefully if this variable is not present.

## Technology Stack

- **Backend Framework**: Python with NiceGUI for dynamic UI generation
- **Retrieval Server**: Company standard web server solution (e.g., Apache) for static file serving - final definition to be determined by company infrastructure
- **Identity Source**: `SSO_USERNAME` environment variable
- **Data Format**: Both JSON and YAML supported for template input and configuration output; user selects preferred format
- **Persistence**: Shared filesystem between Management and Retrieval tiers
