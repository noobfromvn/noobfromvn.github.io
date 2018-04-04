---
layout: post
title: Monitor index creation or rebuild for huge indexes
tags: [Oracle, Script]
comments: true
---

The following query can be used to monitor the progress of Index creation or Index rebuild for huge indexes.

```sql
SET LINES 200 COL "Index Operation"
FOR A60 TRUNC COL "ETA Mins" FORMAT 999.99 COL "Runtime Mins" FORMAT 999.99
SELECT SESS.SID AS "Session ID" ,
       SQL.SQL_TEXT AS "Index Operation" ,
       LONGOPS.TOTALWORK ,
       LONGOPS.SOFAR ,
       LONGOPS.ELAPSED_SECONDS/60 AS "Runtime Mins" ,
       LONGOPS.TIME_REMAINING/60 AS "ETA Mins"
FROM V$session SESS ,
               V$sql SQL ,
                     V$session_longops LONGOPS
WHERE SESS.SID = LONGOPS.SID
  AND SESS.SQL_ADDRESS = SQL.ADDRESS
  AND SESS.SQL_ADDRESS = LONGOPS.SQL_ADDRESS
  AND SESS.STATUS = 'ACTIVE'
  AND LONGOPS.TOTALWORK > LONGOPS.SOFAR
  AND SESS.SID NOT IN
    ( SELECT SYS_CONTEXT('USERENV', 'SID') SID
     FROM DUAL )
  AND UPPER(SQL.SQL_TEXT) LIKE '%INDEX%'
ORDER BY 3 ,
         4 ;
```

The query will give output like something as follows:

```sql
Session ID Index Operation                                               TOTALWORK      SOFAR Runtime Mins   ETA Mins
---------- ------------------------------------------------------------ ---------- ---------- ------------ ----------
        76 ALTER INDEX ABC.IDX_TEST_IND rebuild parallel 4                  3310       2545   .166666667        .05
        75 ALTER INDEX ABC.IDX_TEST_IND rebuild parallel 4                  3330        620   .166666667 .733333333
        60 ALTER INDEX ABC.IDX_TEST_IND rebuild parallel 4                  4028        715   .166666667 .766666667
        77 ALTER INDEX ABC.IDX_TEST_IND rebuild parallel 4                  2666       2495   .166666667 .016666667
```
