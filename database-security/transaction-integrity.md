# Transaction Integrity & Conditional Stored Procedures in MySQL

**Category:** Database Security | **Tools:** MySQL Workbench, MySQL 8.0

## Problem
Financial operations (balance transfers, account updates) need to be
atomic — if one part of a multi-step update fails, none of it should
apply. I wanted to practice transaction control (COMMIT/ROLLBACK) and
build a stored procedure that safely reports account status without
exposing raw balance data to every caller.

## Approach
1. Simulated a balance transfer between two accounts inside an explicit
   transaction block:
   ```sql
   start transaction;
   update ACCOUNT set Balance = Balance - 1000 where ID = 2;
   update ACCOUNT set Balance = Balance + 500 where ID = 3;
   rollback;