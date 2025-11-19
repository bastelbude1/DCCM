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

**Type Inference**: Automatically renders appropriate inputs based on initial values (e.g., number input for integers)

**Explicit Schema Rules**: Recognized keywords that control form generation and validation:

- `min` / `max` / `regex`: Range and pattern enforcement
- `options`: Renders standard dropdown from explicit list
- `lookup_file`: Renders searchable dropdown populated from external file (line-by-line reading)

### Owner-Centric RBAC

Access control is based on `SSO_USERNAME` environment variable:

- **Owner**: Automatically assigned to initial uploader (stored as metadata), can Update/Modify and delegate access
- **Authorized User (Editor)**: Granted access via Owner delegation (Support Unit lookup or manual list), can Update/Modify
- **Config Administrator**: Explicitly assigned role with full control including deletion

**Critical Constraint**: Block save operation if Owner attempts to delegate to a Support Unit they are a member of.

### Configuration Naming & Collision Handling

- Configurations are saved using free-text names provided by users
- Before save, check shared filesystem for name collision
- If collision exists, prompt user to confirm overwrite
- Final files are served at predictable URLs: `http://[config-host]/[free-text-name].json`

## Template Structure Example

```yaml
# Type inference with explicit validation
service_timeout_ms:
  value: 3000
  type_check: 'integer'
  min: 1000
  max: 15000

# Regex validation
debug_log_level:
  value: INFO
  regex: '^(INFO|WARN|DEBUG|ERROR)$'

# Predefined dropdown
region_deployment:
  options:
    - US-EAST-1
    - EU-CENTRAL-1
    - ASIA-SOUTHEAST-2

# External file lookup (searchable dropdown)
database_instance:
  lookup_file: /etc/config/available_databases.txt

# Metadata (not rendered in form)
owner:
  value: user_sso_name
support_units:
  lookup_file: /etc/config/support_groups.txt
```

## Key Implementation Considerations

### Module Separation

Maintain clear separation between the three core modules (Identity/Authorization, Data Validation, Persistence). Each module should be independently testable.

### Form Generator Recursion

The form builder must handle nested structures recursively. Each field in the template should be parsed to determine:
1. What UI component to render (type inference + explicit keywords)
2. What validation rules to apply
3. Whether to include in final output or treat as metadata

### Metadata Storage

Owner and delegation information must be stored separately from the output configuration files. This metadata is required for authorization checks but should never appear in the final JSON/YAML served to consuming applications.

### Lookup File Processing

When encountering `lookup_file` directives:
- Read the specified file line-by-line
- Each line becomes an option in the searchable dropdown
- Handle file read errors gracefully
- Cache file contents if performance becomes an issue

### Environment-Based Identity

The system relies on `SSO_USERNAME` environment variable being set by upstream authentication (reverse proxy, SSO gateway). The application should fail gracefully if this variable is not present.

## Technology Stack

- **Backend Framework**: Python with NiceGUI for dynamic UI generation
- **Retrieval Server**: Company standard web server solution (e.g., Apache) for static file serving - final definition to be determined by company infrastructure
- **Identity Source**: `SSO_USERNAME` environment variable
- **Data Format**: YAML for templates (readability), JSON for output configurations
- **Persistence**: Shared filesystem between Management and Retrieval tiers
