# StrataScratch Solution: New Products
**Company**: Medium Difficulty | **ID**: 10318  
**Skills**: Aggregation, CASE statements, JOINs, Year-over-Year Analysis  
**Interview Context**: Business Intelligence, Product Analytics, Growth Metrics  

## Problem Statement
Calculate the net change in the number of products launched by companies in 2020 compared to 2019. Your output should include the company names and the net difference.

**Formula**: Net difference = Number of products launched in 2020 - Number launched in 2019

## Business Context
**Why this matters in enterprise scenarios:**
- **Product Strategy**: Understanding launch velocity and market expansion
- **Competitive Analysis**: Comparing company innovation rates
- **Business Intelligence**: Tracking growth patterns and strategic shifts
- **Performance Metrics**: KPIs for product development teams

## Solution Approach

**Step 1**: Count products by company for each year (2019, 2020)  
**Step 2**: Calculate difference using conditional aggregation  
**Step 3**: Handle companies that might not have launches in both years  
**Step 4**: Output company name and net difference  

## 🎯 Oracle-Optimized Solution

### Primary Solution (Using Conditional Aggregation)
```sql
-- Oracle Database solution using conditional SUM
SELECT 
    company_name,
    SUM(CASE WHEN year = 2020 THEN 1 ELSE 0 END) -
    SUM(CASE WHEN year = 2019 THEN 1 ELSE 0 END) AS net_difference
FROM car_launches
WHERE year IN (2019, 2020)
GROUP BY company_name
ORDER BY net_difference DESC, company_name;
```

**Why this approach works well:**
- Single table scan (efficient for large datasets)
- Handles missing years automatically (returns 0)
- Clear business logic with CASE statements
- Optimal for Oracle's cost-based optimizer

### Alternative Solution (Using Subqueries)
```sql
-- Alternative approach using subqueries for clarity
SELECT 
    COALESCE(c2020.company_name, c2019.company_name) as company_name,
    COALESCE(c2020.products_2020, 0) - COALESCE(c2019.products_2019, 0) as net_difference
FROM (
    SELECT company_name, COUNT(*) as products_2020
    FROM car_launches
    WHERE year = 2020
    GROUP BY company_name
) c2020
FULL OUTER JOIN (
    SELECT company_name, COUNT(*) as products_2019  
    FROM car_launches
    WHERE year = 2019
    GROUP BY company_name
) c2019 ON c2020.company_name = c2019.company_name
ORDER BY net_difference DESC, company_name;
```

### Advanced Solution (Using Pivot - Oracle 11g+)
```sql
-- Using Oracle PIVOT for elegant year-over-year comparison
SELECT 
    company_name,
    NVL("2020", 0) - NVL("2019", 0) as net_difference
FROM (
    SELECT company_name, year, COUNT(*) as product_count
    FROM car_launches
    WHERE year IN (2019, 2020)
    GROUP BY company_name, year
) 
PIVOT (
    SUM(product_count)
    FOR year IN (2019 AS "2019", 2020 AS "2020")
)
ORDER BY net_difference DESC, company_name;
```

## Business Impact Analysis

**Oracle DBA Perspective - Performance Considerations:**
- **Indexing Strategy**: Composite index on (company_name, year) for optimal performance
- **Partitioning**: Year-based partitioning would benefit this query pattern
- **Statistics**: Ensure histogram statistics on year column for accurate cardinality estimation
- **Execution Plan**: Expect hash aggregation for GROUP BY operations

**Enterprise Scalability:**
- **Large Datasets**: Solution scales well with parallel query execution
- **Real-time Analytics**: Pattern suitable for materialized views with refresh strategies
- **Data Warehouse**: Common pattern for dimensional analysis in OLAP systems

## Expected Business Insights
```
company_name     | net_difference
-----------------|---------------
Tesla           | 5              (launched 5 more products)
Ford            | 2              (moderate growth)  
BMW             | 0              (stable launch rate)
Honda           | -1             (reduced launches)
Toyota          | -3             (significant reduction)
```

## Key Learning Points

### Technical Skills Demonstrated:
1. **Conditional Aggregation**: Using CASE with SUM for pivot-like operations
2. **Year-over-Year Analysis**: Standard business intelligence pattern
3. **NULL Handling**: COALESCE for missing data scenarios
4. **Full Outer Joins**: Ensuring all companies appear in results
5. **Oracle PIVOT**: Modern SQL feature for cleaner code

### Business Intelligence Applications:
- **Trend Analysis**: Understanding product launch patterns
- **Strategic Planning**: Resource allocation for product development
- **Competitive Intelligence**: Market positioning analysis
- **Executive Reporting**: KPI dashboards and executive summaries

## Production Considerations

**Performance Optimization:**
```sql
-- Add execution hints for large datasets
SELECT /*+ PARALLEL(4) */
    company_name,
    SUM(CASE WHEN year = 2020 THEN 1 ELSE 0 END) -
    SUM(CASE WHEN year = 2019 THEN 1 ELSE 0 END) AS net_difference
FROM car_launches
WHERE year BETWEEN 2019 AND 2020
GROUP BY company_name
ORDER BY net_difference DESC;
```

**Monitoring Query:**
```sql
-- Add execution statistics for monitoring
ALTER SESSION SET SQL_TRACE = TRUE;
-- Execute main query
-- Check trace file for performance metrics
```

## Time & Space Complexity
- **Time Complexity**: O(n) where n is number of records in date range
- **Space Complexity**: O(k) where k is number of unique companies
- **Oracle Execution**: Typically uses hash aggregation with sort for ORDER BY

**Enterprise Impact**: This analysis pattern is crucial for quarterly business reviews, strategic planning, and competitive analysis in Fortune 500 environments.
