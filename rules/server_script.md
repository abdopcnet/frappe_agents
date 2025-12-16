# Server Script Reference

## File Locations

**DocType Controllers:**

- `/apps/[app]/[app]/[module]/doctype/[doctype]/[doctype].py`

**API Methods:**

- `/apps/[app]/[app]/[module]/api.py`
- `/apps/[app]/[app]/api.py`
- `/apps/[app]/[app]/[module]/templates/pages/[page].py`

**Hooks:**

- `/apps/[app]/[app]/hooks.py`

## Controller Structure

**Basic Pattern:**

```python
import frappe
from frappe import _
from frappe.model.document import Document

class DocTypeName(Document):
    def validate(self):
        """Called before save"""
        pass

    def on_update(self):
        """Called after save"""
        pass

    def on_submit(self):
        """Called after submit"""
        pass

    def on_cancel(self):
        """Called on cancel"""
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
        integration_request = create_request_log(kwargs, service_name="Manual Payment")
        # Build URL and return
        return url
```

## Lifecycle Methods

**Order of Execution:**

1. `validate()` - Before save
2. `before_save()` - Before save
3. `on_update()` - After save
4. `before_submit()` - Before submit
5. `on_submit()` - After submit
6. `on_cancel()` - On cancel
7. `on_trash()` - Before delete

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

## Whitelisted Methods

**Basic Pattern:**

```python
@frappe.whitelist()
def api_method(param1, param2):
    # Parse JSON if string
    if isinstance(param1, str):
        import json
        param1 = json.loads(param1)

    return {"result": "value"}
```

**Real Example (manual_payment.py):**

```python
@frappe.whitelist(allow_guest=True)
def confirm_manual_payment(token, code):
    """Confirm manual payment by validating code"""
    try:
        current_user = frappe.session.user
        if current_user == "Guest":
            return {"success": False, "message": "Please login"}

        integration_request = frappe.get_doc("Integration Request", token)
        payment_data = frappe.parse_json(integration_request.data)

        # Validation logic...

        return {
            "success": True,
            "redirect": redirect_to,
            "message": "Payment confirmed"
        }
    except Exception as e:
        frappe.log_error(f"Error: {str(e)}")
        return {"success": False, "message": str(e)}
```

**Query Function for Link Field Filtering (Child Table):**

When filtering link fields by child table fields (e.g., User by role from Has Role), create a query function:

```python
@frappe.whitelist()
@frappe.validate_and_sanitize_search_inputs
def get_students(doctype, txt, searchfield, start, page_len, filters):
    """Filter User link field to show only users with 'LMS Student' role
    
    Pattern based on frappe/core/doctype/role/role.py:get_users()
    and frappe/core/doctype/user/user.py:user_query()
    """
    # Get user names from child table (Has Role)
    student_users = frappe.get_all(
        "Has Role",
        filters={"role": "LMS Student", "parenttype": "User"},
        fields=["parent"],
        pluck="parent"
    )
    
    if not student_users:
        return []
    
    # Filter User by those names
    list_filters = {
        "enabled": 1,
        "docstatus": ["<", 2],
        "name": ["in", student_users]
    }
    
    or_filters = [[searchfield, "like", f"%{txt}%"]]
    if "name" in searchfield:
        or_filters += [[field, "like", f"%{txt}%"] for field in ("first_name", "middle_name", "last_name")]
    
    return frappe.get_list(
        "User",
        filters=list_filters,
        fields=["name", "full_name"],
        limit_start=start,
        limit_page_length=page_len,
        order_by="name asc",
        or_filters=or_filters,
        as_list=True,  # Returns [["name", "label"], ...]
    )
```

**Key Points:**
- Use `@frappe.validate_and_sanitize_search_inputs` decorator
- Function signature: `(doctype, txt, searchfield, start, page_len, filters)`
- Get child table data first with `frappe.get_all()`
- Filter parent DocType using `["in", child_data]`
- Return `as_list=True` for link field format
- Pattern: Get child table → Filter parent by child data → Return list

**With Rate Limiting:**

```python
from frappe.rate_limiter import rate_limit

@frappe.whitelist(allow_guest=True)
@rate_limit(key="web_form", limit=5, seconds=60, methods=["POST"])
def accept(web_form, data, docname=None):
    # Method implementation
    pass
```

## Extending DocType Classes

**Pattern (hooks.py):**

```python
extend_doctype_class = {
    "Web Form": "payments.overrides.payment_webform.PaymentWebForm",
    "Employee": "hrms.overrides.employee_master.EmployeeMaster"
}
```

