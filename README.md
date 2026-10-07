# SQL Server Open Transaction and Blocked Query Detection

## Objective

The SQL Server Open Transaction and Blocked Query Detection project aimed to automatically identify blocked queries and open transactions that could affect database performance and application availability. The stored procedure monitors SQL Server sessions using system process information and Dynamic Management Functions, identifies blocking conditions and open transactions, and sends automated HTML-formatted email alerts through SQL Server Database Mail. This hands-on project provided practical experience with transaction monitoring, blocking analysis, session troubleshooting, SQL Server system views, T-SQL, and automated database alerts.

### Skills Learned

* Monitoring SQL Server sessions and processes.
* Identifying blocked queries and blocking conditions.
* Detecting open transactions.
* Monitoring transaction and session activity using `sys.sysprocesses`.
* Using `sys.dm_exec_sql_text()` to capture executing SQL statements.
* Analyzing SPID, blocking SPID, database ID, login, host, and command information.
* Identifying long-running blocking conditions using wait time.
* Generating HTML-formatted monitoring reports using T-SQL.
* Configuring and using SQL Server Database Mail for automated alerts.
* Troubleshooting blocking and open transaction issues.
* Applying proactive SQL Server performance monitoring techniques.

### Tools Used

* **Microsoft SQL Server 2016** for database administration and transaction monitoring.
* **SQL Server Management Studio (SSMS)** for T-SQL development and troubleshooting.
* **T-SQL** for stored procedure development and monitoring logic.
* **SQL Server System Views** for monitoring active sessions and transactions.
* **`sys.sysprocesses`** for identifying blocked sessions and open transactions.
* **`sys.dm_exec_sql_text()`** for retrieving SQL statements associated with sessions.
* **SQL Server Database Mail** for automated email notifications.

## Steps

Below are the key steps taken in the open transaction and blocked query detection process:

### 1. Monitor SQL Server Sessions and Blocking Activity

The stored procedure monitors SQL Server processes using `sys.sysprocesses` and retrieves the SQL statement associated with each session using `sys.dm_exec_sql_text()`.

The monitoring process checks for sessions where:

* A session is blocked.
* A session has an open transaction.
* The blocking session has been waiting for an extended period.

The procedure captures important session information such as SPID, blocking SPID, database ID, login time, last batch time, transaction status, hostname, program name, host process, command, login name, and SQL statement.

