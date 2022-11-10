# Financial-SQL-Assignment
SQL assignment in reconciling financial transactions

This repo contains:
1. Nine (9) .CSV reports to each question (1a.csv, 1b.csv, 1c.csv, 1d.csv, 2a.csv, 2b.csv, 2c.csv, 2d.csv, 3a.csv).
2. The SQL queries to generate each .csv report (Solution Script.sql)
3. PBIX file of the dashboard (Management dashboard v2.pbix)
Note: This requires a connection to the sqlite db in order to work.

### 1. Finance Reconciliation Reports for Product A, within a given date range:
1a. Provide a summary report for successful ProdA transactions with one line
per day for the given date range. Report header must be: Date, Total_Type0_Amount, Total_Type1_Amount, Total_Amount, Total_TP_Amount

The SQL queries are as follows:

```
select
date(completed_at) as 'Date',
round(cast(sum(t0.Total_Type0_Amount) as decimal(16,2)), 2) as 'Total_Type0_Amount',
round(cast(sum(t1.Total_Type1_Amount) as decimal(16,2)), 2)  as 'Total_Type1_Amount',
round(cast(sum(amount) as decimal(16,2)), 2) as 'Total_Amount',
round(cast(sum(tp.Total_TP_Amount) as decimal(16,2)), 2)  as 'Total_TP_Amount'
from transactions t 
left join (
	select
	t.id,
	amount as 'Total_Type0_Amount'
	from transactions t 
	left join users u on u.id = t.user_id 
	where 1=1
	and u.type='0'
) t0 on t.id = t0.id 
left join (
	select
	t.id,
	amount as 'Total_Type1_Amount'
	from transactions t
	left join users u on u.id = t.user_id 
	where 1=1
	and u.type='1'
) t1 on t.id = t1.id 
left join (
	select 
	tpa.id,
	amount as 'Total_TP_Amount'
	from tpa_recon tpa
) tp on t.id = tp.id 
where 1=1
and t.product = "ProdA"
and t.status = "SUCCESS"
and date(completed_at) between '2021/01/01' and '2022/01/01'
group by date(completed_at)
```

1b. Provide an exception report listing any ProdA transactions in FFI’s DB that
are not present in the TPA recon reports for the given date range. 
Report header must be: Completed_At, Transaction_ID, Amount, User_ID, User_Type, Status

The SQL queries are as follows:

```
select 
t.completed_at as 'Completed_At',
t.id as 'Transaction_ID',
round(cast(t.amount as decimal(16,2)), 2) as 'Amount',
t.user_id as 'User_ID',
u."type" as 'User_Type',
t.status as 'Status'
from transactions t
left join users u on u.id = t.user_id
where 1=1
and product = "ProdA"
and date(completed_at) between '2021/01/01' and '2022/01/01'
and t.id not in (select id 
from tpa_recon tpa)
order by completed_at
```

1c. Provide an exception report listing any transactions in the TPA recon
reports that are not present in FFI’s DB for the given date range. Report header must be: Timestamp, Transaction_ID, Amount

The SQL queries are as follows:

```
select 
tpa."timestamp" as 'Timestamp',
tpa."id" as 'Transaction_ID',
round(cast(tpa."amount" as decimal(16,2)), 2) as 'Amount'
from tpa_recon tpa
where 1=1
and date(tpa."timestamp") between '2021/01/01' and '2022/01/01'
and tpa.id not in (select id
from transactions t
where 1=1
and t.product = "ProdA"
and status= 'SUCCESS')
order by tpa."timestamp"
```

1d. Provide an exception report listing any transactions in FFI’s DB that are
present in the TPA recon reports with a different amount, within the given
date range. Report header must be: Completed_At, Transaction_ID, FFI_Amount, TP_Amount, User_ID, User_Type, Status

The SQL queries are as follows:

```
select
completed_at as 'Completed_At', 
t.id as 'Transaction_ID', 
round(cast(round(t.amount, 2) as decimal(16,2)), 2) as 'FFI_Amount',
round(cast(round(tpa.amount, 2) as decimal(16,2)), 2) as 'TP_Amount',
t.user_id as 'User_ID', 
u."type" as 'User_Type', 
t.status as 'Status'
from transactions t
inner join tpa_recon tpa on tpa.id = t.id
left join users u on u.id = t.user_id
where 1=1
and product = 'ProdA'
and t.amount != tpa.amount
and date(completed_at) between '2021/01/01' and '2022/01/01'
order by completed_at
```

### 2. Finance Reconciliation Reports for Product B, within a given date range:
2a. Provide a summary report for successful ProdB transactions with one line
per day for the given date range. Report header must be: Date, Total_Type0_Amount, Total_Type1_Amount, Total_Amount, Total_TP_Amount

