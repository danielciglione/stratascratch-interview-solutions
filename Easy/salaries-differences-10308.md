# StrataScratch Solution: Salaries Differences
**Company**: Easy Difficulty | **ID**: 10308  
**Skills**: JOINs, MAX aggregation, ABS function, Multi-table queries  
**Interview Context**: HR Analytics, Compensation Analysis, Department Comparison  

## Problem Statement
Calculate the difference between the highest salaries in the marketing and engineering departments. Output just the absolute difference in salaries.

## Business Context
**Why this matters in enterprise scenarios:**
- **Compensation Equity**: Identifying pay gaps between departments
- **Budget Planning**: Understanding cost differences across teams
- **Talent Retention**: Competitive salary benchmarking
- **HR Analytics**: Department-level compensation analysis

## Solution Approach

**Step 1**: Join employee and department tables  
**Step 2**: Filter for Marketing and Engineering departments  
**Step 3**: Find MAX salary for each department  
**Step 4**: Calculate absolute difference  

## Oracle-Optimized Solution

### Primary Solution (Single Query with Conditional MAX)
```sql
-- Oracle Database solution using conditional aggregation
SELECT 
    ABS(
        MAX(CASE WHEN d.department = 'marketing' THEN e.salary END) -
        MAX(CASE WHEN d.department = 'engineering' THEN e.salary END)
    ) AS salary_difference
FROM db_employee e
JOIN db_dept d ON e.department_id = d.id
WHERE d.department IN ('marketing', 'engineering');
```

**Why this approach:**
- Single table scan with efficient JOIN
- Conditional MAX handles both departments in one pass  
- ABS ensures absolute difference as requested
- Optimal execution plan for Oracle optimizer

### Alternative Solution (Subqueries Approach)
```sql
-- Alternative using subqueries for clarity
SELECT 
    ABS(marketing_max - engineering_max) AS salary_difference
FROM (
    SELECT 
        MAX(CASE WHEN d.department = 'marketing' THEN e.salary END) as marketing_max,
        MAX(CASE WHEN d.department = 'engineering' THEN e.salary END) as engineering_max
    FROM db_employee e
    JOIN db_dept d ON e.department_id = d.id
    WHERE d.department IN ('marketing', 'engineering')
);
```

### Detailed Solution (For Debugging)
```sql
-- Step-by-step approach for validation
WITH dept_max_salaries AS (
    SELECT 
        d.department,
        MAX(e.salary) as max_salary
    FROM db_employee e
    JOIN db_dept d ON e.department_id = d.id
    WHERE d.department IN ('marketing', 'engineering')
    GROUP BY d.department
)
SELECT 
    ABS(
        (SELECT max_salary FROM dept_max_salaries WHERE department = 'marketing') -
        (SELECT max_salary FROM dept_max_salaries WHERE department = 'engineering')
    ) AS salary_difference
FROM dual;
```

## Expected Output Format
```
salary_difference
-----------------
15000
```

## Business Impact Analysis

**HR Analytics Perspective:**
- **Pay Equity Analysis**: Critical for identifying compensation gaps
- **Departmental Budgeting**: Understanding cost structures across teams
- **Competitive Benchmarking**: Market positioning analysis
- **Retention Strategy**: Identifying potential flight risks

**Oracle DBA Performance Notes:**
- **Index Strategy**: Composite index on (department_id, salary) for optimal JOIN performance
- **Statistics**: Ensure current statistics on both tables for accurate cost estimation  
- **Execution Plan**: Expect nested loop or hash join depending on data volume

## Key Learning Points

### Technical Skills:
1. **Conditional Aggregation**: MAX with CASE statements
2. **Table JOINs**: Efficient multi-table queries
3. **ABS Function**: Mathematical operations in SQL
4. **Filtering**: WHERE clause optimization

### Business Applications:
- **Compensation Analysis**: Standard HR reporting pattern
- **Department Comparison**: Cross-functional analysis
- **Executive Reporting**: KPI dashboard components
- **Budget Planning**: Resource allocation insights

## Production Considerations

**Scalability for Large HR Systems:**
```sql
-- Optimized for large employee databases
SELECT /*+ USE_NL(e d) INDEX(e, emp_dept_sal_idx) */
    ABS(
        MAX(CASE WHEN d.department = 'marketing' THEN e.salary END) -
        MAX(CASE WHEN d.department = 'engineering' THEN e.salary END)
    ) AS salary_difference
FROM db_employee e
JOIN db_dept d ON e.department_id = d.id
WHERE d.department IN ('marketing', 'engineering');
```

**Monitoring Query Performance:**
```sql
-- Add execution plan analysis
EXPLAIN PLAN FOR
SELECT ABS(MAX(CASE WHEN d.department = 'marketing' THEN e.salary END) -
           MAX(CASE WHEN d.department = 'engineering' THEN e.salary END)) 
FROM db_employee e JOIN db_dept d ON e.department_id = d.id
WHERE d.department IN ('marketing', 'engineering');

SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);
```

## Enterprise Applications

**Similar Use Cases:**
- **Network vs IT Department**: Salary comparison across technical teams
- **Regional Analysis**: Compensation differences across geographic locations  
- **Job Level Analysis**: Pay equity across equivalent roles
- **Performance Metrics**: Department efficiency per salary dollar

**Real-World Extensions:**
```sql
-- Extended analysis for multiple departments
SELECT 
    d1.department as dept1,
    d2.department as dept2,
    ABS(MAX(d1_sal.max_sal) - MAX(d2_sal.max_sal)) as salary_diff
FROM (SELECT department, MAX(salary) as max_sal FROM ...) d1_sal
CROSS JOIN (SELECT department, MAX(salary) as max_sal FROM ...) d2_sal
-- Additional logic for pairwise comparisons
```

## Time & Space Complexity
- **Time Complexity**: O(n + m) where n = employees, m = departments  
- **Space Complexity**: O(1) for aggregation result
- **Oracle Execution**: Hash join with aggregation, typically very efficient

**Enterprise Impact**: This analysis pattern is fundamental for HR dashboards, executive compensation reports, and departmental budget planning in large organizations.
