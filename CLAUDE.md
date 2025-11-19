# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

**IMPORTANT**: This repository contains a **specification document** for the Dynamic Central Configuration Manager (DCCM) project. Our role is as the **customer/requirements owner**, not the implementer.

The specification serves as the baseline for an apprentice's final practice exam. The apprentice will decide how to implement the requirements based on the specification provided in `project_specification.md`.

When working in this repository:
- Focus on **refining and clarifying requirements** from a customer perspective
- Do NOT implement code unless explicitly asked to create examples or prototypes
- Maintain the specification document with clear, testable requirements
- Think as an IT Manager defining "what" is needed, not "how" to build it

## Project Overview

**Dynamic Central Configuration Manager (DCCM)** is a two-tier configuration management system built with Python and NiceGUI. The system uses a template-first approach where uploading a JSON/YAML schema automatically generates a user-friendly validation form.

## Architecture

### Two-Tier Model

The system consists of two loosely coupled components communicating via a shared filesystem:

1. **Management Tier (NiceGUI/Python)**: Handles write operations, complex validation, RBAC, and dynamic form generation. Accessed by administrators and editors.

2. **Retrieval Tier (Company Web Server)**: Serves validated configuration files as static content via HTTP GET using the company's standard web server solution. Accessed by consuming applications at `http://[config-host]/[filename].json`.

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

- `min` / `max` / `regex`: Range and pattern enforcement
- `options`: Renders standard dropdown from explicit list
- `lookup_file`: Renders searchable dropdown populated from external local .txt file. Supports one value per line, or multiple columns with delimiter (first column used as values).
- `separator`: Defines delimiter character for delimited files (e.g., `;`, `,`, `\t`). Used with `lookup_file`.
- `multi_select`: When `true`, enables multi-select mode for `lookup_file` or `options` dropdowns.

### Owner-Centric RBAC

Access control is based on `SSO_USERNAME` environment variable:

- **Owner**: Initial uploader automatically assigned as Owner. **Multiple Owners supported** - existing Owners can add additional Owners. Full control over templates and configurations. Stored as metadata.
- **Authorized User (Editor)**: Granted access via Owner delegation (Support Unit lookup or manual list). **Can only update configuration values** through forms, cannot upload or modify templates.
- **Config Administrator**: Explicitly assigned role with full control including deletion of any template or configuration

### Template Upload & Validation

The system has two distinct workflows: template upload and configuration creation.

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
  value: ""

# Type inference with optional explicit validation
service_timeout_ms:
  value: 3000
  type_check: 'integer'
  min: 1000
  max: 15000

# Optional regex validation
debug_log_level:
  value: INFO
  regex: '^(INFO|WARN|DEBUG|ERROR)$'

# Predefined dropdown
region_deployment:
  options:
    - US-EAST-1
    - EU-CENTRAL-1
    - ASIA-SOUTHEAST-2

# External file lookup (single-select searchable dropdown)
file_instance:
  lookup_file: /etc/config/available_instances.txt
  # Reads local .txt file: one value per line or columns with separator

# External file lookup with multi-select and delimiter
support_units:
  lookup_file: /etc/config/support_groups.txt
  separator: ";"
  multi_select: true
  # File contains: SU_NAME;LOGIN_NAME (only first column shown)

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

### Lookup File Processing

When encountering `lookup_file` directives:
- Only local .txt files are supported
- Read the specified file line-by-line
- File format options:
  - **Simple**: One value per line (entire line becomes the option)
  - **Delimited**: Multiple columns with `separator` character - only first column used as dropdown value
- Separator support:
  - Use `separator` keyword to define delimiter (e.g., `;`, `,`, `\t`)
  - Parse each line, extract first column before delimiter
  - Example: `IT-Support;john.doe` with `;` separator → shows only `IT-Support`
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
