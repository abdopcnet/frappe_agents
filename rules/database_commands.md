# Database Commands Reference

## Access Methods

**Command Line:**

```bash
bench --site [site_name] mariadb
bench --site [site_name] mariadb -e "SELECT * FROM \`tabUser\` LIMIT 10;"
```

**Python Console:**

```bash
bench --site [site_name] console
>>> import frappe
>>> frappe.db.get_value('User', 'Administrator', 'email')
```

## Schema Inspection

**Table Structure:**

```bash
DESCRIBE \`tabSales Invoice\`;
SHOW CREATE TABLE \`tabSales Invoice\`;
SHOW INDEX FROM \`tabSales Invoice\`;
```

**Table Naming:**

- Parent: `tabDocType Name`
- Child: `tabChild DocType Name`
- Single: `tabSingles` (stores all single values)

## Frappe DB API

**Get Value:**

```python
# Single field
value = frappe.db.get_value('DocType', 'name', 'field')

# Multiple fields
values = frappe.db.get_value(
    'DocType',
    'name',
    ['field1', 'field2'],
    as_dict=True
)

# With filters
name = frappe.db.get_value(
    "Code Payment Gateways",
    {"enabled": 1},
    "name",
    order_by="creation desc"
)
```

**Get List:**

```python
items = frappe.db.get_list(
    'DocType',
    filters={
        'field': 'value',
        'status': ['in', ['Active', 'Pending']]
    },
    fields=['name', 'field'],
    order_by='name desc',
    limit=10
)
```

**Set Value:**

```python
# Single field
frappe.db.set_value('DocType', 'name', 'field', 'value')

# Multiple fields
frappe.db.set_value('DocType', 'name', {
    'field1': 'value1',
    'field2': 'value2'
})

frappe.db.commit()
```

**SQL Query:**

```python
# With parameters (safe from SQL injection)
result = frappe.db.sql("""
    SELECT * FROM \`tabDocType\`
    WHERE field = %s
    AND date >= %s
""", (value, date), as_dict=True)

# With filters dict
result = frappe.db.sql("""
    SELECT * FROM \`tabDocType\`
    WHERE field = %(field)s
    AND date >= %(from_date)s
""", filters, as_dict=True)
```

**Other Operations:**

```python
# Exists
if frappe.db.exists('DocType', 'name'):
    pass

# Count
count = frappe.db.count('DocType', filters={'field': 'value'})

# Delete
frappe.db.delete('DocType', {'name': 'name'})
```

## Common Queries

**DocTypes:**

```sql
SELECT name, module, custom FROM \`tabDocType\` ORDER BY name;
```

**Custom Fields:**

```sql
SELECT dt, fieldname, label, fieldtype
FROM \`tabCustom Field\`
WHERE dt = 'Sales Invoice';
```

**User Roles:**

```sql
SELECT u.name, hr.role
FROM \`tabUser\` u
JOIN \`tabHas Role\` hr ON hr.parent = u.name
WHERE u.name = 'user@example.com';
```

**Installed Apps:**

```python
installed_apps = frappe.get_installed_apps()
# Returns: ['frappe', 'erpnext', 'hrms', 'lms', 'payments', ...]
```

## Safety Rules

**Best Practices:**

- ✅ Always backup before direct DB operations
- ✅ Use parameters to prevent SQL injection
- ✅ Test queries with LIMIT first
- ✅ Use `frappe.db.*` methods when possible
- ❌ Never modify core tables directly
