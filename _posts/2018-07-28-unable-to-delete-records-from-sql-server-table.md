---
layout: post
title: Unable to delete records from SQL Server table
subtitle: The query processor ran out of stack space during query optimization. Please simplify the query
# image: /img/hello_world.jpeg
---

I recently came across an error I hadn't seen before whilst developing a batch job to update records in a vendor database. On one particular table, any attempt to delete a record resulted in the following error:

*The query processor ran out of stack space during query optimization. Please simplify the query*

The query couldn't have been any simpler, as after profiling the stored procedure in Management Studio, it broke down to a simple DELETE FROM tblA WHERE ID = 1 and the error was still occurring.

Curiously, this error occurred in our test environment, but users were able to perform equivalent delete operations in the Production system with no errors, so there was clearly some inconsistency between environments in play here.

My initial thoughts upon seeing the error were to check for any DML triggers on the table or check constraints that may trigger an infinite loop to cause the error, but nothing of interest was found.

After some Googling, I discovered [this thread](https://social.msdn.microsoft.com/Forums/sqlserver/en-US/1747c0e8-f675-40eb-a3ab-b11c7f7a3f5a/the-query-processor-ran-out-of-stack-space-during-query-optimization-please-simplify-the-query?forum=transactsql)
where other developers reported seeing the same error when a large number of foreign keys referenced the table. That proved to be the issue. After running the sp_fkeys stored procedure against both the test and Production databases, it turned out there were over 2,000 foreign keys referencing the table I was attempting to delete from in the test database! 

The root cause of the 2,000 foreign keys was a feature in the vendor software allowing users to create custom lists of dropdown values. Every time a user created a custom list, the application created a new table in the database with a foreign key pointing back to a parent table containing the names of all custom lists. It turns out that users of the test system had created a lot of custom lists!

After seeing this issue, I've taken away the learning point to think twice before designing an application to create its own database tables at runtime, and in cases where there is a legitimate technical reason to do this, avoid automatically adding a common foreign key to the generated tables.