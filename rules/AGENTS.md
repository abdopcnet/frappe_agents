# AI Agent Guidelines - Frappe Framework

## ⚠️ CRITICAL: LANGUAGE REQUIREMENT

**ALL RESPONSES MUST BE IN ENGLISH - NO EXCEPTIONS**

- ✅ **ALWAYS respond in English** - Even if the user writes in Arabic or any other language
- ✅ **NEVER respond in Arabic** - This is a strict requirement
- ✅ **This rule has highest priority** - It overrides any other consideration
- ✅ **User may write in Arabic** - But you MUST respond in English only

**This is the #1 rule. All other rules are secondary to this requirement.**

## ⚠️ CRITICAL: CODE COMMENTS MUST BE IN ENGLISH

**ALL CODE COMMENTS MUST BE IN ENGLISH - NO EXCEPTIONS**

- ✅ **ALWAYS write comments in English** - When writing or editing any code file (`.js`, `.py`, `.ts`, `.jsx`, `.tsx`, `.vue`, etc.), ALL code comments must be in English
- ✅ **NEVER use Arabic or other languages in comments** - Code comments (lines starting with `//`, `#`, `/* */`) must be in English
- ✅ **Applies to all comment types** - Single-line comments (`//`, `#`), multi-line comments (`/* */`, `""" """`), and inline comments
- ✅ **Includes documentation strings** - Docstrings, function descriptions, and code documentation must be in English
- ✅ **Replace non-English comments** - When editing files with non-English comments, translate them to English

**IMPORTANT DISTINCTION:**

- 🚨 **Code comments** (explanatory text for developers) → **MUST be in English**
- ✅ **User-facing text** (frappe.msgprint messages, HTML labels, dialog titles, button text) → **Keep original language** (typically Arabic for Arabic interfaces)
- ✅ **String literals** in `__()`, `frappe.msgprint()`, HTML templates → **Do NOT translate** (these are UI strings)

**Examples:**

```javascript
// ✅ CORRECT: Comment in English
// If custom table exists, skip native table updates
if (frm.fields_dict.custom_items_table) {
  return;
}

// ✅ CORRECT: User message stays in Arabic
frappe.msgprint(__("الرجاء اختيار الصنف أولاً")); // Comment can be English

// ❌ WRONG: Comment in Arabic
// إذا كان هناك جدول مخصص
```

**This is the #2 rule. All code comments must be in English when writing or editing files.**

## ⚠️ CRITICAL: RULES-BASED WORK ONLY

**ALL WORK MUST BE BASED ON RULES ONLY - NO EXCEPTIONS**

- 🚨 **ONLY work based on existing RULES** - Do not proceed if no rule exists
- 🚨 **MANDATORY: Alert before answering** - If no rule exists, you MUST show a PROMPT before answering
- 🚨 **DO NOT guess or assume** - Never create solutions without a rule
- 🚨 **PROMPT format required** - When no rule exists, show a clear PROMPT asking for rule development
- 🚨 **Wait for user guidance** - Do not proceed until user provides the new rule

**This is the #3 rule. Work only with existing rules. Alert before answering if no rule exists.**

## ⚠️ CRITICAL: CACHE CLEARING AFTER CODE CHANGES

**MANDATORY: Prompt user to run cache clearing commands after modifying JS or PY files**

- 🚨 **ALWAYS prompt after JS/PY changes** - After modifying any `.js`, `.py`, or Python/JavaScript files, you MUST prompt the user to run cache clearing commands
- 🚨 **Provide app-specific commands** - Generate commands based on the `app_folder_name` where changes were made
- 🚨 **Use exact command format** - Follow this exact pattern for each app:

**Command Template:**

```bash
cd ~/frappe-bench/apps/[app_folder_name]
bench --site all clear-cache &&
bench --site all clear-website-cache &&
find . -name "*.pyc" -print -delete && bench restart
bench build --app [app_folder_name] --force
```

**Examples:**

- For `frappe` app: `cd ~/frappe-bench/apps/frappe && bench --site all clear-cache && bench --site all clear-website-cache && find . -name "*.pyc" -print -delete && bench restart && bench build --app frappe --force`
- For `lms` app: `cd ~/frappe-bench/apps/lms && bench --site all clear-cache && bench --site all clear-website-cache && find . -name "*.pyc" -print -delete && bench restart && bench build --app lms --force`
- For `payments` app: `cd ~/frappe-bench/apps/payments && bench --site all clear-cache && bench --site all clear-website-cache && find . -name "*.pyc" -print -delete && bench restart && bench build --app payments --force`

**When to prompt:**

- After modifying any `.py` file (Python)
- After modifying any `.js` file (JavaScript)
- After modifying any `.ts` file (TypeScript)
- After modifying any `.vue` file (Vue components)
- After modifying any `.jsx` or `.tsx` file (React components)

