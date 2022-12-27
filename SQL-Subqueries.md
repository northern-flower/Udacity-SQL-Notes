SQL Subqueries (17)
CTE (with statements)
But in my examples CTE actually took more time to execute ..

With
Inline (where)
Nested (from)
Scalar (1 column or value, performance optimisation)
Correlated


### 1 Provide the name of the sales_rep in each region with the largest amount of total_amt_usd sales.





### 2 For the region with the largest (sum) of sales total_amt_usd, how many total (count) orders were placed?

```sql
select r.id region, count(o.id) cnt, sum(o.total_amt_usd) max_amt_usd
from orders o, accounts a, region r, sales_reps s
where o.account_id = a.id
and s.id = a.sales_rep_id
and r.id = s.region_id
group by r.id
order by 3 desc
limit 1;
```

### 3 How many accounts had more total purchases than the account name which has bought the most standard_qty paper throughout their lifetime as a customer?

```sql
select a.name, sum(o.total) sum_total
from orders o, accounts a
where o.account_id = a.id
group by a.name
having sum(o.total) > (select sum(o.standard_qty) sum_standard
from orders o, accounts a
where o.account_id = a.id
group by a.name
order by 1 desc
limit 1);
```

### 4 For the customer that spent the most (in total over their lifetime as a customer) total_amt_usd, how many web_events did they have for each channel?

```sql
/* 2 - channels for this account and their count */
select w.channel, count(*) ct_ch from (
/* 1 - max spending account (life-time, so would do max but easier with limit to avoid additional subquery) */
select a.id, a.name, sum(o.total_amt_usd) sum
from orders o, accounts a
where o.account_id = a.id
group by a.id, a.name
order by sum desc
limit 1) t1, web_events w
where w.account_id = t1.id
group by w.channel;
```

### 5 What is the lifetime average amount spent in terms of total_amt_usd for the top 10 total spending accounts?

```sql
/* 2 - take average of their sum */
select avg(spent_in_life_time) from (
/* 1 - top 10 life-time-spending accounts */
select a.name, sum(o.total_amt_usd) spent_in_life_time
from orders o, accounts a
where o.account_id = a.id
group by a.name
order by spent_in_life_time desc
limit 10) t1;

/* rewrite using WITH */

with t1 as (select a.name, sum(o.total_amt_usd) spent_in_life_time
from orders o, accounts a
where o.account_id = a.id
group by a.name
order by spent_in_life_time desc
limit 10)
select avg(t1.spent_in_life_time)
from t1;
```

### 6 What is the lifetime average amount spent in terms of total_amt_usd, including only the companies that spent more per order, on average, than the average of all orders?

```sql
/* 2 - take average of their sum */
select avg(spent_in_life_time) from (
/* 1 - accounts spending more than average per order */
select a.name, sum(o.total_amt_usd) spent_in_life_time
from orders o, accounts a
where o.account_id = a.id
and o.total_amt_usd > (select avg(total_amt_usd) from orders)
group by a.name) t1;
```

