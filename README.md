# Academic Data Analysis 

## Project Overview

The institution has identified a decline in student academic performance and aims to better understand the underlying causes. This project focuses on analyzing historical academic data, including:

- Student information  
- Course enrollment  
- Attendance records  
- Exam results  
- Faculty and departmental data  

The goal is to generate insights, identify improvement opportunities, and propose strategies to enhance academic outcomes and administrative efficiency.

---

## Objectives

- Analyze student performance across courses  
- Evaluate the relationship between attendance and grades  
- Identify performance patterns and risk groups  
- Assess faculty workload and effectiveness  
- Detect data quality issues (nulls, duplicates)  
- Provide actionable recommendations  

---

## Database

**Schema used:** `AD (Academic Database)`

### Main tables:
```
AD_STUDENT_DETAILS
AD_STUDENT_COURSE_DETAILS
AD_STUDENT_ATTENDANCE
AD_EXAM_RESULTS
AD_FACULTY_DETAILS
AD_FACULTY_COURSE_DETAILS
AD_DEPARTMENTS
AD_COURSE_DETAILS
```

---

# 1️. Student Analysis

## Basic Student Information
```sql
SELECT 
    STUDENT_ID, 
    FIRST_NAME, 
    PARENT_ID, 
    STUDENT_REG_YEAR, 
    EMAIL_ADDR 
FROM AD.AD_STUDENT_DETAILS;
```

## Students Enrolled in Courses
```sql
SELECT 
    S.STUDENT_ID, 
    S.FIRST_NAME, 
    C.COURSE_ID 
FROM AD.AD_STUDENT_DETAILS S 
JOIN AD.AD_STUDENT_COURSE_DETAILS C 
ON S.STUDENT_ID = C.STUDENT_ID;
```

## Attendance
```sql
SELECT 
    S.STUDENT_ID, 
    S.FIRST_NAME, 
    A.NO_OF_WORKING_DAYS, 
    A.NO_OF_DAYS_OFF, 
    A.ELIGIBILITY_FOR_EXAMS 
FROM AD.AD_STUDENT_DETAILS S 
JOIN AD.AD_STUDENT_ATTENDANCE A 
ON S.STUDENT_ID = A.STUDENT_ID;
```

## Exam Results
```sql
SELECT 
    S.STUDENT_ID, 
    S.FIRST_NAME, 
    R.COURSE_ID, 
    R.EXAM_ID, 
    R.MARKS 
FROM AD.AD_STUDENT_DETAILS S 
JOIN AD.AD_EXAM_RESULTS R 
ON S.STUDENT_ID = R.STUDENT_ID;
```

## Average Grade per Course
```sql
SELECT 
    COURSE_ID, 
    AVG(MARKS) AS AVERAGE_MARKS 
FROM AD.AD_EXAM_RESULTS 
GROUP BY COURSE_ID 
ORDER BY AVERAGE_MARKS;
```

## Attendance vs Grades
```sql
SELECT 
    S.STUDENT_ID, 
    A.NO_OF_DAYS_OFF, 
    R.MARKS 
FROM AD.AD_STUDENT_DETAILS S 
JOIN AD.AD_STUDENT_ATTENDANCE A 
ON S.STUDENT_ID = A.STUDENT_ID 
JOIN AD.AD_EXAM_RESULTS R 
ON S.STUDENT_ID = R.STUDENT_ID;
```

## Performance Classification
```sql
SELECT 
    MARKS, 
    CASE 
        WHEN MARKS < 60 THEN 'DEFICIENTE' 
        WHEN MARKS BETWEEN 60 AND 80 THEN 'MEDIO' 
        WHEN MARKS > 80 THEN 'EFICIENTE' 
    END AS PERFORMANCE_LEVEL 
FROM AD.AD_EXAM_RESULTS;
```

## Students per Performance Level
```sql
SELECT 
    CASE 
        WHEN MARKS < 60 THEN 'DEFICIENTE' 
        WHEN MARKS BETWEEN 60 AND 80 THEN 'MEDIO' 
        WHEN MARKS > 80 THEN 'EFICIENTE' 
    END AS PERFORMANCE_LEVEL, 
    COUNT(*) AS TOTAL_STUDENTS 
FROM AD.AD_EXAM_RESULTS 
GROUP BY 
    CASE 
        WHEN MARKS < 60 THEN 'DEFICIENTE' 
        WHEN MARKS BETWEEN 60 AND 80 THEN 'MEDIO' 
        WHEN MARKS > 80 THEN 'EFICIENTE' 
    END;
```

```sql
SELECT 
    CASE 
        WHEN MARKS < 60 THEN 'DEFICIENTE' 
        WHEN MARKS BETWEEN 60 AND 80 THEN 'MEDIO' 
        WHEN MARKS > 80 THEN 'EFICIENTE' 
    END AS PERFORMANCE_LEVEL, 
    COUNT(*) AS TOTAL_STUDENTS 
FROM AD.AD_EXAM_RESULTS 
GROUP BY 
    CASE 
        WHEN MARKS < 60 THEN 'DEFICIENTE' 
        WHEN MARKS BETWEEN 60 AND 80 THEN 'MEDIO' 
        WHEN MARKS > 80 THEN 'EFICIENTE' 
    END;
```

---

# 2️. Faculty Analysis

## Faculty Information
```sql
SELECT 
    FACULTY_ID,
    FIRST_NAME,
    LAST_NAME,
    EMAIL,
    PHONE_NUMBER,
    HIRE_DATE,
    JOB_ID,
    SALARY  
FROM AD.AD_FACULTY_DETAILS;
```

## Faculty Courses
```sql
SELECT 
    FACULTY_ID,
    COURSE_ID  
FROM AD.AD_FACULTY_COURSE_DETAILS;
```

## Faculty Workload
```sql
SELECT  
    F.FACULTY_ID, 
    F.FIRST_NAME || ' ' || F.LAST_NAME AS PROFESOR, 
    COUNT(*) OVER (PARTITION BY F.FACULTY_ID) AS TOTAL_CURSOS
FROM AD.AD_FACULTY_DETAILS F 
JOIN AD.AD_FACULTY_COURSE_DETAILS FC  
ON F.FACULTY_ID = FC.FACULTY_ID;
```

---

# 3️. Integrated Analysis (Full Dataset)

## Unified Table
```sql
SELECT  
    s.STUDENT_ID,
    s.FIRST_NAME AS STUDENT_NAME,
    c.COURSE_NAME,
    d.DEPARTMENT_NAME,
    f.FIRST_NAME || ' ' || f.LAST_NAME AS FACULTY_NAME,
    er.MARKS,
    att.NO_OF_DAYS_OFF
FROM AD.AD_STUDENT_DETAILS s
LEFT JOIN AD.AD_STUDENT_COURSE_DETAILS sc ON s.STUDENT_ID = sc.STUDENT_ID
LEFT JOIN AD.AD_COURSE_DETAILS c ON sc.COURSE_ID = c.COURSE_ID
LEFT JOIN AD.AD_DEPARTMENTS d ON c.DEPARTMENT_ID = d.DEPARTMENT_ID
LEFT JOIN AD.AD_FACULTY_COURSE_DETAILS fc ON c.COURSE_ID = fc.COURSE_ID
LEFT JOIN AD.AD_FACULTY_DETAILS f ON fc.FACULTY_ID = f.FACULTY_ID
LEFT JOIN AD.AD_EXAM_RESULTS er ON s.STUDENT_ID = er.STUDENT_ID
LEFT JOIN AD.AD_STUDENT_ATTENDANCE att ON s.STUDENT_ID = att.STUDENT_ID;
```
