# Export Custom Fields App - Development Guide

## Overview

Export Custom Fields is a Frappe app that simplifies exporting Frappe customizations (custom fields, scripts, property setters) to version control. It adds "Export to Module" buttons to various DocTypes for easy customization management.

For general Frappe guidelines, see `frappe.md`. For general AI guidelines, see `AGENTS.md`.

---

## 📋 Table of Contents

1. [App Architecture](#app-architecture)
2. [Core Features](#core-features)
3. [Export Functionality](#export-functionality)
4. [File Structure](#file-structure)
5. [Usage Patterns](#usage-patterns)
6. [Developer Notes](#developer-notes)

---

## App Architecture

### Application Structure

```
/home/frappe/frappe-bench/apps/export_custom_fields/
├── export_custom_fields/
│   ├── hooks.py              # App configuration
│   ├── public/
│   │   └── js/               # Client-side scripts
│   │       ├── customize_form.js
│   │       ├── server_script.js
│   │       ├── client_script.js
│   │       ├── custom_field.js
│   │       ├── property_setter.js
│   │       └── custom_html_block.js
│   └── modules.txt
└── README.md
```

### Key Concepts

**Purpose:** Export Frappe customizations to app directories for:
- Version control (Git tracking)
- Deployment across environments
- Backup and recovery
- Team collaboration

**Requirements:**
- Developer mode must be enabled
- Module assignment required for most exports
- Frappe Framework v15+

---

## Core Features

### 1. Export Custom Fields

**DocType:** `Custom Field`

**Functionality:** Export all custom fields for a specific module

**Button Location:** Custom Field form (in developer mode)

**Export Path:**
```
{app_name}/{module_name}/custom/{doctype_name}.json
```

**Example:**
```json
// erpnext/hr/custom/employee.json
{
    "custom_fields": [
        {
            "fieldname": "custom_employee_code",
            "label": "Employee Code",
            "fieldtype": "Data",
            "insert_after": "employee_name"
        }
    ],
    "property_setters": [...]
}
```

### 2. Export Property Setters

**DocType:** `Property Setter`

**Functionality:** Export property modifications by module

**Export Path:**
```
{app_name}/{module_name}/custom/{doctype_name}.json
```

**Use Case:** Track field property changes (hidden, mandatory, read-only, etc.)

### 3. Export Server Scripts

**DocType:** `Server Script`

**Functionality:** One-click export of server-side Python scripts

**Export Path:**
```
{app_name}/fixtures/server_script.json
```

**Features:**
- Complete script code
- Module assignments
- Execution order (idx)
- All metadata

### 4. Export Client Scripts

**DocType:** `Client Script`

**Functionality:** Export client-side JavaScript customizations

**Export Path:**
```
{app_name}/fixtures/client_script.json
```

**Features:**
- JavaScript code
- DocType assignments
- Event triggers
- Enabled status

### 5. Export Custom HTML Blocks

**DocType:** `Custom HTML Block`

**Functionality:** Export custom HTML/CSS/JS blocks

**Export Path:**
```
{app_name}/fixtures/custom_html_block.json
```

**Includes:**
- HTML content
- CSS styles
- JavaScript code
- Block configuration

### 6. Export from Customize Form

**DocType:** `Customize Form`

**Functionality:** Bulk export of all customizations for a module

**Features:**
- Module-based filtering
- Sync on migrate option
- Exports custom fields + property setters
- One-click bulk export

**Options:**
```python
{
    "module_to_export": "HR",  # Target module
    "sync_on_migrate": 1       # Auto-sync during migrations
}
```

---

## Export Functionality

### How It Works

**1. Client-Side Hooks:**
```javascript
// Added via hooks.py doctype_js
frappe.ui.form.on('Custom Field', {
    refresh: function(frm) {
        if (frappe.boot.developer_mode && frm.doc.module) {
            frm.add_custom_button(__('📦 Export to Module'), function() {
                export_custom_fields_to_module(frm.doc.module);
            });
        }
    }
});
```

**2. Server-Side Export:**
```python
# Python controller exports to JSON
import json
import os

def export_to_module(module, doctype):
    # Get all custom fields for module
    fields = frappe.get_all("Custom Field", 
                           filters={"module": module, "dt": doctype})
    
    # Export to app directory
    path = f"{app_path}/{module}/custom/{doctype}.json"
    with open(path, 'w') as f:
        json.dump(fields, f, indent=4)
```

### Export Formats

**Standard Frappe JSON:**
- Human-readable with indentation
- UTF-8 encoding for international characters
- Compatible with `bench migrate`
- Compatible with `bench export-fixtures`

**File Organization:**
- Grouped by module and DocType
- Maintains Frappe directory structure
- Ready for version control

---

## File Structure

### Custom Fields Export Structure

```
app_name/
├── module_name/
│   └── custom/
│       ├── doctype1.json      # Custom fields + property setters
│       ├── doctype2.json
│       └── doctype3.json
```

**Example:**
```
erpnext/
├── hr/
│   └── custom/
│       ├── employee.json
│       ├── leave_application.json
│       └── attendance.json
└── selling/
    └── custom/
        └── sales_order.json
```

### Fixtures Export Structure

```
app_name/
└── fixtures/
    ├── server_script.json
    ├── client_script.json
    └── custom_html_block.json
```

### Sync on Migrate

**Purpose:** Automatically apply exported customizations during `bench migrate`

**Configuration in hooks.py:**
```python
# hooks.py
fixtures = [
    {
        "dt": "Custom Field",
        "filters": [["module", "=", "HR"]]
    },
    "Server Script",
    "Client Script"
]
```

**Result:** Customizations sync automatically on:
```bash
bench migrate
bench --site [sitename] migrate
```

---

## Usage Patterns

### 1. Export Custom Fields by Module

```bash
1. Navigate to: Custom Field list
2. Open any custom field with module assigned
3. Click: 📦 Export to Module button
4. All custom fields for that module export to:
   {app}/module/custom/{doctype}.json
```

### 2. Export Scripts

```bash
1. Navigate to: Server Script / Client Script form
2. Ensure module is assigned
3. Click: 📦 Export to Module button
4. Script exports to: {app}/fixtures/
```

### 3. Bulk Export from Customize Form

```bash
1. Navigate to: Customize Form for any DocType
2. Click: 📦 Export to Module button
3. Select module to export
4. Enable "Sync on Migrate" if needed
5. All customizations for that module export
```

### 4. Version Control Workflow

```bash
# After exporting
cd /home/frappe/frappe-bench/apps/{app_name}

# Check exported files
git status

# Add to version control
git add {module}/custom/*.json
git add fixtures/*.json

# Commit
git commit -m "Export custom fields for HR module"

# Push to repository
git push origin main
```

### 5. Deploy to Another Environment

```bash
# On target environment
cd /home/frappe/frappe-bench

# Pull latest code
git pull

# Migrate (auto-syncs if fixtures configured)
bench --site [sitename] migrate

# Or manually import
bench --site [sitename] import-doc {path/to/file.json}
```

---

## Developer Notes

### Hooks Configuration

**File:** `/home/frappe/frappe-bench/apps/export_custom_fields/export_custom_fields/hooks.py`

**Key Configuration:**
```python
app_name = "export_custom_fields"
app_title = "Export Custom Fields"
app_publisher = "abdopcnet"
app_license = "mit"

# Add JS to DocTypes
doctype_js = {
    "Customize Form": "public/js/customize_form.js",
    "Server Script": "public/js/server_script.js",
    "Client Script": "public/js/client_script.js",
    "Custom Field": "public/js/custom_field.js",
    "Property Setter": "public/js/property_setter.js",
    "Custom HTML Block": "public/js/custom_html_block.js"
}
```

### Client-Side Implementation

**Pattern:**
```javascript
// public/js/custom_field.js
frappe.ui.form.on('Custom Field', {
    refresh: function(frm) {
        // Only show in developer mode
        if (!frappe.boot.developer_mode) return;
        
        // Require module assignment
        if (!frm.doc.module) return;
        
        // Add export button
        frm.add_custom_button(__('📦 Export to Module'), function() {
            frappe.call({
                method: 'export_custom_fields.api.export_custom_fields',
                args: {
                    module: frm.doc.module
                },
                callback: function(r) {
                    frappe.msgprint(__('Exported successfully'));
                }
            });
        });
    }
});
```

### Server-Side API Pattern

```python
import frappe
import json
import os

@frappe.whitelist()
def export_custom_fields(module):
    """Export all custom fields for a module."""
    if not frappe.conf.get('developer_mode'):
        frappe.throw('Developer mode required')
    
    # Get app path
    app = frappe.get_module_app(module)
    app_path = frappe.get_app_path(app)
    
    # Get all doctypes with custom fields in this module
    doctypes = frappe.get_all('Custom Field', 
                             filters={'module': module},
                             pluck='dt',
                             distinct=True)
    
    for doctype in doctypes:
        export_doctype_customizations(app_path, module, doctype)
    
    return {'exported': len(doctypes)}

def export_doctype_customizations(app_path, module, doctype):
    """Export custom fields and property setters for a doctype."""
    
    # Get custom fields
    custom_fields = frappe.get_all('Custom Field',
                                  filters={'module': module, 'dt': doctype},
                                  fields=['*'])
    
    # Get property setters
    property_setters = frappe.get_all('Property Setter',
                                     filters={'module': module, 'doc_type': doctype},
                                     fields=['*'])
    
    # Prepare export data
    data = {
        'custom_fields': custom_fields,
        'property_setters': property_setters,
        'sync_on_migrate': True
    }
    
    # Create directory
    custom_dir = os.path.join(app_path, module.lower(), 'custom')
    os.makedirs(custom_dir, exist_ok=True)
    
    # Export to JSON
    file_path = os.path.join(custom_dir, f'{doctype.lower().replace(" ", "_")}.json')
    with open(file_path, 'w', encoding='utf-8') as f:
        json.dump(data, f, indent=4, ensure_ascii=False)
```

### Best Practices

**1. Always Use Developer Mode:**
```python
# Check in hooks or controllers
if not frappe.conf.get('developer_mode'):
    return  # Don't show export buttons
```

**2. Module Assignment:**
```python
# Validate module before export
if not doc.module:
    frappe.throw('Please assign a module before exporting')
```

**3. File Overwriting:**
```python
# Warn before overwriting
if os.path.exists(file_path):
    frappe.msgprint('Existing file will be overwritten', indicator='orange')
```

**4. Fixtures Configuration:**
```python
# In hooks.py for auto-sync
fixtures = [
    {
        "dt": "Custom Field",
        "filters": [["module", "in", ["HR", "Selling"]]]
    }
]
```

---

## Common Use Cases

### 1. Version Control

**Workflow:**
```bash
# Export customizations
Export via UI buttons

# Track in Git
git add */custom/*.json fixtures/*.json
git commit -m "Add custom fields for employee badges"
git push
```

### 2. Environment Deployment

**Dev → Staging → Production:**
```bash
# Development
1. Create customizations
2. Export to module
3. Commit to Git

# Staging
1. Pull latest code
2. bench migrate (auto-syncs)

# Production
1. Pull tested code
2. bench migrate
```

### 3. Backup Before Changes

```bash
# Before major customization work
1. Export all customizations
2. Commit: "Backup before refactor"
3. Make changes safely
4. Can revert if needed
```

### 4. Team Collaboration

```bash
# Developer A
1. Creates custom fields
2. Exports to module
3. Pushes to branch

# Developer B
1. Pulls branch
2. Runs migrate
3. Customizations appear automatically
```

---

## Installation

```bash
# Install app
bench get-app export_custom_fields https://github.com/abdopcnet/export_custom_fields
bench --site [sitename] install-app export_custom_fields

# Enable developer mode
bench --site [sitename] set-config developer_mode 1

# Clear cache
bench clear-cache

# Restart
bench restart
```

---

## Important Notes

**Requirements:**
- ✅ Developer mode must be enabled
- ✅ Module assignment required (most features)
- ✅ Appropriate permissions needed
- ✅ Frappe Framework v15+

**Limitations:**
- Export buttons only visible in developer mode
- Cannot export without module assignment
- File overwriting occurs without confirmation
- No selective export within module

**Best Practices:**
- Always commit to version control after export
- Use "Sync on Migrate" for automatic deployment
- Export before major changes (backup)
- Test in staging before production deployment

---

## Quick Commands

```bash
# Enable developer mode
bench --site [sitename] set-config developer_mode 1

# Check exported files
ls apps/{app_name}/{module}/custom/
ls apps/{app_name}/fixtures/

# Import manually
bench --site [sitename] import-doc apps/{app}/fixtures/server_script.json

# Migrate (auto-import if configured)
bench --site [sitename] migrate
```

---

**Repository:** https://github.com/abdopcnet/export_custom_fields  
**License:** MIT  
**Version:** 10.12.2025
