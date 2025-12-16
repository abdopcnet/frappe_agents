# Frappe Web Pages & Templates Reference

## File Locations

**Web Pages:**

- `[app]/www/[page].html` + `[app]/www/[page].py`
- `[app]/www/[page]/index.html` + `[app]/www/[page]/index.py`

**Print Formats:**

- `[app]/[module]/print_format/[name]/[name].html`

**Portal Pages:**

- `[app]/templates/pages/[page].html` + `[app]/templates/pages/[page].py`

**Real Example (payments):**

```
payments/templates/pages/manual_payment.py
payments/templates/pages/manual_payment.html
```

## Jinja2 Basics

**Syntax:**

```jinja2
{{ doc.field }}                    {# Variable #}
{% if condition %}...{% endif %}   {# Condition #}
{% for item in doc.items %}...{% endfor %}  {# Loop #}
{{ frappe.format_value(doc.amount, {'fieldtype': 'Currency'}) }}
```

## Web Page Structure

**HTML Template:**

```html
{% extends "templates/web.html" %} {% block title %}Page Title{% endblock %} {%
block page_content %}
<div>{{ content }}</div>
{% endblock %}
```

**Python Context:**

```python
# www/page.py
import frappe

def get_context(context):
    context.items = frappe.get_all('Item', fields=['name', 'item_name'])
    return context
```

**Real Example (manual_payment.py):**

```python
import frappe

no_cache = True

def get_context(context):
    context.no_cache = True
    context.token = frappe.form_dict.get("token")
    context.code = frappe.form_dict.get("code")
    context.amount = frappe.form_dict.get("amount")
    context.currency = frappe.form_dict.get("currency")
    context.current_user = frappe.session.user

    # Get Code Payment Gateways
    code_gateway_name = frappe.db.get_value(
        "Code Payment Gateways",
        {"enabled": 1},
        "name",
        order_by="creation desc"
    )
    if code_gateway_name:
        code_gateway = frappe.get_doc("Code Payment Gateways", code_gateway_name)
        context.code_gateway_enabled = code_gateway.enabled
        # Get user codes...
    return context
```

## Print Format

**Structure:**

```html
<div class="print-format">
  <h1>{{ doc.name }}</h1>
  <table>
    {% for item in doc.items %}
    <tr>
      <td>{{ item.item_name }}</td>
      <td>{{ item.qty }}</td>
    </tr>
    {% endfor %}
  </table>
</div>

<style>
  .print-format {
    font-family: Arial;
  }
  table {
    width: 100%;
    border-collapse: collapse;
  }
</style>
```

## Email Template

**Send Email:**

```python
frappe.sendmail(
    recipients=['user@example.com'],
    subject='Invoice {{ doc.name }}',
    message=frappe.render_template(template, {'doc': doc}),
    reference_doctype='Sales Invoice',
    reference_name=doc.name
)
```

## Web Form

**Create:**

```python
frappe.get_doc({
    'doctype': 'Web Form',
    'title': 'Feedback',
    'route': 'feedback',
    'doc_type': 'Customer Feedback'
}).insert()
```

## Route Parameters

**Dynamic Routes:**

```python
# www/product/[item_code].py
def get_context(context):
    item_code = frappe.form_dict.item_code
    context.item = frappe.get_doc("Item", item_code)
```

## Website Routes (hooks.py)

**Route Rules:**

```python
website_route_rules = [
    {"from_route": "/lms/<path:app_path>", "to_route": "lms"},
    {"from_route": "/courses/<course_name>/<certificate_id>", "to_route": "certificate"}
]

website_redirects = [
    {"source": "/update-profile", "target": "/edit-profile"},
    {"source": "/courses", "target": "/lms/courses"}
]
```
