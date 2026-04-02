# Data Analysis (SQL)

## Objective  
To analyze transaction data and identify patterns contributing to transaction failures, user behavior, and system performance.

## Dataset Overview  

The dataset simulates a digital payment system with the following tables:

### 1. Users Table
- user_id  
- user_type (retail, merchant, agent)  
- signup_date  

### 2. Transactions Table
- transaction_id  
- user_id  
- amount  
- status (success, failed, pending)  
- bank_name  
- created_at  

### 3. Failed Transactions Table
- transaction_id  
- failure_reason  
- retry_attempt  

## Key Business Questions  

1. What is the overall transaction success rate?  
2. Which banks have the highest failure rates?  
3. When do transaction failures peak?  
4. Do retries improve success rates?  
5. Which user segment experiences the most failures?  

## Analysis & Queries  
### 1. Transaction Success Rate  

```sql
SELECT 
    COUNT(*) AS total_transactions,
    SUM(CASE WHEN status = 'success' THEN 1 ELSE 0 END) AS successful_transactions,
    (SUM(CASE WHEN status = 'success' THEN 1 ELSE 0 END) * 100.0 / COUNT(*)) AS success_rate
FROM transactions;
```
**Insight:**
This measures overall system reliability.


### 2. Failure Rate by Bank

```sql
SELECT 
    bank_name,
    COUNT(*) AS total_transactions,
    SUM(CASE WHEN status = 'failed' THEN 1 ELSE 0 END) AS failed_transactions,
    (SUM(CASE WHEN status = 'failed' THEN 1 ELSE 0 END) * 100.0 / COUNT(*)) AS failure_rate
FROM transactions
GROUP BY bank_name
ORDER BY failure_rate DESC;
```
**Insight:**
Identifies underperforming financial integrations.

### 3. Failure Rate by Time (Peak Analysis)

```sql
SELECT 
    EXTRACT(HOUR FROM created_at) AS hour,
    COUNT(*) AS total_transactions,
    SUM(CASE WHEN status = 'failed' THEN 1 ELSE 0 END) AS failed_transactions
FROM transactions
GROUP BY hour
ORDER BY hour;
```
**Insight:**
Reveals peak failure periods.

### 4. Retry Effectiveness

```sql
SELECT 
    retry_attempt,
    COUNT(*) AS total_attempts,
    SUM(CASE WHEN t.status = 'success' THEN 1 ELSE 0 END) AS successful_retries
FROM failed_transactions f
JOIN transactions t ON f.transaction_id = t.transaction_id
GROUP BY retry_attempt
ORDER BY retry_attempt;
```
**Insight:**
Determines whether retry systems improve outcomes.

### 5. Failures by User Type

```sql
SELECT 
    u.user_type,
    COUNT(*) AS total_transactions,
    SUM(CASE WHEN t.status = 'failed' THEN 1 ELSE 0 END) AS failed_transactions
FROM transactions t
JOIN users u ON t.user_id = u.user_id
GROUP BY u.user_type;
```
**Insight:**
Highlights which user segments are most affected.

## Key Findings
- Transaction failures are concentrated around specific banks
- Peak hours show increased failure rates
- Retry attempts improve success rates but are not optimized
- Certain user segments experience higher failure rates

## Product Implications
- Improve routing logic for high-failure banks
- Introduce intelligent retry mechanisms
- Optimize system performance during peak hours
- Enhance experience for high-risk user segments

## Conclusion
Data analysis reveals that transaction failures are influenced by both system-level constraints and user interaction patterns.

Addressing these issues requires a combination of:
- Infrastructure improvements
- Product design enhancements
- Better user communication