**Real Example (PaymentWebForm):**

```python
from frappe.website.doctype.web_form.web_form import WebForm

class PaymentWebForm(WebForm):
    def validate(self):
        super().validate()
        if getattr(self, "accept_payment", False):
            self.validate_payment_amount()

    def get_payment_gateway_url(self, doc):
        if getattr(self, "accept_payment", False):
            controller = get_payment_gateway_controller(self.payment_gateway)
            # Build payment URL
            return url
```

**Real Example (EmployeeMaster):**

```python
from erpnext.setup.doctype.employee.employee import Employee

class EmployeeMaster(Employee):
    def autoname(self):
        naming_method = frappe.db.get_value("HR Settings", None, "emp_created_by")
        if naming_method == "Naming Series":
            set_name_by_naming_series(self)
        elif naming_method == "Employee Number":
            self.name = self.employee_number
```

## Document Events (Hooks)

**Pattern (hooks.py):**

```python
doc_events = {
    "DocType": {
        "validate": "app.module.handler",
        "on_submit": ["app.module.handler1", "app.module.handler2"],
        "on_update": "app.module.handler"
    },
    "*": {
        "on_change": ["app.module.global_handler"]
    }
}
```

**Real Example (lms/hooks.py):**

```python
doc_events = {
    "*": {
        "on_change": ["lms.lms.doctype.lms_badge.lms_badge.process_badges"]
    },
    "Discussion Reply": {
        "after_insert": "lms.lms.utils.handle_notifications",
        "validate": "lms.lms.utils.validate_discussion_reply"
    },
    "User": {
        "validate": "lms.lms.user.validate_username_duplicates",
        "after_insert": "lms.lms.user.after_insert"
    }
}
```

**Handler Function:**

```python
def handler_function(doc, method):
    """Handler receives doc and method name"""
    # Custom logic
    pass
```

## Override Whitelisted Methods

**Pattern (hooks.py):**

```python
override_whitelisted_methods = {
    "frappe.website.doctype.web_form.web_form.accept":
        "payments.overrides.payment_webform.accept"
}
```

**Real Example:**

```python
@frappe.whitelist(allow_guest=True)
@rate_limit(key="web_form", limit=5, seconds=60, methods=["POST"])
def accept(web_form, data, docname=None, for_payment=False):
    """Override Web Form accept method"""
    # Custom implementation
    web_form = frappe.get_doc("Web Form", web_form)
    # ... custom logic
    if for_payment:
        return web_form.get_payment_gateway_url(doc)
    return doc
```

## Database Operations

**Common Patterns:**

```python
# Get value
value = frappe.db.get_value("DocType", "name", "field")

# Get with filters
name = frappe.db.get_value(
    "Code Payment Gateways",
    {"enabled": 1},
    "name",
    order_by="creation desc"
)

# Get list
items = frappe.db.get_list(
    "DocType",
    filters={"field": "value"},
    fields=["name", "field"],
    order_by="name desc",
    limit=10
)

# Set value
frappe.db.set_value("DocType", "name", "field", "value")

# SQL query
result = frappe.db.sql("""
    SELECT * FROM `tabDocType`
    WHERE field = %s
""", (value,), as_dict=True)

# Check installed apps
installed_apps = frappe.get_installed_apps()
if "payments" not in installed_apps:
    frappe.throw(_("App not installed"))
```

## Error Handling

**Patterns:**

```python
# Throw error
frappe.throw(_("Error message"))

# Log error
frappe.log_error(f"Error: {str(e)}", "Error Title")

# Try-except
try:
    doc = frappe.get_doc("DocType", name)
    doc.submit()
except frappe.DoesNotExistError:
    return {"success": False, "message": "Not found"}
except Exception as e:
    frappe.log_error(frappe.get_traceback())
    return {"success": False, "message": str(e)}
```

## Scheduler Events

**Pattern (hooks.py):**

```python
scheduler_events = {
    "hourly": ["app.module.hourly_task"],
    "daily": ["app.module.daily_task"],
    "all": ["app.module.all_task"]
}
```

**Real Example (payments/hooks.py):**

```python
scheduler_events = {
    "all": [
        "payments.payment_gateways.doctype.razorpay_settings.razorpay_settings.capture_payment"
    ]
}
```

## Common Patterns

**Check Installed Apps:**

```python
installed_apps = frappe.get_installed_apps()
if "payments" in installed_apps:
    # Use payments app features
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

**Parse JSON:**

```python
data = frappe.parse_json(integration_request.data)
payment_amount = float(data.get("amount", 0))
```
