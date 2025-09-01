# StrataScratch SQL Solutions 

> Real interview questions from tech companies, solved by a Senior Oracle DBA

## About

This repository contains my solutions to SQL problems from StrataScratch platform. Each solution includes my approach and reasoning, bringing 15+ years of Oracle database experience to data science challenges.

## Current Progress

**Problems Solved**
- Easy: 1 
- Medium: 2
- Hard: 1

## Repository Structure

```
├── Easy/           # Basic SQL problems
├── Medium/         # Intermediate challenges  
├── Hard/           # Advanced problems
└── README.md       # This file
```

## Solution Format

Each solution includes:
- Problem description
- My approach/reasoning
- SQL solution with comments
- Alternative approaches when applicable

## Example Solution

```sql
-- Problem: Find top 3 customers by revenue
-- Approach: Use aggregation with LIMIT

SELECT 
    customer_id,
    customer_name,
    SUM(order_amount) as total_revenue
FROM orders o
JOIN customers c ON o.customer_id = c.id
GROUP BY customer_id, customer_name
ORDER BY total_revenue DESC
LIMIT 3;
```

## Background

Senior Oracle DBA with 15+ years managing enterprise databases, currently learning data science applications. Experience includes:
- 120+ TB database management
- Performance optimization
- Mission-critical systems

## Goals

- Complete 50+ StrataScratch problems
- Document solutions with clear explanations
- Apply database expertise to analytics challenges
- Prepare for international remote opportunities

## Connect

- LinkedIn: [www.linkedin.com/in/danielciglione-oracledba]
- Email: [daniel.ciglione@gmail.com]

---

*Updated: September 2025 | Just getting started! 🚀*
