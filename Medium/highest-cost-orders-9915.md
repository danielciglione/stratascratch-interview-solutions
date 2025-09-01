# StrataScratch Solution: Highest Cost Orders
**Company**: Medium Difficulty | **ID**: 9915  
**Skills**: SQL Window Functions, CTEs, Date Filtering, Aggregations  
**Interview Context**: Data Analyst/Database roles requiring daily spending analysis  

## Problem Statement
Find the customers with the highest daily total order cost between 2019-02-01 and 2019-05-01. If a customer had more than one order on a certain day, sum the order costs on daily basis. Output each customer's first name, total cost of their items, and the date.

For simplicity, you can assume that every first name in the dataset is unique.

## Business Context
**Why this matters in enterprise scenarios:**
- Daily revenue tracking and customer behavior analysis
- Identifying high-value customers for retention strategies
- Understanding spending patterns for inventory management
- Performance optimization for real-time analytics systems (common in T-Mobile-scale environments)

## Solution Approach

**Step 1**: Filter orders within the date range (2019-02-01 to 2019-05-01)  
**Step 2**: Join customers and orders tables to get customer names  
**Step 3**: Group by customer and date to calculate daily totals  
**Step 4**: Use window function to rank customers by daily spend  
**Step 5**: Filter for highest spenders per day  

## ✅ VALIDATED SQL Solution

```sql

WITH daily_customer_totals AS (
    SELECT 
        c.first_name,
        o.order_date,
        SUM(o.total_order_cost) as daily_total_cost
    FROM customers c
    JOIN orders o ON c.id = o.cust_id
    WHERE o.order_date >= TO_DATE('2019-02-01', 'YYYY-MM-DD')
      AND o.order_date <= TO_DATE('2019-05-01', 'YYYY-MM-DD')
    GROUP BY c.first_name, o.order_date
),
ranked_customers AS (
    SELECT 
        first_name,
        order_date,
        daily_total_cost,
        RANK() OVER (PARTITION BY order_date ORDER BY daily_total_cost DESC) as cost_rank
    FROM daily_customer_totals
)
SELECT 
    first_name,
    order_date,
    daily_total_cost as max_cost
FROM ranked_customers
WHERE cost_rank = 1
ORDER BY order_date, first_name;
```

**Oracle-Specific Changes:**
- Added `TO_DATE('YYYY-MM-DD')` for proper date literal conversion
- Oracle requires explicit date format specification
- All other syntax remains compatible with Oracle 19c+

**Alternative Date Format (if TO_DATE doesn't work):**
```sql
-- Try this if the above still gives date errors:
WHERE o.order_date >= DATE '2019-02-01'
  AND o.order_date <= DATE '2019-05-01'
```

## Alternative Approach (Without Window Functions)
```sql
-- Alternative using correlated subqueries (compatible with older SQL versions)
SELECT DISTINCT
    c.first_name,
    o.order_date,
    daily_totals.max_cost
FROM customers c
JOIN orders o ON c.id = o.cust_id
JOIN (
    SELECT 
        c2.first_name,
        o2.order_date,
        SUM(o2.total_order_cost) as max_cost
    FROM customers c2
    JOIN orders o2 ON c2.id = o2.cust_id
    WHERE o2.order_date >= '2019-02-01' 
      AND o2.order_date <= '2019-05-01'
    GROUP BY c2.first_name, o2.order_date
) daily_totals ON c.first_name = daily_totals.first_name 
                AND o.order_date = daily_totals.order_date
WHERE NOT EXISTS (
    SELECT 1
    FROM customers c3
    JOIN orders o3 ON c3.id = o3.cust_id
    WHERE o3.order_date = o.order_date
      AND o3.order_date >= '2019-02-01' 
      AND o3.order_date <= '2019-05-01'
    GROUP BY c3.first_name, o3.order_date
    HAVING SUM(o3.total_order_cost) > daily_totals.max_cost
)
ORDER BY o.order_date, c.first_name;
```

## Business Impact (DBA Perspective)
**Performance Considerations:**
- **Index Strategy**: Composite index on (order_date, cust_id) for optimal join performance
- **Partitioning**: Date-based partitioning would improve this query on large datasets
- **Statistics**: Ensure table statistics are current for accurate execution plans

**Scalability for Enterprise Systems:**
- With 120+ TB data (T-Mobile scale), this pattern requires partitioning and parallel processing
- Memory allocation for window functions needs tuning for optimal performance
- Consider materialized views for frequently accessed daily aggregations

## Expected Output Validation
```
first_name | order_date | max_cost
-----------|------------|----------
Mia        | 2019-02-01 | 100
Farida     | 2019-03-01 | 80
Mia        | 2019-03-01 | 80
Farida     | 2019-03-04 | 100
Farida     | 2019-03-07 | 30
```

## Time Complexity
- **O(n log n)** where n is the number of orders in the date range
- **Space Complexity**: O(n) for intermediate result storage

## Key Learning Points
1. **Window Functions**: RANK() OVER with PARTITION BY for daily rankings
2. **Date Filtering**: Efficient range queries on date columns
3. **Aggregation**: GROUP BY with multiple columns for daily totals
4. **Business Logic**: Handling ties in rankings (RANK vs DENSE_RANK)

**Enterprise Application**: This pattern is fundamental for daily revenue reporting, customer segmentation, and real-time dashboard creation in production systems.

---

## 🎯 Learning Journey & Professional Growth

**Key Technical Challenges Resolved**:
1. ✅ **Database Compatibility**: PostgreSQL ➜ Oracle syntax adaptation
2. ✅ **Date Format Handling**: ORA-01861 resolution using TO_DATE()
3. ✅ **Window Functions**: RANK() vs ROW_NUMBER() decision
4. ✅ **Performance Optimization**: CTE vs Subquery trade-offs

**Enterprise Relevance**: This iterative debugging process directly mirrors production DBA scenarios where solutions must be adapted across different Oracle versions and environments.

### **Professional Insights Gained**
- **Cross-Database Expertise**: Reinforced importance of SGBD-specific knowledge
- **Systematic Debugging**: Applied production troubleshooting methodology
- **Performance Awareness**: Oracle-specific optimization considerations
- **Business Impact Focus**: Connected technical solution to real-world analytics needs

### **Next Strategic Problems**
**Target Improvement**: 85%+ success rate through Oracle-first approach
1. **Date/Time Functions**: Advanced Oracle temporal queries
2. **Complex Joins**: Multi-table optimization scenarios  
3. **Window Functions**: Advanced analytics patterns
4. **Performance Tuning**: Query optimization case studies

**Portfolio Impact**: This transparent learning journey demonstrates continuous improvement mindset and real-world problem-solving approach that resonates with technical hiring managers.

---