*Ref 1: Stored Procedure*
`![SQL Server Session and Blocking Monitoring Procedure](https://github.com/Vishnupd/SQL-Server-Open-Transaction-and-Blocked-Query-Detection/blob/main/SP1.png)`
![SQL Server Session and Blocking Monitoring](https://github.com/Vishnupd/SQL-Server-Open-Transaction-and-Blocked-Query-Detection/blob/main/SP2.png)`
![SQL Server Session and Blocking Monitoring](https://github.com/Vishnupd/SQL-Server-Open-Transaction-and-Blocked-Query-Detection/blob/main/SP3.png)`

### 2. Detect Blocked Queries

The procedure identifies blocked sessions using:

```sql
WHERE (req.blocked != 0)
```

The maximum wait time for blocked requests is captured using:

```sql
SELECT @time = MAX(req.waittime)
FROM sys.sysprocesses req
CROSS APPLY sys.dm_exec_sql_text(sql_handle) AS sqltext
WHERE req.blocked != 0
```

The procedure generates an alert when the blocking wait time exceeds **300,000 milliseconds (5 minutes)**.

This helps identify blocking conditions that may have a significant impact on database performance and application users.

*Ref 2.1: Created a blocked query intentionally*
`![Blocked Query Detection](screenshots/02-blocked-query-detection.png)`

*Ref 2.2: Blocked Query Detection*
This screenshot shows the blocking detection logic and the five-minute wait-time threshold.

`![Blocked Query Detection](screenshots/02-blocked-query-detection.png)`

### 3. Capture Blocking and Session Details

When a blocking condition is detected, the procedure collects detailed information about the affected sessions.

The report includes:

* SPID
* Blocking SPID
* Database ID
* Login Time
* Last Batch Time
* Open Transaction Status
* Session Status
* Hostname
* Program Name
* Host Process
* Command
* Login Name
* SQL Statement

The SQL statement is retrieved using:

```sql
CROSS APPLY sys.dm_exec_sql_text(sql_handle) AS sqltext
```

*Ref 3: Blocking Session Details*
This screenshot shows the detailed information captured for blocked sessions, including the blocking session and SQL statement.

`![Blocking Session Details](screenshots/03-blocking-session-details.png)`

### 4. Generate an HTML Blocking Report

The procedure uses:

```sql
FOR XML PATH('tr'), ELEMENTS
```

to convert the SQL Server monitoring results into HTML table rows.

The generated email report provides a structured view of the blocked sessions and makes it easier for the database administrator to investigate the blocking condition.

The HTML report includes columns for:

* SPID
* Blocked
* DBID
* Login Time
* Last Batch
* Open Transaction
* Status
* Hostname
* Program Name
* Host Process
* Command
* Login Name
* SQL Text

### 5. Send Blocked Query Alert

When the maximum blocking wait time exceeds five minutes, the procedure sets the email subject to:

```sql
Blocked queries detected
```

The procedure then sends the HTML report using SQL Server Database Mail:

```sql
EXEC msdb.dbo.sp_send_dbmail
    @profile_name = 'SQL Server Mail Profile',
    @body = @body,
    @body_format = 'HTML',
    @recipients = 'vishnuprasad19931994@gmail.com',
    @subject = @mailsubject;
```

This provides an automated notification to the database administrator when a significant blocking condition is detected.

*Ref 5: Blocked Query Email Alert*
This screenshot shows the automated email notification generated when a blocked query exceeds the configured wait-time threshold.

`![Blocked Query Email Alert](screenshots/05-blocked-query-alert.png)`

### 6. Detect Open Transactions

The procedure separately checks for sessions that have open transactions and have not executed a batch for more than four hours.

The detection logic uses:

```sql
WHERE req.last_batch <= DATEADD(hh,-4,GETDATE())
  AND req.open_tran = 1
```

The count of open transactions is stored in `@p_count`.

If one or more open transactions are detected, the procedure generates a detailed report containing the session and SQL statement information.

This helps identify transactions that may remain open for an extended period and potentially contribute to blocking, resource usage, or transaction log growth.

*Ref 6: Open Transaction Detection*
This screenshot shows the logic used to identify open transactions that have remained inactive for more than four hours.

`![Open Transaction Detection](screenshots/06-open-transaction-detection.png)`

### 7. Generate Open Transaction Alert

When open transactions are detected, the procedure generates another HTML-formatted report and changes the email subject to:

```sql
Open transactions queries detected
```

The email contains details about the sessions with open transactions, including:

* SPID
* Blocking information
* Database ID
* Login Time
* Last Batch
* Open Transaction
* Session Status
* Hostname
* Program Name
* Host Process
* Command
* Login Name
* SQL Statement

The report allows the database administrator to investigate the source of the open transaction and determine whether corrective action is required.

*Ref 7: Open Transaction Email Alert*
This screenshot shows the automated email notification generated when open transactions are detected.

`![Open Transaction Email Alert](screenshots/07-open-transaction-alert.png)`

### 8. Investigate and Troubleshoot Blocking and Open Transactions

The detected sessions were reviewed in detail to identify the root cause of the blocking or open transaction.

The SQL statements were analyzed to determine:

* Which session was causing or experiencing blocking.
* Which database and application were involved.
* Whether a transaction had remained open unnecessarily.
* Whether the SQL statement was waiting on another session.
* Whether application logic or transaction handling needed improvement.
* Whether the blocking condition could affect other users or processes.

The session information and SQL statement were used to investigate the source of the problem and determine the appropriate corrective action.

Where necessary, the transaction or query logic could be reviewed and optimized, and application-side transaction handling could be investigated to prevent recurring blocking conditions.

After corrective action, the sessions were monitored again to confirm that the blocking or open transaction condition had been resolved.
