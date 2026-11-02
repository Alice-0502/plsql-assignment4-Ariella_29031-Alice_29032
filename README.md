# PL/SQL Assignment IV – Student Registration & Grading System

**Course:** Database Development with PL/SQL (INSY 8311)
**Instructor:** Eric Maniraguha

## Group Members

| Full Name | Student ID |
|-----------|------------|
| Iriza Ntaganda Ariella | 29031 |
| Umutoni Higiro Alice | 29032 |

## Application Description

A small university system that:

- registers **students** and **courses**,
- **enrolls** students in courses (one at a time or many at once),
- records **scores** and computes **letter grades** (A–F),
- prints a **transcript** for a student and a **class list** for a course.

### Requirements of the application

1. A student cannot be enrolled twice in the same course.
2. A score must be between 0 and 100.
3. Grades: A ≥ 80, B ≥ 70, C ≥ 60, D ≥ 50, F < 50.
4. Errors (unknown student, unknown course, invalid score, duplicates) must give a clear message.
5. Enrolling many students at once must be fast and must not stop because of one bad row.

## Database Schema

```
STUDENTS (student_id PK, full_name, email UNIQUE, status CHECK)
    │ 1
    │
    │ N
ENROLLMENTS (enrollment_id PK, student_id FK, course_id FK, score CHECK 0-100, grade CHECK A-F,
             UNIQUE(student_id, course_id))
    │ N
    │
    │ 1
COURSES (course_id PK, course_code UNIQUE, title, credits CHECK 1-6)
```

`ENROLLMENTS` links students and courses (a many-to-many relationship).
`T_ID_LIST` is a schema-level nested table type used to pass a list of student IDs.

## How to Run

Run the scripts **in this order** (SQL Developer, SQL*Plus, or Oracle Live SQL):

1. `00_schema/01_create_tables.sql`
2. `00_schema/02_insert_sample_data.sql`
3. All files in `02_functions/` (**functions first**, because the procedures call them)
4. All files in `01_procedures/`
5. `03_tests/test_success_cases.sql`
6. `03_tests/test_error_cases.sql`

> Note: we compile functions before procedures because procedures such as
> `print_transcript` and `bulk_enroll` call functions. Compiling a procedure first
> would leave it INVALID until the function exists.

## Where Each Concept Is Used

| Requirement | File(s) |
|---|---|
| User-defined record (`TYPE ... IS RECORD`) | `print_transcript.sql` |
| `%ROWTYPE` record | `get_student_average.sql`, `count_students_in_course.sql`, `compute_course_grades.sql` |
| Nested table collection | `t_id_list` (schema), `bulk_enroll.sql`, `compute_course_grades.sql` |
| Associative array (`INDEX BY`) | `compute_course_grades.sql` |
| Implicit cursor – INSERT / UPDATE / DELETE | `add_student.sql`, `update_score.sql`, `delete_enrollment.sql` |
| `SQL%ROWCOUNT`, `SQL%FOUND`, `SQL%NOTFOUND` | `update_score.sql`, `delete_enrollment.sql`, `add_student.sql` |
| Explicit cursor OPEN / FETCH / CLOSE | `print_transcript.sql`, `count_students_in_course.sql` |
| Cursor FOR loop (with parameter) | `list_course_students.sql` |
| `BULK COLLECT` | `compute_course_grades.sql` |
| `FORALL` (INSERT and UPDATE) | `bulk_enroll.sql`, `compute_course_grades.sql` |
| `SAVE EXCEPTIONS` + `SQL%BULK_EXCEPTIONS` | `bulk_enroll.sql` |
| `NO_DATA_FOUND` | `get_student_average.sql`, `find_student_id_by_name.sql` |
| `TOO_MANY_ROWS` | `find_student_id_by_name.sql` |
| `DUP_VAL_ON_INDEX` | `add_student.sql` |
| User-defined exception | `update_score.sql` (`e_invalid_score`), `add_student.sql` (`e_invalid_email`), `bulk_enroll.sql` (`e_bulk_errors`) |
| `RAISE_APPLICATION_ERROR` | all procedures and functions |
| Function called from SQL | `test_success_cases.sql` – TEST S8 |

## Error Codes

| Code | Meaning |
|---|---|
| -20001 | Student does not exist |
| -20002 | Student not enrolled in the course (update) |
| -20003 | Score not between 0 and 100 |
| -20004 | No scored enrollments to grade |
| -20005 | Empty list of students |
| -20006 | Score is NULL |
| -20007 | No student matches the name |
| -20008 | More than one student matches the name |
| -20009 | Course does not exist |
| -20010 | Invalid email |
| -20011 | Duplicate student ID or email |
| -20012 | Nothing to delete |

## Screenshots

See the `screenshots/` folder – one screenshot per test case (success and error).

## Use of AI

We used Claude (an AI assistant by Anthropic) to help us understand the assignment,
explain the PL/SQL concepts, and generate a first version of the code. We then ran
every script ourselves, checked the results, and made sure both members can
explain every line.
