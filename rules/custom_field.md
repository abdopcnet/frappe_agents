# Custom Field Reference

## Creation Methods

### Via UI

**Path:** Desk → Customize → Customize Form → Add Row

### Via Python

**Pattern:**

```python
import frappe

if not frappe.db.exists('Custom Field',
                       {'dt': 'Sales Invoice', 'fieldname': 'custom_tax_id'}):
    frappe.get_doc({
        'doctype': 'Custom Field',
        'dt': 'Sales Invoice',
        'label': 'Tax ID',
        'fieldname': 'custom_tax_id',
        'fieldtype': 'Data',
        'insert_after': 'customer_name',
        'reqd': 1
    }).insert()
    frappe.db.commit()
```

**Real Example (payments/utils.py):**

```python
def make_custom_fields():
    if not frappe.get_meta("Web Form").has_field("payments_tab"):
        create_custom_fields({
            "Web Form": [
                {
                    "fieldname": "payments_tab",
                    "fieldtype": "Tab Break",
                    "label": "Payments",
                    "insert_after": "custom_css",
                },
                {
                    "fieldname": "accept_payment",
                    "fieldtype": "Check",
                    "label": "Accept Payment",
                    "insert_after": "payments",
                    "default": "0"
                }
            ]
        })
```

## Field Types

**Common Types:**

- `Data` - Single line text (140 chars)
- `Text` - Multi-line text
- `Link` - Link to DocType (requires `options`)
- `Select` - Dropdown (options: `Option1\nOption2`)
- `Date` - Date picker
- `Currency` - Money field
- `Float` - Decimal number
- `Check` - Checkbox (0 or 1)
- `Table` - Child table (requires `options`)

## Essential Properties

**Required:**

```python
{
    'fieldname': 'custom_field_name',  # Must start with custom_
    'label': 'Field Label',
    'fieldtype': 'Data',
    'insert_after': 'existing_field',  # Position
}
```

**Common Properties:**

```python
{
    'reqd': 1,                         # Mandatory
    'hidden': 1,                       # Hidden
    'read_only': 1,                    # Read-only
    'default': 'Default Value',        # Default value
    'options': 'DocType Name',         # For Link/Table
    'in_list_view': 1,                 # Show in list
    'depends_on': 'eval:doc.status',   # Conditional display
    'fetch_from': 'customer.tax_id'     # Auto-fetch
}
```

## Property Setter

**Modify Existing Field:**

```python
frappe.make_property_setter({
    'doctype': 'Sales Invoice',
    'fieldname': 'customer_name',
    'property': 'reqd',
    'value': '1',
    'property_type': 'Check'
})
```

## Bulk Creation

**Pattern:**

```python
def create_custom_fields():
    custom_fields = {
        'Sales Invoice': [
            {
                'fieldname': 'custom_tax_id',
                'label': 'Tax ID',
                'fieldtype': 'Data',
                'insert_after': 'customer_name'
            }
        ]
    }

    for doctype, fields in custom_fields.items():
        for field in fields:
            if not frappe.db.exists('Custom Field',
                                   {'dt': doctype, 'fieldname': field['fieldname']}):
                frappe.get_doc({
                    'doctype': 'Custom Field',
                    'dt': doctype,
                    **field
                }).insert()
    frappe.db.commit()
```

**Install Hook:**

```python
# hooks.py
after_install = "app.utils.make_custom_fields"
```

## Naming Rules

**Required:**

- ✅ Prefix: `custom_`
- ✅ Descriptive: `custom_tax_id`, `custom_delivery_date`
- ❌ Avoid: `custom_field1`, `custom_date`
