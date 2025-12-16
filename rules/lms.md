# Frappe LMS Development Guide

## Overview

Frappe LMS (Learning Management System) is an easy-to-use, open-source learning platform built on Frappe Framework. This guide covers LMS-specific architecture, modules, and development patterns.

For general Frappe guidelines, see `frappe.md`. For general AI guidelines, see `AGENTS.md`.

---

## 📋 Table of Contents

1. [LMS Architecture](#lms-architecture)
2. [Core Modules](#core-modules)
3. [Course Management](#course-management)
4. [Batch & Live Classes](#batch--live-classes)
5. [Assessment System](#assessment-system)
6. [Certification](#certification)
7. [User Roles](#user-roles)
8. [Common Patterns](#common-patterns)
9. [Frontend (Vue.js)](#frontend-vuejs)

---

## LMS Architecture

### Application Structure

```
/home/frappe/frappe-bench/apps/lms/lms/
├── lms/                    # Main LMS module
│   ├── doctype/            # 74+ DocTypes for LMS features
│   ├── page/               # Desk pages
│   ├── report/             # Script reports
│   └── workspace/          # LMS workspace configuration
├── job/                    # Job opportunities module (3 DocTypes)
│   └── doctype/
├── www/                    # Web routes
├── templates/              # Jinja2 templates
├── public/                 # Static assets
└── frontend/               # Vue.js SPA (Single Page Application)
    ├── src/
    │   ├── components/     # Vue components
    │   ├── pages/          # Vue pages
    │   └── router/         # Vue Router
    └── package.json
```

### Key Concepts

**1. Course Hierarchy:**
```
LMS Course
  ├── Chapter (Course Chapter)
  │   └── Lessons (Course Lesson)
  │       ├── Content (Text, Video, Quiz)
  │       ├── Exercises
  │       └── Assignments
```

**2. Batch vs Self-Learning:**
- **Self-Learning**: Students enroll and progress at own pace
- **Batch-Based**: Structured learning with schedule, instructors, and evaluators

---

## Core Modules

### 1. LMS Module (74 DocTypes)

**Main DocTypes:**
- `LMS Course` - Course definition
- `Course Chapter` - Chapter organization
- `Course Lesson` - Individual lessons
- `LMS Batch` - Batch/cohort management
- `LMS Enrollment` - Course enrollment tracking
- `LMS Assignment` - Assignment creation
- `LMS Quiz` - Quiz creation
- `LMS Certificate` - Certificate generation

**Supporting DocTypes:**
- `LMS Settings` - Global LMS configuration
- `LMS Category` - Course categories
- `LMS Sidebar Item` - Custom sidebar navigation
- `LMS Live Class` - Zoom integration for live classes

### 2. Job Module (3 DocTypes)

- `Job Opportunity` - Job postings
- `LMS Job Application` - Job application submissions

---

## Course Management

### LMS Course DocType

**Key Fields:**
```python
{
    "title": "Course title",
    "short_introduction": "Brief description",
    "description": "Detailed description (Rich Text)",
    "video_link": "YouTube video embed ID",
    "image": "Preview image",
    "instructors": "Table MultiSelect (Course Instructor)",
    "chapters": "Table (Chapter Reference)",
    "published": "Check (0/1)",
    "paid_course": "Check - requires Payments app",
    "course_price": "Currency",
    "enable_certification": "Check",
    "disable_self_learning": "Check - batch-only mode"
}
```

**Important Methods:**
- `validate_instructors()` - Auto-adds owner as instructor if empty
- `validate_payments_app()` - Checks if Payments app installed for paid courses
- `validate_certification()` - Validates certificate settings

**File Location:** `/home/frappe/frappe-bench/apps/lms/lms/lms/doctype/lms_course/`

### Course Chapter

**Structure:**
- Organizes lessons into logical groups
- `title`, `description`, `idx` (order)
- Child table: `lessons` (Lesson Reference)

### Course Lesson

**Key Fields:**
```python
{
    "title": "Lesson title",
    "body": "Markdown content",
    "content": "Rendered HTML",
    "youtube": "YouTube video ID",
    "quiz_id": "Link to LMS Quiz",
    "include_in_preview": "Check - visible before enrollment",
    "instructor_notes": "Markdown - visible only to instructors"
}
```

**Content Types:**
- Text/Markdown content
- Video (YouTube/Vimeo)
- Quiz
- Assignment
- Exercise (coding exercises)

---

## Batch & Live Classes

### LMS Batch

**Purpose:** Group students for structured learning with schedule

**Key Fields:**
```python
{
    "title": "Batch name",
    "courses": "Table (Batch Course)",
    "start_date": "Date",
    "end_date": "Date",
    "seat_count": "Int - max students",
    "instructors": "Table MultiSelect",
    "students": "Table (LMS Batch Enrollment)",
    "timetable": "Table - lesson schedule",
    "assessment": "Table - assessment schedule"
}
```

**Features:**
- Enrollment management
- Timetable/schedule
- Live class integration (Zoom)
- Batch-specific assessments
- Completion certificates

### LMS Live Class

**Integration:** Connects with Zoom via `LMS Zoom Settings`

**Key Fields:**
- `course`, `batch_name`
- `title`, `description`
- `start_time`, `end_time`, `duration`
- `join_url`, `start_url` (Zoom links)

**Auto-notification:** Sends email invites to participants

---

## Assessment System

### LMS Assignment

**Key Fields:**
```python
{
    "title": "Assignment title",
    "question": "Rich Text Editor",
    "course": "Link to LMS Course",
    "type": "Select: Document/Upload/URL",
    "deadline": "DateTime",
    "points": "Int - max points"
}
```

**Submission:** `LMS Assignment Submission` DocType
- Student uploads file/document
- Instructor reviews and assigns grade
- Comments and feedback

### LMS Quiz

**Structure:**
```python
{
    "title": "Quiz title",
    "questions": "Table (LMS Quiz Question)"
}
```

**Question Types:**
- Single choice (radio buttons)
- Multiple choice (checkboxes)
- Open-ended (text input)

**LMS Quiz Submission:** Tracks student attempts and scores

### LMS Exercise (Coding)

**Features:**
- Python/JavaScript code evaluation
- Test cases validation
- `LMS Programming Exercise` for advanced coding challenges

---

## Certification

### LMS Certificate

**Automatic Generation:**
- Issued on course/batch completion
- PDF generation with custom template
- Verification code for authenticity

**Key Fields:**
```python
{
    "member": "Link to User",
    "course": "Link to LMS Course",
    "batch": "Link to LMS Batch (optional)",
    "issue_date": "Date",
    "certificate_pdf": "Attach"
}
```

### LMS Certificate Request

Students can request certificates manually. Requires moderator approval.

---

## User Roles

### LMS Roles

1. **LMS Student**
   - Default role for learners
   - Enroll in courses
   - Submit assignments/quizzes
   - View certificates

2. **Course Creator**
   - Create and manage courses
   - Add lessons, chapters
   - Assign as course instructor
   - **Critical:** Required for Instructors field in courses

3. **Moderator**
   - Full access to all LMS features
   - Manage all courses/batches
   - Approve certificate requests
   - Assign badges

4. **Batch Evaluator**
   - Evaluate batch assessments
   - Grade assignments
   - View batch progress

### Role Assignment

**New Users:** Automatically assigned `LMS Student` role on signup

**Instructors:** Must have `Course Creator` role to be added to course instructors

**Permissions:**
```python
# Check if user is instructor
from lms.lms.utils import has_course_instructor_role
if has_course_instructor_role():
    # Allow action

# Check moderator
from lms.lms.utils import has_moderator_role
if has_moderator_role():
    # Allow administrative action
```

---

## Common Patterns

### 1. Get Course Details

```python
from lms.lms.utils import get_course_details

course = get_course_details(course_name)
# Returns: course object with chapters, lessons, instructors
```

### 2. Get Instructors

```python
from lms.lms.utils import get_instructors

instructors = get_instructors("LMS Course", course_name)
# Returns: [{"name": "user@example.com", "full_name": "John Doe", ...}]
```

### 3. Enrollment

```python
import frappe

# Create enrollment
enrollment = frappe.get_doc({
    "doctype": "LMS Enrollment",
    "course": course_name,
    "member": frappe.session.user
})
enrollment.insert(ignore_permissions=True)
```

### 4. Check Course Access

```python
from lms.lms.utils import has_course_access

if has_course_access(course_name):
    # User is enrolled or course instructor
    pass
```

### 5. Progress Tracking

```python
# LMS Course Progress DocType
{
    "doctype": "LMS Course Progress",
    "member": "user@example.com",
    "course": "course-name",
    "lesson": "lesson-name",
    "status": "Complete/In Progress"
}
```

---

## Frontend (Vue.js)

### Architecture

**Location:** `/home/frappe/frappe-bench/apps/lms/frontend/`

**Tech Stack:**
- Vue 3 (Composition API)
- Vue Router
- Frappe UI components
- TailwindCSS

### Key Pages

```
frontend/src/pages/
├── Home.vue             # Landing page
├── Courses.vue          # Course listing
├── CourseDetail.vue     # Course overview
├── CourseForm.vue       # Create/edit course
├── Lesson.vue           # Lesson viewer
├── LessonForm.vue       # Lesson editor
├── BatchDetail.vue      # Batch overview
├── BatchForm.vue        # Create/edit batch
└── Profile.vue          # User profile
```

### API Calls

**Pattern:**
```javascript
// Using createResource (Frappe UI)
import { createResource } from 'frappe-ui'

const course = createResource({
    url: 'lms.lms.api.get_course_details',
    params: { course: courseName },
    auto: true
})

// Access data: course.data
```

**Common API Methods:**
- `lms.lms.api.get_courses` - List all courses
- `lms.lms.api.get_course_details` - Get course with chapters/lessons
- `lms.lms.api.enroll_in_course` - Enroll student
- `lms.lms.api.save_progress` - Update lesson progress
- `lms.lms.api.get_user_info` - Get current user details

### Build & Development

```bash
# Frontend development
cd /home/frappe/frappe-bench/apps/lms/frontend
yarn install
yarn dev

# Build for production
yarn build
```

---

## Important Utilities

### File: `/home/frappe/frappe-bench/apps/lms/lms/lms/utils.py`

**Key Functions:**
- `get_course_details(course)` - Fetch full course data
- `get_instructors(doctype, docname)` - Get instructors list
- `get_students(course, batch)` - Get enrolled students
- `has_course_instructor_role(member)` - Check if user is instructor
- `has_moderator_role(member)` - Check moderator role
- `is_certified(course)` - Check if user has certificate
- `get_lesson_index(lesson_name)` - Get lesson position
- `get_progress(course, lesson, member)` - Get lesson completion status

### File: `/home/frappe/frappe-bench/apps/lms/lms/lms/api.py`

**Whitelisted Methods:** (Accessible via REST API)
- `get_courses()` - List courses
- `enroll_in_course(course)` - Enroll user
- `update_progress(lesson, course)` - Mark lesson complete
- `get_user_info()` - Get current user data
- `save_evaluation_details()` - Save batch evaluation

---

## Installation Notes

### Dependencies

**Required:**
- Frappe Framework v15+
- No other Frappe apps required

**Optional:**
- `Payments` app - For paid courses (`paid_course` field)

**Install Payments:**
```bash
bench get-app payments
bench --site [sitename] install-app payments
```

### Post-Install

1. **Roles Created:** Course Creator, Moderator, Batch Evaluator, LMS Student
2. **Workspace:** LMS workspace added to Desk
3. **Web Routes:** Automatically configured (`/lms`, `/courses`, `/batches`)

---

## Development Best Practices

### 1. Course Creation Flow

```python
# Create course
course = frappe.get_doc({
    "doctype": "LMS Course",
    "title": "Python Programming",
    "short_introduction": "Learn Python basics",
    "instructors": [{"instructor": "instructor@example.com"}]
})
course.insert()

# Add chapter
chapter = frappe.get_doc({
    "doctype": "Course Chapter",
    "title": "Introduction",
    "course": course.name
})
chapter.insert()

# Add lesson
lesson = frappe.get_doc({
    "doctype": "Course Lesson",
    "title": "Variables",
    "chapter": chapter.name,
    "body": "# Variables in Python..."
})
lesson.insert()

# Link lesson to chapter
course.append("chapters", {
    "chapter": chapter.name,
    "lessons": [{"lesson": lesson.name}]
})
course.save()
```

### 2. Permissions

**Website Access:** Most DocTypes have `allow_guest=1` for public courses

**Instructor Checks:**
```python
# In DocType validation
from lms.lms.utils import has_course_instructor_role

def validate(self):
    if not has_moderator_role() and not has_course_instructor_role():
        frappe.throw("Only instructors can modify this")
```

### 3. Testing

**Test Files:** Located in `lms/lms/doctype/[doctype]/test_[doctype].py`

**Run Tests:**
```bash
bench --site [sitename] run-tests --app lms
bench --site [sitename] run-tests --doctype "LMS Course"
```

---

## Common Issues & Solutions

### 1. Instructor Assignment Error

**Error:** `No permission for Website Route Meta`

**Solution:** Add `Website Manager` role to course creator user

### 2. Paid Course Error

**Error:** `Please install the Payments App`

**Solution:** Either uncheck `paid_course` or install Payments app

### 3. Socket.io CORS Error

**Solution:** Configure `site_config.json`:
```json
{
    "host_name": "http://yoursite.local:8000",
    "socketio_port": "9000"
}
```

### 4. Missing Cohort DocType

**Error:** `DocType Cohort not found`

**Note:** `Cohort` was deprecated and renamed to `LMS Batch`. Old migrations may reference it.

**Solution:** Run `bench migrate`

---

## API Reference Summary

### Course Management
- `frappe.get_doc("LMS Course", course_name)`
- `lms.lms.utils.get_course_details(course)`
- `lms.lms.api.get_courses()`

### Enrollment
- `frappe.get_doc("LMS Enrollment", enrollment_name)`
- `lms.lms.api.enroll_in_course(course)`

### Progress Tracking
- `frappe.get_doc("LMS Course Progress", {...})`
- `lms.lms.api.update_progress(lesson, course)`

### Batch Management
- `frappe.get_doc("LMS Batch", batch_name)`
- `lms.lms.utils.get_students(course, batch)`

### Assessment
- `frappe.get_doc("LMS Assignment", assignment_name)`
- `frappe.get_doc("LMS Quiz", quiz_name)`

---

## Quick Commands

```bash
# Install LMS
bench get-app lms
bench --site [sitename] install-app lms

# Run migrations
bench --site [sitename] migrate

# Clear cache
bench clear-cache

# Restart
bench restart

# Frontend development
cd apps/lms/frontend && yarn dev

# Build frontend
cd apps/lms/frontend && yarn build
```

---

**Documentation:** https://docs.frappe.io/learning  
**Repository:** https://github.com/frappe/lms  
**Demo:** https://frappe.io/learning
