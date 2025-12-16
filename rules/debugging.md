# Debugging & Logging Reference

## Unified Logging Standards

**MANDATORY:** All logging must follow these unified patterns for consistency and easier debugging.

## Frontend Logging (JavaScript)

**Rule:** Use ONLY `console.log` with filename and short important results.

**Format:**

```javascript
console.log(`[filename.js] (short_important_result)`);
```

**Examples:**

```javascript
// ✅ CORRECT
console.log(`[payment_gateway.js] (Payment processed: ${payment_id})`);
console.log(`[form.js] (Validation passed)`);
console.log(`[api.js] (Request sent: ${method})`);

// ❌ WRONG - Don't use these
console.info(); // Don't use
console.warn(); // Don't use
console.error(); // Don't use (unless for actual errors)
console.log(long_array); // Don't log long arrays
console.log(entire_object); // Don't log entire objects
```

## Active Debugging Sessions

### LMS Signup Issue - December 14, 2025

**Problem:** User signup showing unwanted messages:

- "Please ask your administrator to verify your sign-up"
- "Please setup default outgoing Email Account from Settings > Email Account"

**Goal:** Create user with password "123123" and show it in success message, no email verification

**Files Modified:**

- `/home/frappe/frappe-bench/apps/lms/lms/lms/user.py` - Added debug logs

**Debug Logs Added:**

```python
frappe.log_error(f"[user.py] (Signup started for: {email})", "LMS Signup Debug")
frappe.log_error(f"[user.py] (Creating new user with enabled=1, password set)", "LMS Signup Debug")
frappe.log_error(f"[user.py] (Before insert - flags set: no_welcome_mail={user.flags.no_welcome_mail})", "LMS Signup Debug")
frappe.log_error(f"[user.py] (After insert - user created: {user.name}, email_sent={user.flags.get('email_sent')})", "LMS Signup Debug")
frappe.log_error(f"[user.py] (User fully created: {user.name}, returning success message)", "LMS Signup Debug")
```

**What to Check:**

1. Check Error Log in Frappe for "LMS Signup Debug" entries
2. Verify if `email_sent` flag is being set despite `no_welcome_mail` flag
3. Check if the return message is being overridden somewhere
4. Look for hooks in `lms/hooks.py` that might intercept user creation

**Solution Applied:**

1. ✅ Set password as string: `"123123"` instead of `123123`
2. ✅ Set `send_welcome_email: 0` in User doc creation
3. ✅ Set `user.flags.no_welcome_mail = True` before insert
4. ✅ Set `enabled: 1` to activate user immediately
5. ✅ Added comprehensive debug logging throughout signup flow

**Key Findings:**

- LMS has `after_insert` hook on User doctype that adds "LMS Student" role
- Core Frappe User tries to send welcome email if `send_welcome_email` is not explicitly set to 0
- Must set BOTH `send_welcome_email: 0` AND `flags.no_welcome_mail = True` to prevent email

**To View Debug Logs:**

- Go to: Desk → Error Log → Filter by "LMS Signup Debug"

**Guidelines:**

- Only use `console.log`
- Include filename in brackets: `[filename.js]`
- Show short important results only
- No long arrays or full objects
- Keep messages concise and meaningful

## Backend Logging (Python)

**Rule:** Use `frappe.log_error` with filename and short important results.

**Format:**

```python
frappe.log_error(f"[filename.py] (short_important_result)")
```

**Source:** `/home/frappe/frappe-bench/apps/frappe/frappe/utils/error.py`

**Example Pattern:**

```python
import frappe

try:
    prospect_name = frappe.db.get_value("Prospect", {"company_name": prospect.company_name})
    if not prospect_name:
        prospect.insert()
        prospect_name = prospect.name
except Exception as e:
    frappe.log_error(f"[file.py] create_customer_address: {str(e)}")
    pass
```

**Guidelines:**

- Use `frappe.log_error()` from `frappe.utils.error`
- Include filename in brackets: `[filename.py]`
- Include function/method name if relevant
- Show short important results only
- No long arrays or full objects
- Keep messages concise and meaningful
- Always catch exceptions properly with `except Exception as e:`

**Error Logging Pattern:**

```python
try:
    # Your code here
    result = some_operation()
    frappe.log_error(f"[module.py] operation_name: Success - {result}")
except Exception as e:
    frappe.log_error(f"[module.py] operation_name: {str(e)}")
    raise  # Re-raise if needed
```

## Logging Best Practices

1. **Filename Format:** Always use `[filename.ext]` format
2. **Short Results:** Only log essential information, not full data structures
3. **Context:** Include method/function name when relevant
4. **Consistency:** Use same format across all files
5. **No Verbose Logs:** Avoid logging entire arrays, objects, or long strings
6. **Error Context:** Include enough context to debug the issue

## Common Patterns

**Frontend - API Call:**

```javascript
frappe.call({
  method: "app.module.api.method",
  args: { param: value },
  callback: function (r) {
    console.log(`[api.js] (Method called: ${r.message})`);
  },
});
```

**Backend - Database Operation:**

```python
try:
    doc = frappe.get_doc("DocType", name)
    doc.save()
    frappe.log_error(f"[controller.py] save_doc: Saved {doc.name}")
except Exception as e:
    frappe.log_error(f"[controller.py] save_doc: {str(e)}")
```

**Backend - Whitelisted Method:**

```python
@frappe.whitelist()
def api_method(param1, param2):
    try:
        result = process_data(param1, param2)
        frappe.log_error(f"[api.py] api_method: Processed {result}")
        return {"success": True, "result": result}
    except Exception as e:
        frappe.log_error(f"[api.py] api_method: {str(e)}")
        return {"success": False, "message": str(e)}
```

## Migration Checklist

When updating existing code:

1. ✅ Replace `console.info()` → `console.log()`
2. ✅ Replace `console.warn()` → `console.log()`
3. ✅ Replace `console.error()` → `console.log()` (unless actual error handling)
4. ✅ Add filename to all console.log statements
5. ✅ Replace verbose logging with short results
6. ✅ Update `frappe.log_error()` calls to include filename
7. ✅ Ensure exception handling uses proper format