The SQL queries are as follows:
```
select
date(completed_at) as 'Date',
round(cast(sum(t0.Total_Type0_Amount) as decimal(16,2)), 2) as 'Total_Type0_Amount',
round(cast(sum(t1.Total_Type1_Amount) as decimal(16,2)), 2)  as 'Total_Type1_Amount',
round(cast(sum(amount) as decimal(16,2)), 2) as 'Total_Amount',
round(cast(sum(tp.Total_TP_Amount) as decimal(16,2)), 2)  as 'Total_TP_Amount'
from transactions t 
left join (
	select
	t.id,
	amount as 'Total_Type0_Amount'
	from transactions t 
	left join users u on u.id = t.user_id 
	where 1=1
	and u.type='0'
) t0 on t.id = t0.id 
left join (
	select
	t.id,
	amount as 'Total_Type1_Amount'
	from transactions t
	left join users u on u.id = t.user_id 
	where 1=1
	and u.type='1'
) t1 on t.id = t1.id 
left join (
	select 
	tpb.id,
	amount as 'Total_TP_Amount'
	from tpb_recon tpb
) tp on t.id = tp.id 
where 1=1
and t.product = "ProdB"
and t.status = "SUCCESS"
and date(completed_at) between '2021/01/01' and '2022/01/01'
group by date(completed_at)
```

2b. Provide an exception report listing any ProdB transactions in FFI’s DB that
are not present in the TPA recon reports for the given date range. 
Report header must be: Completed_At, Transaction_ID, Amount, User_ID, User_Type, Status

```
select
t.completed_at as 'Completed_At',
t.id as 'Transaction_ID',
round(cast(t.amount as decimal(16,2)), 2) as 'Amount',
t.user_id as 'User_ID',
u."type" as 'User_Type',
t.status as 'Status'
from transactions t
left join users u on u.id = t.user_id
where 1=1
and product = "ProdB"
and date(completed_at) between '2021/01/01' and '2022/01/01'
and t.id not in (select id 
from tpb_recon tpb)
order by t.completed_at
```

2c. Provide an exception report listing any transactions in the TPB recon
reports that are not present in FFI’s DB for the given date range. Report header must be: Timestamp, Transaction_ID, Amount

The SQL queries are as follows:
```
select 
tpb."timestamp" as 'Timestamp',
tpb."id" as 'Transaction_ID',
round(cast(tpb."amount" as decimal(16,2)), 2) as 'Amount'
from tpb_recon tpb
where 1=1
and date(tpb."timestamp") between '2021/01/01' and '2022/01/01'
and tpb.id not in (select id
from transactions t
where 1=1
and t.product = "ProdB"
and status= 'SUCCESS')
order by tpb."timestamp"
```

2d. Provide an exception report listing any transactions in FFI’s DB that are
present in the TPB recon reports with a different amount, within the given
date range. Report header must be: Completed_At, Transaction_ID, FFI_Amount, TP_Amount, User_ID, User_Type, Status

The SQL queries are as follows:

```
select
completed_at as 'Completed_At', 
t.id as 'Transaction_ID', 
round(cast(round(t.amount, 2) as decimal(16,2)), 2) as 'FFI_Amount',
round(cast(round(tpb.amount, 2) as decimal(16,2)), 2) as 'TP_Amount',
t.user_id as 'User_ID', 
u."type" as 'User_Type', 
t.status as 'Status'
from transactions t
inner join tpb_recon tpb on tpb.id = t.id
left join users u on u.id = t.user_id
where 1=1
and product = 'ProdB'
and t.amount != tpb.amount
and date(completed_at) between '2021/01/01' and '2022/01/01'
order by completed_at
```

### 3. Management Summary Report
3a. Provide a report for the most recent 30 days in the database, giving daily Gross Transaction Value for ProdA, ProdB and a total of the two.

The SQL queries are as follows:
```

select 
date(completed_at) as 'Date', 
round(cast(sum(t_ProdA.Total_ProdA_Amount) as decimal(16,2)), 2) as 'Gross Transaction Value for ProdA',
round(cast(sum(t_ProdB.Total_ProdB_Amount) as decimal(16,2)), 2) as 'Gross Transaction Value for ProdB',
round(cast(sum(amount) as decimal(16,2)), 2) as 'Total Gross Transaction Value'
from transactions t
left join (
	select
	t.id,
	amount as 'Total_ProdA_Amount'
	from transactions t  
	where 1=1
	and product = "ProdA"
) t_ProdA on t.id = t_ProdA.id 
left join (
	select
	t.id,
	amount as 'Total_ProdB_Amount'
	from transactions t  
	where 1=1
	and product = "ProdB"
) t_ProdB on t.id = t_ProdB.id
where 1=1
and DATE(completed_at) >= CURRENT_TIMESTAMP -30
and DATE(completed_at) <= DATE(CURRENT_TIMESTAMP)
group by DATE(completed_at)
```

