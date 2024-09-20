## 稽核機制

### 新增 稽核機制(BSDB)
```sql
USE master ;  
GO  
-- Create the server audit.   
CREATE SERVER AUDIT [BSDB_Audit]  
    TO FILE (FILEPATH = N'E:\AuditLogs\' ) ;   
GO  
-- Enable the server audit.   
ALTER SERVER AUDIT [BSDB_Audit]   
WITH (STATE = ON) ;  

USE BSDB
GO
CREATE DATABASE AUDIT SPECIFICATION [BSDB_Audit_Spec]
FOR SERVER AUDIT [BSDB_Audit]
ADD (DELETE, SELECT , INSERT ON SCHEMA::STAGE BY dbo)
WITH (STATE = ON);
```


### 查詢 稽核

```sql
USE master;
GO

SELECT
    event_time,
    succeeded,
    server_principal_name,
    object_name,
    statement
FROM sys.fn_get_audit_file('E:\AuditLogs\BSDB_Audit*', NULL, NULL)
order by event_time desc
```