# DocType Commands Reference

## File Structure

**Standard Pattern:**

```
/apps/[app]/[app]/[module]/doctype/[doctype_name]/
├── __init__.py
├── [doctype_name].json          # DocType definition
├── [doctype_name].py            # Python controller
├── [doctype_name].js            # Client script
├── [doctype_name]_list.js       # List view (optional)
└── test_[doctype_name].py       # Unit tests
```

**Real Example (Code Payment Gateways):**

```
/apps/payments/payments/payment_gateways/doctype/code_payment_gateways/
├── code_payment_gateways.json
├── code_payment_gateways.py
└── code_payment_gateways.js
```

## JSON Structure

**Essential Fields:**

```json
{
  "doctype": "DocType",
  "name": "Custom Order",
  "module": "My Module",
  "autoname": "field:order_id",
  "is_submittable": 1,
  "field_order": ["order_id", "customer", "items"],
  "fields": [
    {
      "fieldname": "order_id",
      "fieldtype": "Data",
      "label": "Order ID",
      "reqd": 1
    }
  ],
  "permissions": [
    {
      "role": "System Manager",
      "create": 1,
      "read": 1,
      "write": 1
    }
  ]
}
```

## Python Controller

**Basic Pattern:**

```python
import frappe
from frappe import _
from frappe.model.document import Document

class CustomOrder(Document):
    def validate(self):
        self.calculate_total()

    def on_submit(self):
        # Post-submit logic
        pass

    def on_cancel(self):
        # Cancel logic
        pass
```

**Real Example (CodePaymentGateways):**

```python
class CodePaymentGateways(Document):
    supported_currencies = ("USD", "EUR", "GBP", "INR", "AED", "SAR", "EGP")

    def on_update(self):
        if self.enabled:
            create_payment_gateway("Manual Payment")
            call_hook_method("payment_gateway_enabled", gateway="Manual Payment")

    def validate_transaction_currency(self, currency):
        if currency not in self.supported_currencies:
            frappe.throw(_("Currency '{0}' is not supported").format(currency))

    def get_payment_url(self, **kwargs):
        # Generate payment URL
        return url
```

**Real Example (LMSCourse):**

```python
class LMSCourse(Document):
    def validate(self):
        self.validate_published()
        self.validate_instructors()
        self.validate_video_link()
        self.validate_status()
        self.validate_payments_app()
        self.validate_certification()
        self.validate_amount_and_currency()
        self.image = validate_image(self.image)
        self.validate_card_gradient()

    def validate_payments_app(self):
        if self.paid_course:
            installed_apps = frappe.get_installed_apps()
            if "payments" not in installed_apps:
                frappe.throw(_("Please install the Payments App"))
```

## Field Types

**Common Types:**

- `Data` - Single line text (140 chars max)
- `Text` - Multi-line text
- `Link` - Link to DocType (requires `options`)
- `Select` - Dropdown (options: `Option1\nOption2`)
- `Date` - Date picker
- `Currency` - Money field
- `Float` - Decimal number
- `Check` - Checkbox (0 or 1)
- `Table` - Child table (requires `options`)

## Naming Rules

**Autoname Patterns:**

```json
{
  "autoname": "field:order_id", // Use field value
  "autoname": "naming_series:", // Naming series
  "autoname": "format:ORD-{YYYY}-{####}", // Format string
  "autoname": "Prompt" // User input
}
```

**Custom Naming (Python):**

```python
def autoname(self):
    from frappe.model.naming import make_autoname
    self.name = make_autoname('ORD-.YY.-.####')
```

## Bench Commands

**Reload DocType:**

```bash
bench --site [site] reload-doctype "DocType Name"
```

**Migrate:**

```bash
bench --site [site] migrate
```

**Clear Cache:**

```bash
bench --site [site] clear-cache
```

## Child DocTypes

**Parent-Child Relationship:**

```json
// Parent DocType
{
  "fieldname": "items",
  "fieldtype": "Table",
  "options": "Custom Order Item"
}

// Child DocType
{
  "istable": 1,
  "fields": [
    {
      "fieldname": "parent",
      "fieldtype": "Data",
      "hidden": 1
    },
    {
      "fieldname": "parenttype",
      "fieldtype": "Data",
      "hidden": 1
    },
    {
      "fieldname": "parentfield",
      "fieldtype": "Data",
      "hidden": 1
    }
  ]
}
```

## Single DocTypes

**Pattern:**

```json
{
  "issingle": 1,
  "name": "Settings Name"
}
```

**Access:**

```python
# Get single value
value = frappe.db.get_single_value("Settings Name", "field")

# Get document
doc = frappe.get_single("Settings Name")
```
