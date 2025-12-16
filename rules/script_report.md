# Script Report Reference

## File Structure

```
[app]/[module]/report/[report_name]/
├── [report_name].json    # Filters definition
├── [report_name].py      # Main logic
└── [report_name].js      # Optional client-side customization
```

## Python Structure

**Main Function:**

```python
import frappe
from frappe import _

def execute(filters=None):
    """
    Main report function
    Returns: columns, data, message, chart, report_summary
    """
    columns = get_columns()
    data = get_data(filters)
    return columns, data

def get_columns():
    """Define report columns"""
    return [
        {
            "fieldname": "name",
            "label": _("Name"),
            "fieldtype": "Link",
            "options": "DocType",
            "width": 150
        },
        {
            "fieldname": "total",
            "label": _("Total"),
            "fieldtype": "Currency",
            "width": 120
        }
    ]

def get_data(filters):
    """Fetch report data"""
    conditions = get_conditions(filters)

    return frappe.db.sql("""
        SELECT name, field, total
        FROM `tabDocType`
        WHERE docstatus = 1
        {conditions}
        ORDER BY creation DESC
    """.format(conditions=conditions), filters, as_dict=1)

def get_conditions(filters):
    """Build WHERE conditions"""
    conditions = ""
    if filters.get("from_date"):
        conditions += " AND date >= %(from_date)s"
    if filters.get("customer"):
        conditions += " AND customer = %(customer)s"
    return conditions
```

## Filters (JSON)

**Filter Definition:**

```json
{
  "filters": [
    {
      "fieldname": "from_date",
      "label": "From Date",
      "fieldtype": "Date",
      "reqd": 1,
      "default": "Today"
    },
    {
      "fieldname": "to_date",
      "label": "To Date",
      "fieldtype": "Date",
      "reqd": 1
    },
    {
      "fieldname": "customer",
      "label": "Customer",
      "fieldtype": "Link",
      "options": "Customer"
    }
  ]
}
```

## Column Types

**Common Types:**

- `Data` - Text
- `Link` - Clickable link (requires `options`)
- `Date` - Date format
- `Currency` - Money format (requires `options` for currency field)
- `Float` - Decimal number
- `Percent` - Percentage format
- `Select` - Dropdown

## Report Summary

**Summary Cards:**

```python
def execute(filters=None):
    columns = get_columns()
    data = get_data(filters)

    report_summary = [
        {
            "value": len(data),
            "label": _("Total Records"),
            "datatype": "Int"
        },
        {
            "value": sum(row.get("total", 0) for row in data),
            "label": _("Total Amount"),
            "datatype": "Currency",
            "currency": "EGP"
        }
    ]

    return columns, data, None, None, report_summary
```

## Chart Data

**Chart Definition:**

```python
def execute(filters=None):
    columns = get_columns()
    data = get_data(filters)
    chart = get_chart_data(data)

    return columns, data, None, chart

def get_chart_data(data):
    from collections import defaultdict
    monthly_data = defaultdict(float)

    for row in data:
        month = row.get("date").strftime("%b %Y")
        monthly_data[month] += row.get("total", 0)

    return {
        "data": {
            "labels": list(monthly_data.keys()),
            "datasets": [{
                "name": _("Sales"),
                "values": list(monthly_data.values())
            }]
        },
        "type": "bar",
        "colors": ["#7cd6fd"]
    }
```

## Common Patterns

**Date Range Filter:**

```python
def get_conditions(filters):
    conditions = ""
    if filters.get("from_date"):
        conditions += " AND posting_date >= %(from_date)s"
    if filters.get("to_date"):
        conditions += " AND posting_date <= %(to_date)s"
    return conditions
```

**Join Tables:**

```python
def get_data(filters):
    return frappe.db.sql("""
        SELECT
            si.name,
            si.customer,
            c.customer_name,
            si.grand_total
        FROM `tabSales Invoice` si
        INNER JOIN `tabCustomer` c ON c.name = si.customer
        WHERE si.docstatus = 1
    """, filters, as_dict=1)
```

**Child Table Data:**

```python
def get_data(filters):
    return frappe.db.sql("""
        SELECT
            si.name,
            sii.item_code,
            sii.qty,
            sii.amount
        FROM `tabSales Invoice` si
        INNER JOIN `tabSales Invoice Item` sii ON sii.parent = si.name
        WHERE si.docstatus = 1
    """, filters, as_dict=1)
```
