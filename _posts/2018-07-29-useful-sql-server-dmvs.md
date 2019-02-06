---
layout: post
title: Useful queries for SQL Server Index tuning
subtitle: Making use of dynamic management views to optimise indexes on a SQL Server database
# image: /img/hello_world.jpeg
---

Whilst studying for the Microsoft exam 70-762 (Developing SQL Databases), I read the [official study guide](https://www.microsoftpressstore.com/store/exam-ref-70-762-developing-sql-databases-9781509304912) by Microsoft Press. It highlighted a number of useful SQL queries for analysing indexes on a database. Here are a few examples that I've used the most in my day-to-day work.

#### Review current index usage
```  
SELECT
    OBJECT_NAME(ixu.object_id, 
    DB_ID('DatabaseName')) AS [object_name] ,
    ix.[name] AS index_name ,
    ixu.user_seeks + ixu.user_scans + ixu.user_lookups AS user_reads,
    ixu.user_updates AS user_writes
FROM sys.dm_db_index_usage_stats ixu
INNER JOIN DatabaseName.sys.indexes ix ON
    ixu.[object_id] = ix.[object_id] AND
    ixu.index_id = ix.index_id
WHERE ixu.database_id = DB_ID('DatabaseName')
ORDER BY user_reads DESC;
```

#### Find unused indexes
```
SELECT
    OBJECT_NAME(ix.object_id) AS ObjectName ,
    ix.name
FROM sys.indexes AS ix
INNER JOIN sys.objects AS o ON ix.object_id = o.object_id
WHERE ix.index_id NOT IN (
    SELECT ixu.index_id
    FROM sys.dm_db_index_usage_stats AS ixu
    WHERE
        ixu.object_id = ix.object_id AND
        ixu.index_id = ix.index_id AND
        database_id = DB_ID()
    ) AND
    o.[type] = 'U'
ORDER BY OBJECT_NAME(ix.object_id) ASC ;
```

#### Find indexes that are updated, but never used
```
SELECT
    o.name AS ObjectName ,
    ix.name AS IndexName ,
    ixu.user_seeks + ixu.user_scans + ixu.user_lookups AS user_reads ,
    ixu.user_updates AS user_writes ,
    SUM(p.rows) AS total_rows
FROM sys.dm_db_index_usage_stats ixu
INNER JOIN sys.indexes ix ON ixu.object_id = ix.object_id AND ixu.index_id = ix.index_id
INNER JOIN sys.partitions p ON ixu.object_id = p.object_id AND ixu.index_id = p.index_id
INNER JOIN sys.objects o ON ixu.object_id = o.object_id
WHERE
    ixu.database_id = DB_ID() 
    AND OBJECTPROPERTY(ixu.object_id, 'IsUserTable') = 1 
    AND ixu.index_id > 0
GROUP BY
    o.name ,
    ix.name ,
    ixu.user_seeks + ixu.user_scans + ixu.user_lookups ,
    ixu.user_updates
HAVING ixu.user_seeks + ixu.user_scans + ixu.user_lookups = 0
ORDER BY
    ixu.user_updates DESC,
    o.name ,
    ix.name ;
```

#### Review index fragmentation
```
DECLARE  @db_id SMALLINT, @object_id INT;
SET @db_id = DB_ID(N'WideWorldImporters');
SET @object_id = OBJECT_ID N'WideWorldImporters.Sales.Orders');

SELECT
    ixs.index_id AS idx_id,
    ix.name AS ObjectName,
    index_type_desc,
    page_count,
    avg_page_space_used_in_percent AS AvgPageSpacePct,
    fragment_count AS frag_ct,
    avg_fragmentation_in_percent AS AvgFragPct
FROM sys.dm_db_index_physical_stats
    (@db_id, @object_id, NULL, NULL , 'Detailed') ixs
INNER JOIN sys.indexes ix ON
    ixs.index_id = ix.index_id AND
    ixs.object_id = ix.object_id
ORDER BY avg_fragmentation_in_percent DESC;
```