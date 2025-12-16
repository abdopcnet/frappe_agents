# Client Script Reference

## File Locations

**DocType Client Scripts:**

- `/apps/[app]/[app]/[module]/doctype/[doctype]/[doctype].js`

**Client Script DocType:**

- Desk → Customize → Customize Form → Client Script

## Basic Structure

**Pattern:**

```javascript
frappe.ui.form.on("DocType Name", {
  onload: function (frm) {},
  refresh: function (frm) {},
  field_name: function (frm) {},
});
```

**Real Example (LMS Course):**

```javascript
frappe.ui.form.on("LMS Course", {
  onload: function (frm) {
    frm.set_query("chapter", "chapters", function () {
      return {
        filters: {
          course: frm.doc.name,
        },
      };
    });
  },
  refresh: (frm) => {
    frm.add_web_link(`/lms/courses/${frm.doc.name}`, "See on Website");

    if (!frm.doc.currency) {
      frappe.db
        .get_single_value("LMS Settings", "default_currency")
        .then((value) => {
          frm.set_value("currency", value);
        });
    }
  },
});
```

**Real Example (Razorpay Settings):**

```javascript
frappe.ui.form.on("Razorpay Settings", {
  refresh: function (frm) {
    frm.add_custom_button(__("Clear"), function () {
      frm.call({
        doc: frm.doc,
        method: "clear",
        callback: function (r) {
          frm.refresh();
        },
      });
    });
  },
});
```

## Form Events

**Event Order:**

- `onload` - First load only
- `refresh` - Every refresh (load, save, etc.)
- `before_save` - Before save
- `validate` - Before save validation
- `after_save` - After save
- `before_submit` - Before submit
- `on_submit` - After submit

## Field Operations

**Get/Set Values:**

```javascript
// Get value
let value = frm.doc.field_name;

// Set value
frm.set_value("field_name", "value");

// Set multiple
frm.set_value({
  field1: "value1",
  field2: "value2",
});

// Get from database
frappe.db.get_single_value("DocType", "field").then((value) => {
  frm.set_value("field", value);
});
```

**Field Properties:**

```javascript
// Hide/Show
frm.set_df_property("field_name", "hidden", 1);
frm.toggle_display("field_name", false);

// Enable/Disable
frm.toggle_enable("field_name", true);

// Make mandatory
frm.set_df_property("field_name", "reqd", 1);

// Read-only
frm.set_df_property("field_name", "read_only", 1);
```

## Child Table Operations

**Add/Remove Rows:**

```javascript
// Add row
let row = frm.add_child("items", {
  item_code: "ITEM-001",
  qty: 1,
});
frm.refresh_field("items");

// Remove row
frm.doc.items.pop();
frm.refresh_field("items");

// Clear table
frm.clear_table("items");
frm.refresh_field("items");

// Set value in row
frappe.model.set_value("Child DocType", row.name, "field", value);
```

**Loop Through Rows:**

```javascript
(frm.doc.items || []).forEach(function (item) {
  item.amount = item.qty * item.rate;
});
frm.refresh_field("items");
```

## Field Filtering

**Set Query:**

```javascript
frm.set_query("field_name", function () {
  return {
    filters: {
      status: "Active",
    },
  };
});

// Child table filter
frm.set_query("item_code", "items", function (doc, cdt, cdn) {
  return {
    filters: {
      item_group: "Products",
    },
  };
});
```

**Real Example:**

```javascript
frm.set_query("chapter", "chapters", function () {
  return {
    filters: {
      course: frm.doc.name,
    },
  };
});
```

**Custom Query Function (for child table filtering):**

When you need to filter by child table fields (e.g., User by role from Has Role), use a custom query function:

```javascript
frm.set_query("student", function () {
  return {
    query: "app.module.doctype.doctype_name.get_students",
  };
});
```

**Pattern:** Use `query` property pointing to a whitelisted server method that returns `[["name", "label"], ...]` format.

## API Calls

**Basic Pattern:**

```javascript
frm.call({
  doc: frm.doc,
  method: "method_name",
  callback: function (r) {
    if (r.message) {
      frm.set_value("field", r.message);
    }
    frm.refresh();
  },
});
```

**Real Example:**

```javascript
frm.add_custom_button(__("Clear"), function () {
  frm.call({
    doc: frm.doc,
    method: "clear",
    callback: function (r) {
      frm.refresh();
    },
  });
});
```

**With frappe.call:**

```javascript
frappe.call({
  method: "app.module.method",
  args: { param: value },
  callback: function (r) {
    if (r.message) {
      // Handle response
    }
  },
});
```

## Custom Buttons

**Pattern:**

```javascript
frm.add_custom_button(
  "Label",
  function () {
    // Action
  },
  "Group Name"
);

// Add web link
frm.add_web_link(`/lms/courses/${frm.doc.name}`, "See on Website");
```

## Utilities

**Type Conversion:**

```javascript
flt(value); // Float conversion
cint(value); // Integer conversion
```

**Date Operations:**

```javascript
frappe.datetime.get_today(); // Today's date
frappe.datetime.add_days(date, days); // Add days
```

**Messages:**

```javascript
frappe.msgprint("Message"); // Show message
frappe.throw("Error"); // Throw error
frappe.show_alert("Alert"); // Show alert
```

**Translation:**

```javascript
__("Translatable String"); // Translate string
```
