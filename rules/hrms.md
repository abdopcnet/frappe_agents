# HRMS (Human Resource Management System) Development Guide

## Overview

HRMS is a comprehensive Human Resource and Payroll Management application built on Frappe Framework. This guide covers HRMS-specific modules, HR processes, and payroll operations.

For general Frappe guidelines, see `frappe.md`. For general AI guidelines, see `AGENTS.md`.

---

## 📋 Table of Contents

1. [HRMS Architecture](#hrms-architecture)
2. [Core Modules](#core-modules)
3. [Employee Management](#employee-management)
4. [Attendance & Leave](#attendance--leave)
5. [Payroll System](#payroll-system)
6. [Recruitment](#recruitment)
7. [Performance & Training](#performance--training)
8. [Common Patterns](#common-patterns)

---

## HRMS Architecture

### Application Structure

```
/home/frappe/frappe-bench/apps/hrms/hrms/
├── hr/                     # HR Management (110 DocTypes)
│   ├── doctype/
│   ├── page/
│   └── report/
├── payroll/                # Payroll Management (38 DocTypes)
│   ├── doctype/
│   └── report/
├── regional/               # Country-specific features
│   └── india/
├── controllers/            # Base controllers
├── overrides/              # DocType overrides
└── www/                    # Web routes
```

### Modules

1. **HR Module (110 DocTypes)**
   - Employee lifecycle management
   - Attendance tracking
   - Leave management
   - Performance appraisal
   - Training & development
   - Shift management
   - Expense claims

2. **Payroll Module (38 DocTypes)**
   - Salary structure
   - Salary slip generation
   - Tax calculations
   - Payroll entry
   - Additional salary
   - Retention bonus

---

## Core Modules

### HR Module

**Location:** `/home/frappe/frappe-bench/apps/hrms/hrms/hr/`

**Key DocTypes:**
- `Employee` - Employee master data
- `Attendance` - Daily attendance records
- `Leave Application` - Leave requests
- `Leave Allocation` - Leave balance allocation
- `Leave Type` - Leave categories (Casual, Sick, etc.)
- `Employee Checkin` - Biometric/manual check-in
- `Shift Type` - Shift definitions
- `Shift Assignment` - Employee shift assignments
- `Expense Claim` - Employee expense reimbursements
- `Appraisal` - Performance appraisals
- `Training Event` - Training programs

### Payroll Module

**Location:** `/home/frappe/frappe-bench/apps/hrms/hrms/payroll/`

**Key DocTypes:**
- `Salary Structure` - Salary template
- `Salary Structure Assignment` - Assign structure to employee
- `Salary Slip` - Monthly salary document
- `Payroll Entry` - Bulk salary processing
- `Salary Component` - Earnings/deductions components
- `Additional Salary` - One-time payments
- `Retention Bonus` - Retention incentives
- `Employee Tax Exemption Declaration` - Tax declarations
- `Income Tax Slab` - Tax brackets

---

## Employee Management

### Employee DocType

**Key Fields:**
```python
{
    "employee": "Auto-generated ID (HR-EMP-00001)",
    "first_name": "First name",
    "last_name": "Last name",
    "employee_name": "Full name (auto-generated)",
    "user_id": "Link to User (for portal access)",
    "company": "Link to Company",
    "date_of_joining": "Date",
    "date_of_birth": "Date",
    "gender": "Male/Female/Other",
    "department": "Link to Department",
    "designation": "Link to Designation",
    "employment_type": "Permanent/Contract/Temporary",
    "status": "Active/Left/Suspended",
    "relieving_date": "Date (for exit)",
    "holiday_list": "Link to Holiday List",
    "default_shift": "Link to Shift Type"
}
```

**Important Methods:**
```python
# Validate employee
from hrms.hr.utils import validate_active_employee
validate_active_employee(employee_id)

# Get employee by user
employee = frappe.get_value("Employee", {"user_id": "user@example.com"}, "name")
```

**File Location:** `/home/frappe/frappe-bench/apps/hrms/hrms/hr/doctype/employee/`

---

## Attendance & Leave

### Attendance System

**Attendance DocType:**
```python
{
    "employee": "Link to Employee",
    "attendance_date": "Date",
    "status": "Present/Absent/Half Day/On Leave/Work From Home",
    "shift": "Link to Shift Type",
    "in_time": "DateTime",
    "out_time": "DateTime",
    "working_hours": "Float (auto-calculated)"
}
```

**Marking Attendance:**
```python
# Create attendance
attendance = frappe.get_doc({
    "doctype": "Attendance",
    "employee": employee_id,
    "attendance_date": today(),
    "status": "Present",
    "company": company
})
attendance.insert()
attendance.submit()
```

**Employee Checkin:**
- Biometric integration
- GPS-based check-in
- Mobile app support
- Auto-attendance creation based on checkin/checkout

### Leave Management

**Leave Type:**
- Casual Leave
- Sick Leave
- Earned Leave
- Leave Without Pay (LWP)
- Compensatory Off

**Leave Application:**
```python
{
    "employee": "Link to Employee",
    "leave_type": "Link to Leave Type",
    "from_date": "Date",
    "to_date": "Date",
    "total_leave_days": "Float",
    "half_day": "Check",
    "leave_approver": "Link to User",
    "status": "Open/Approved/Rejected/Cancelled"
}
```

**Leave Allocation:**
```python
# Allocate leave balance
allocation = frappe.get_doc({
    "doctype": "Leave Allocation",
    "employee": employee_id,
    "leave_type": "Casual Leave",
    "from_date": "2025-01-01",
    "to_date": "2025-12-31",
    "new_leaves_allocated": 12
})
allocation.insert()
allocation.submit()
```

**Leave Encashment:**
- Convert unused leaves to cash
- Linked to salary slip
- Requires salary structure

---

## Payroll System

### Salary Structure

**Structure Definition:**
```python
{
    "name": "Monthly Salary - Sales Team",
    "payroll_frequency": "Monthly/Fortnightly/Weekly/Daily",
    "company": "Link to Company",
    "currency": "INR/USD/etc",
    "earnings": "Table (Salary Detail)",
    "deductions": "Table (Salary Detail)",
    "hour_rate": "Float (for hourly workers)"
}
```

**Salary Components:**
```python
# Earnings
- Basic Salary (BS)
- House Rent Allowance (HRA)
- Dearness Allowance (DA)
- Conveyance Allowance
- Medical Allowance

# Deductions
- Professional Tax
- Income Tax (TDS)
- Provident Fund (PF)
- Employee State Insurance (ESI)
```

**Formula Variables:**
```python
# From Salary Structure Assignment
base = Base amount
variable = Variable amount

# From Employee
employment_type = Employment Type
branch = Branch name

# From Salary Slip
payment_days = Working days paid
leave_without_pay = LWP days

# Component abbreviations
BS = Basic Salary
HRA = House Rent Allowance

# Additional
gross_pay = Total earnings
annual_taxable_earning = Taxable income
```

**Example Formulas:**
```python
# HRA (40% of Basic)
BS * 0.40

# Conditional component
base if employment_type == "Permanent" else variable

# Taxable component
BS + HRA + DA if gross_pay > 250000 else 0
```

### Salary Structure Assignment

**Assign to Employee:**
```python
from hrms.payroll.doctype.salary_structure.salary_structure import (
    create_salary_structure_assignment
)

assignment = create_salary_structure_assignment(
    employee=employee_id,
    salary_structure="Monthly Salary",
    company="My Company",
    currency="INR",
    from_date="2025-01-01",
    base=50000,  # Base salary
    variable=10000  # Variable pay
)
```

### Salary Slip

**Monthly Salary Document:**
```python
{
    "employee": "Link to Employee",
    "salary_structure": "Link to Salary Structure",
    "start_date": "2025-01-01",
    "end_date": "2025-01-31",
    "payment_days": "Float (auto-calculated)",
    "leave_without_pay": "Float",
    "earnings": "Table (Salary Detail)",
    "deductions": "Table (Salary Detail)",
    "gross_pay": "Float",
    "total_deduction": "Float",
    "net_pay": "Float",
    "rounded_total": "Float"
}
```

**Generate Salary Slip:**
```python
from hrms.payroll.doctype.salary_structure.salary_structure import make_salary_slip

# Create draft slip
salary_slip = make_salary_slip(
    source_name="Salary Structure Name",
    employee=employee_id,
    posting_date="2025-01-31"
)
salary_slip.insert()
salary_slip.submit()
```

**Payment Days Calculation:**
- Total working days = Days in month - Holidays - Sundays
- Payment days = Working days - Leave without pay
- Half-day affects payment days

### Payroll Entry

**Bulk Salary Processing:**
```python
{
    "company": "Link to Company",
    "start_date": "2025-01-01",
    "end_date": "2025-01-31",
    "payroll_frequency": "Monthly",
    "payment_account": "Link to Account",
    "employees": "Table (Payroll Employee Detail)",
    "currency": "INR"
}
```

**Process:**
1. Create Payroll Entry
2. Get Employees (filters by department, branch, etc.)
3. Create Salary Slips (bulk generation)
4. Submit Salary Slips (after review)
5. Make Bank Entry (payment journal)

**Commands:**
```python
# Create payroll entry
payroll = frappe.get_doc("Payroll Entry", payroll_name)

# Get employees
payroll.fill_employee_details()

# Create salary slips
payroll.create_salary_slips()

# Submit slips
payroll.submit_salary_slips()

# Create payment entry
payroll.make_payment_entry()
```

### Additional Salary

**One-Time Payments:**
```python
{
    "employee": "Link to Employee",
    "salary_component": "Bonus/Arrear/Other",
    "amount": "Float",
    "payroll_date": "Date",
    "overwrite_salary_structure_amount": "Check",
    "ref_doctype": "Leave Encashment/Other",
    "ref_docname": "LE-00001"
}
```

**Use Cases:**
- Performance bonus
- Leave encashment
- Salary arrears
- Festival allowance
- Overtime payment

---

## Recruitment

### Job Opening

**Job Posting:**
```python
{
    "job_title": "Software Engineer",
    "designation": "Link to Designation",
    "department": "Link to Department",
    "status": "Open/Closed",
    "description": "Text",
    "publish": "Check (for website)",
    "route": "jobs/software-engineer"
}
```

### Job Applicant

**Application Tracking:**
```python
{
    "applicant_name": "Name",
    "email_id": "Email",
    "job_title": "Link to Job Opening",
    "status": "Open/Replied/Hold/Accepted/Rejected",
    "resume_attachment": "Attach"
}
```

### Job Offer

**Offer Letter:**
```python
{
    "job_applicant": "Link to Job Applicant",
    "designation": "Link to Designation",
    "offer_date": "Date",
    "status": "Awaiting Response/Accepted/Rejected",
    "offer_terms": "Table"
}
```

---

## Performance & Training

### Appraisal

**Performance Review:**
```python
{
    "employee": "Link to Employee",
    "start_date": "Date",
    "end_date": "Date",
    "kra_template": "Link to Appraisal Template",
    "goals": "Table (Appraisal Goal)",
    "total_score": "Float",
    "status": "Draft/Submitted/Completed"
}
```

### Training Event

**Training Programs:**
```python
{
    "event_name": "Python Training",
    "course": "Link to Training Program",
    "start_time": "DateTime",
    "end_time": "DateTime",
    "employees": "Table",
    "trainer_name": "Name",
    "location": "Text"
}
```

---

## Common Patterns

### 1. Validate Active Employee

```python
from hrms.hr.utils import validate_active_employee

validate_active_employee(employee_id)
# Throws error if employee is not active
```

### 2. Get Salary Structure

```python
from hrms.payroll.doctype.salary_structure.salary_structure import (
    get_assigned_salary_structure
)

salary_structure = get_assigned_salary_structure(employee_id, date)
```

### 3. Get Leave Balance

```python
from hrms.hr.doctype.leave_application.leave_application import (
    get_leave_balance_on
)

balance = get_leave_balance_on(
    employee=employee_id,
    leave_type="Casual Leave",
    date=today()
)
```

### 4. Calculate Tax

```python
from hrms.payroll.doctype.salary_slip.salary_slip import (
    calculate_tax_by_tax_slab
)

tax_amount = calculate_tax_by_tax_slab(
    annual_taxable_earning,
    tax_slab
)
```

### 5. Check Leave Approver

```python
from hrms.hr.utils import get_leave_approver

leave_approver = get_leave_approver(employee_id)
```

---

## Regional Features

### India-Specific

**Location:** `/home/frappe/frappe-bench/apps/hrms/hrms/regional/india/`

**Features:**
- Provident Fund (PF)
- Employee State Insurance (ESI)
- Professional Tax
- Gratuity calculations
- HRA exemption
- Income Tax (with Section 80C, 80D, etc.)

**Tax Slabs:**
- Old tax regime
- New tax regime
- Marginal relief

---

## Important Utilities

### File: `/home/frappe/frappe-bench/apps/hrms/hrms/hr/utils.py`

**Key Functions:**
- `validate_active_employee(employee)` - Check employee status
- `get_leave_approver(employee)` - Get leave approving authority
- `set_employee_name(doc)` - Set employee name from ID
- `get_holiday_list_for_employee(employee)` - Get applicable holidays

### File: `/home/frappe/frappe-bench/apps/hrms/hrms/payroll/utils.py`

**Key Functions:**
- `get_salary_component_data(component)` - Get component details
- `validate_tax_declaration(proofs)` - Validate tax declarations

---

## Common Issues & Solutions

### 1. Salary Slip Not Generating

**Error:** Missing Salary Structure Assignment

**Solution:** Assign salary structure to employee from the required date

### 2. Leave Application Rejected

**Error:** Insufficient leave balance

**Solution:** Check leave allocation for that leave type and period

### 3. Payroll Entry Not Creating Slips

**Error:** No employees found

**Solution:** Verify filters (company, branch, department, designation)

### 4. Tax Calculation Incorrect

**Error:** Wrong tax slab or exemptions

**Solution:** 
- Set Income Tax Slab in Salary Structure Assignment
- Verify tax declarations and proofs

---

## Best Practices

### 1. Employee Onboarding

```python
# 1. Create Employee
employee = frappe.get_doc({
    "doctype": "Employee",
    "first_name": "John",
    "last_name": "Doe",
    "company": "My Company",
    "date_of_joining": today()
})
employee.insert()

# 2. Assign Salary Structure
create_salary_structure_assignment(...)

# 3. Allocate Leaves
# Create leave allocations for each leave type

# 4. Set Shift (if applicable)
```

### 2. Monthly Payroll Processing

```bash
1. Mark attendance for all employees
2. Process leave applications
3. Create Payroll Entry
4. Get Employees
5. Review employee list
6. Create Salary Slips
7. Review salary slips for errors
8. Submit salary slips
9. Make Bank Entry for payment
10. Generate payslips (PDF)
```

### 3. Year-End Processing

```bash
1. Leave carry-forward/encashment
2. Annual tax calculations
3. Form 16 generation (India)
4. Bonus/gratuity calculations
5. Archive old records
```

---

## Quick Commands

```bash
# Install HRMS
bench get-app hrms
bench --site [sitename] install-app hrms

# Run migrations
bench --site [sitename] migrate

# Process payroll (via bench console)
bench --site [sitename] console
>>> from hrms.payroll.doctype.payroll_entry.payroll_entry import get_start_end_dates
>>> get_start_end_dates("Monthly", "01-2025")

# Clear cache
bench clear-cache
```

---

**Documentation:** https://docs.frappe.io/hrms  
**Repository:** https://github.com/frappe/hrms