**This is the #4 rule. Always prompt for cache clearing after code changes.**

## ⚠️ CRITICAL: JSON FILE MODIFICATIONS

**MANDATORY: Never modify JSON files without explicit user permission**

- 🚨 **NEVER modify JSON files directly** - Do not change `.json` files (DocType JSON, field JSON, etc.) without prompting the user first
- 🚨 **ALWAYS prompt before JSON changes** - Show a clear attention prompt asking for permission
- 🚨 **Explain what will change** - Clearly describe what JSON modifications you want to make and why
- 🚨 **Wait for explicit approval** - Do not proceed until user explicitly approves the JSON changes

**Prompt Format:**

```
⚠️ ATTENTION REQUIRED: JSON File Modification

I need to modify the following JSON file(s):
- [file path]
- [specific changes to be made]

This will affect: [describe impact]

Do you want me to proceed with these JSON modifications? (Yes/No)
```

**This is the #5 rule. Always prompt before JSON modifications.**

---

This directory contains specialized technical reference guides for AI agents working with Frappe Framework. Each file provides concise, practical guidance based on actual codebase analysis.

**This file (AGENTS.md) should be updated dynamically during responses when new patterns, apps, or configurations are discovered.**

## Installed Applications

**Core Framework:**

- **frappe** - Frappe Framework (core)
- **erpnext** - ERPNext (ERP application)

**Domain Applications:**

- **hrms** - Frappe HR (Human Resources)
- **lms** - Frappe LMS (Learning Management System, app_name: `frappe_lms`)
- **payments** - Payments (Payment gateway integrations)

**Custom Applications:**

- **hrms_custom** - HRMS Custom extensions
- **export_custom_fields** - Custom fields export utility

## File Structure

### Core Framework References

- **frappe.md** - Framework structure, file locations, hooks system
- **erpnext.md** - ERPNext modules, business logic patterns

### Development References

- **doctype_commands.md** - DocType creation, structure, lifecycle
- **custom_field.md** - Custom field creation and management
- **client_script.md** - Client-side JavaScript patterns
- **server_script.md** - Server-side Python patterns, controllers
- **script_report.md** - Script Report development

### System Operations

- **database_commands.md** - Database queries, Frappe DB API
- **site_commands.md** - Bench commands, site management
- **frappe_site.md** - Web pages, Jinja2 templates, print formats
- **frappe_site_ssl.md** - SSL certificate setup with Let's Encrypt (Certbot)

### Debugging & Troubleshooting

- **debugging.md** - Frontend and backend logging patterns
- **network_debug.md** - Network/server troubleshooting (run first)
- **bench_debug.md** - Frappe bench-specific debugging (run second) - includes migration errors like module **file** TypeError

## Usage Guidelines

### For AI Agents

1. **Read AGENTS.md first** - Understand installed apps and file structure
2. **Check for rules BEFORE answering** - Search all relevant .md files for applicable rules
3. **Work ONLY based on rules** - Never proceed without a rule. If no rule exists, show PROMPT first.
4. **Use specialized files** - Each file covers one specific topic
5. **Reference actual code** - All patterns are based on real codebase analysis
6. **Verify before use** - Check actual field names, file paths from code
7. **Update AGENTS.md when needed** - If new patterns or apps are discovered during responses, update this file
8. **MANDATORY: Alert before answering** - If no rule exists, show PROMPT format before any answer

### Communication Rules

- 🚨 **CRITICAL: English responses ONLY** - ALL responses MUST be in English. This is the highest priority rule. User may write in Arabic, but you MUST respond in English only. NO EXCEPTIONS.
- ✅ **Short and direct** - Brief, practical answers
- ✅ **Numbered points** - Use numbered lists when listing items
- ✅ **Code examples** - Show actual code patterns, not explanations
- ✅ **Respect all rules** - Follow all rules defined in this file and specialized files
- ✅ **Ask for new rules** - If no rule exists for a situation, ask the user to develop new rules instead of guessing

### Code Quality Standards

- ✅ **Read actual files** - Always inspect existing code before modifying
- ✅ **Verify field names** - Check database schema or DocType JSON
- ✅ **Follow patterns** - Match existing code patterns in the app
- ✅ **Never modify core** - Don't edit frappe/erpnext/hrms core files in `/home/frappe/frappe-bench/apps/frappe`, `/home/frappe/frappe-bench/apps/erpnext`, or `/home/frappe/frappe-bench/apps/hrms`
- ✅ **English comments only** - ALL code comments must be in English (see CRITICAL: CODE COMMENTS MUST BE IN ENGLISH rule)

## Project Context

**Environment:**

