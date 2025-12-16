# Frappe Framework Reference

## Locations

**Framework:**

- Frappe: `/home/frappe/frappe-bench/apps/frappe/frappe/`
- ERPNext: `/home/frappe/frappe-bench/apps/erpnext/erpnext/`

**Apps:**

- `/home/frappe/frappe-bench/apps/[app_name]/[app_name]/`

**Sites:**

- `/home/frappe/frappe-bench/sites/[site_name]/`

## Core Modules

**Frappe Framework:**

```
frappe/
├── api/              # API handlers
├── desk/             # Desk UI (forms, lists, reports)
├── model/            # Data model (Document, BaseDocument)
├── database/         # Database abstraction
├── website/          # Website builder
├── printing/         # Print formats
├── integrations/     # Third-party integrations
└── utils/            # Utility functions
```

## DocType File Pattern

**Naming Convention:**

- DocType: "Sales Invoice"
- File: `sales_invoice`
- Table: `tabSales Invoice`
- Child: `tabSales Invoice Item`

**File Structure:**

```
[app]/[module]/doctype/[doctype_name]/
├── [doctype_name].json
├── [doctype_name].py
└── [doctype_name].js
```

**Real Example:**

```
payments/payment_gateways/doctype/code_payment_gateways/
├── code_payment_gateways.json
├── code_payment_gateways.py
└── code_payment_gateways.js
```

## Hooks System

**Common Hooks (hooks.py):**

```python
# App metadata
app_name = "app_name"
app_title = "App Title"

# Installation
before_install = "app.utils.before_install"
after_install = "app.utils.after_install"

# Extend DocType classes
extend_doctype_class = {
    "Web Form": "app.overrides.webform.CustomWebForm"
}

# Document events
doc_events = {
    "DocType": {
        "validate": "app.module.handler",
        "on_submit": ["app.module.handler1", "app.module.handler2"]
    },
    "*": {
        "on_change": ["app.module.global_handler"]
    }
}

# Override methods
override_whitelisted_methods = {
    "frappe.method.path": "app.overrides.custom_method"
}

# Scheduler
scheduler_events = {
    "hourly": ["app.tasks.hourly_task"],
    "daily": ["app.tasks.daily_task"],
    "all": ["app.tasks.all_task"]
}

# Fixtures (export data)
fixtures = ["Custom Field", "Property Setter"]
```

**Real Example (payments/hooks.py):**

```python
extend_doctype_class = {
    "Web Form": "payments.overrides.payment_webform.PaymentWebForm"
}

scheduler_events = {
    "all": [
        "payments.payment_gateways.doctype.razorpay_settings.razorpay_settings.capture_payment"
    ]
}

override_whitelisted_methods = {
    "frappe.website.doctype.web_form.web_form.accept":
        "payments.overrides.payment_webform.accept"
}
```

**Real Example (lms/hooks.py):**

```python
doc_events = {
    "*": {
        "on_change": ["lms.lms.doctype.lms_badge.lms_badge.process_badges"]
    },
    "User": {
        "validate": "lms.lms.user.validate_username_duplicates",
        "after_insert": "lms.lms.user.after_insert"
    }
}

scheduler_events = {
    "hourly": [
        "lms.lms.api.update_course_statistics"
    ],
    "daily": [
        "lms.lms.doctype.lms_payment.lms_payment.send_payment_reminder"
    ]
}
```

## Bench Commands

**App Management:**

```bash
bench new-app [app_name]
bench get-app [app_name]
bench --site [site] install-app [app_name]
bench --site [site] uninstall-app [app_name]
```

**Development:**

```bash
bench --site [site] migrate
bench build
bench restart
bench --site [site] clear-cache
bench watch
```

## Integration Patterns

**Check Installed Apps:**

```python
installed_apps = frappe.get_installed_apps()
if "payments" in installed_apps:
    # Use payments features
    pass
```

**Create Integration Request:**

```python
from frappe.integrations.utils import create_request_log

integration_request = create_request_log(
    kwargs,
    service_name="Manual Payment",
    name=kwargs.get("order_id")
)
```

**Call Hook Methods:**

```python
from frappe.utils import call_hook_method

call_hook_method("payment_gateway_enabled", gateway="Manual Payment")
```