- Bench: `/home/frappe/frappe-bench-15/`
- Apps: `/home/frappe/frappe-bench-15/apps/`
- Sites: `/home/frappe/frappe-bench-15/sites/`
- Default Site: `tbook.app`
- Server IP: `95.111.234.1`
- DNS Configuration: `tbook.app` A record → `95.111.234.1`

**Configuration:**

- Developer Mode: Enabled
- Socket.IO Port: 9000
- Web Server Port: 8000
- Redis: Cache (13000), Queue (11000), SocketIO (13000)

## Quick Navigation

| Task               | File                 |
| ------------------ | -------------------- |
| Find file location | frappe.md            |
| Create DocType     | doctype_commands.md  |
| Add custom field   | custom_field.md      |
| Form validation    | client_script.md     |
| API method         | server_script.md     |
| Database query     | database_commands.md |
| Bench command      | site_commands.md     |
| Print format       | frappe_site.md       |
| SSL certificates   | frappe_site_ssl.md   |
| ERPNext logic      | erpnext.md           |
| Network issues     | network_debug.md     |
| Bench issues       | bench_debug.md       |

## Response Format

**MANDATORY:** Every response must end with Applied Rules section in this exact format:

```
Applied Rules:
📢 COMMUNICATION RULES (AGENTS.md)
🏛️ FRAMEWORK LOCATIONS (frappe.md)
📁 FRAPPE FRAMEWORK STRUCTURE (frappe.md)
📦 ERPNEXT APPLICATION STRUCTURE (erpnext.md)
📌 FIELD NAMING & DATA INTEGRITY (custom_field.md)
🧭 CODE ANALYSIS PRINCIPLES (server_script.md)
🏛️ CORE SYSTEM APPLICATIONS REFERENCE (frappe.md)
🔒 DATABASE MODIFICATION PROTOCOL (database_commands.md)
🧭 DEVELOPMENT WORKFLOW (doctype_commands.md)
🎯 FRAPPE DEVELOPMENT PATTERNS (server_script.md)
🎛️ CODE QUALITY STANDARDS (AGENTS.md)
✅ COMMUNICATION & VERIFICATION (AGENTS.md)
```

**Format Rules:**

- Use emoji + rule name + (filename.md)
- List only rules that were actually applied
- One rule per line
- Always include at least: `📢 COMMUNICATION RULES (AGENTS.md)`

## Handling Unknown Situations

**MANDATORY PROTOCOL: When No Rule Exists**

**BEFORE answering any question, check if a rule exists:**

1. **Check all relevant .md files** - Search for applicable rules
2. **If NO rule exists:**
   - 🚨 **STOP immediately** - Do NOT proceed with the answer
   - 🚨 **Show PROMPT first** - Display the PROMPT format below BEFORE any answer
   - 🚨 **Wait for user** - Do not continue until user provides the new rule
   - 🚨 **Then proceed** - Only after rule is provided, proceed with the answer

**MANDATORY PROMPT Format (show this BEFORE answering):**

```
⚠️ MISSING RULE ALERT

I don't have a specific rule for this situation: [describe situation]

Before I can proceed, I need you to help me develop a new rule.

Please provide:
1. What should be the approach for this situation?
2. Are there any constraints or limitations?
3. Which file should this rule be added to? (e.g., server_script.md, custom_field.md, etc.)
4. What should the rule be named? (e.g., "CUSTOM FIELD VALIDATION", "API ERROR HANDLING")

Once you provide the rule, I will:
- Update the relevant .md file with the new rule
- Update AGENTS.md if needed
- Apply the new rule in my response
```

**After user provides guidance:**

- Update the relevant .md file with the new rule
- Update AGENTS.md "Applied Rules Reference" section
- Apply the new rule in the response
- Show the new rule in "Applied Rules" section

## Applied Rules Reference

**Available Rules (with emoji and file):**

- 📢 **COMMUNICATION RULES** (AGENTS.md) - English only, short and direct
- 🏛️ **FRAMEWORK LOCATIONS** (frappe.md) - Know file paths and structure
- 📁 **FRAPPE FRAMEWORK STRUCTURE** (frappe.md) - Understand core modules
- 📦 **ERPNEXT APPLICATION STRUCTURE** (erpnext.md) - Understand ERPNext modules
- 📌 **FIELD NAMING & DATA INTEGRITY** (custom_field.md) - Use correct field names
- 🧭 **CODE ANALYSIS PRINCIPLES** (server_script.md) - Read actual code before modifying
- 🏛️ **CORE SYSTEM APPLICATIONS REFERENCE** (frappe.md) - Reference core apps for patterns
- 🔒 **DATABASE MODIFICATION PROTOCOL** (database_commands.md) - Safe database operations
- 🧭 **DEVELOPMENT WORKFLOW** (doctype_commands.md) - Follow DocType creation patterns
- 🎯 **FRAPPE DEVELOPMENT PATTERNS** (server_script.md) - Follow controller patterns
- 🎛️ **CODE QUALITY STANDARDS** (AGENTS.md) - Code quality requirements
- ✅ **COMMUNICATION & VERIFICATION** (AGENTS.md) - Response format requirements
- 📝 **CLIENT SCRIPT PATTERNS** (client_script.md) - JavaScript form patterns, set_query for link filtering
- 📊 **REPORT DEVELOPMENT** (script_report.md) - Script Report patterns
- 🌐 **WEB PAGES & TEMPLATES** (frappe_site.md) - Jinja2 and web page patterns
- 🔒 **SSL CERTIFICATE SETUP** (frappe_site_ssl.md) - Let's Encrypt SSL setup with Certbot
- ⚙️ **SITE MANAGEMENT** (site_commands.md) - Bench commands
- 🔍 **LINK FIELD QUERY FILTERING** (server_script.md) - Query functions for filtering by
- 🌐 **NETWORK TROUBLESHOOTING** (network_debug.md) - System-level network and server debugging
- 🔧 **BENCH TROUBLESHOOTING** (bench_debug.md) - Frappe bench-specific debugging and diagnostics, including migration errors (module **file** TypeError)
- 🔄 **FRAPPE DATA LOGIC** (AGENTS.md) - Data flow between frontend and backend
- 🐛 **UNIFIED LOGGING STANDARDS** (debugging.md) - Frontend and backend logging patterns
- 🔄 **CACHE CLEARING PROTOCOL** (AGENTS.md) - Mandatory cache clearing after JS/PY file modifications
- 📋 **JSON MODIFICATION PROTOCOL** (AGENTS.md) - Always prompt before modifying JSON files
- 💬 **CODE COMMENTS LANGUAGE** (AGENTS.md) - All code comments must be in English only

## Frappe Data Logic - What Frappe Uses

**Data Flow Layers:**

| Layer            | Data Format                       | Processing                       |
| ---------------- | --------------------------------- | -------------------------------- |
| Frontend JS      | JavaScript objects `{key: value}` | Automatically serialized to JSON |
| HTTP Request     | JSON string in request body       | Sent via POST/PUT                |
| Backend Python   | `json.loads()` converts to dict   | Auto-parsed by Frappe in app.py  |
| Whitelist Method | Receives Python dict or str       | Check type with `isinstance()`   |

**Key Point:** Frappe automatically parses JSON to dict, but only sends what the frontend loaded, which is why you must re-fetch documents from the database if you need all fields!

## Dynamic Updates

**When to Update AGENTS.md:**

- New app discovered during analysis
- New pattern found in codebase
- Configuration changes detected
- New DocType or module discovered
- Important hook or override pattern found
- New rule developed based on user guidance

**How to Update:**

- Add new information to relevant sections
- Keep existing structure
- Maintain concise format
- Update "Installed Applications" if new app found
- Update "Quick Navigation" if new common task identified
- Add new rules to "Applied Rules Reference" section

---

## Reminder: Critical Requirements

**BEFORE EVERY RESPONSE: Check these five things:**

1. **Language Check: Am I responding in English?**

   - If user writes in Arabic → Respond in English
   - If user writes in English → Respond in English
   - If user writes in any language → Respond in English
   - **This is non-negotiable. English only. Always.**

2. **Code Comments Check: Are all code comments in English?**

   - When writing code → Use English for ALL code comments (lines starting with `//`, `#`, etc.)
   - When editing code → Translate any non-English code comments to English
   - Includes docstrings, inline comments, and block comments
   - **User-facing messages** (frappe.msgprint, HTML labels) → Keep original language, do NOT translate
   - **All code comments must be in English. No exceptions.**

3. **Rules Check: Do I have a rule for this?**

   - Search all relevant .md files for applicable rules
   - If NO rule exists → Show PROMPT format BEFORE answering
   - If rule exists → Proceed with answer based on the rule
   - **Work ONLY based on rules. No guessing. No assumptions.**

4. **Code Change Check: Did I modify JS or PY files?**

   - If YES → Prompt user to run cache clearing commands
   - Provide app-specific commands based on `app_folder_name`
   - **Always prompt after code modifications.**

5. **JSON Change Check: Am I about to modify JSON files?**
   - If YES → Show attention prompt and wait for approval
   - Explain what will change and why
   - **Never modify JSON without explicit permission.**

---

_Each file is a technical reference based on actual codebase analysis. Keep responses concise and practical. Always end with Applied Rules section. **ALL RESPONSES MUST BE IN ENGLISH - NO EXCEPTIONS. ALL CODE COMMENTS MUST BE IN ENGLISH - NO EXCEPTIONS. ALL WORK MUST BE BASED ON RULES ONLY - NO EXCEPTIONS. If no rule exists, show PROMPT format BEFORE answering.**_
