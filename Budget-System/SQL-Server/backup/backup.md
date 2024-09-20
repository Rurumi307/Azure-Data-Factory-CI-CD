Buckup
===

![SQL_Server_Backup][SQL_Server_Backup]
| PRD                        | QAS                        |
| -------------------------- | -------------------------- |
| 每六小時備份一次, 本機備份 | 每天8:00備份一次, 本機備份 |

``` SQL
-- 切换到 master 数据库并设置数据库为单用户模式
USE master;
GO
ALTER DATABASE [BSDB] SET SINGLE_USER WITH ROLLBACK IMMEDIATE;
GO

-- 备份当前的数据库日志（可选，但推荐）
BACKUP LOG [BSDB] TO DISK = 'E:\Backup\BSDB_LogBackup.bak';
GO

-- 分离数据库
EXEC sp_detach_db @dbname = N'BSDB';
GO

-- 这一步需要手动进行：将日志文件从旧位置移动到新位置
-- 例如，将 D:\DatabaseLog\BSDB_log.ldf 移动到 E:\DatabaseLog\BSDB_log.ldf

-- 重新附加数据库，指定新的日志文件位置
EXEC sp_attach_db @dbname = N'BSDB',
    @filename1 = N'E:\Database\BSDB.mdf', -- 数据文件路径
    @filename2 = N'E:\DatabaseLog\BSDB_log.ldf'; -- 新日志文件路径
GO

-- 将数据库设置为多用户模式
ALTER DATABASE [BSDB] SET MULTI_USER;
GO
```

## References

<!-- url references -->
[SQL_Server_Backup]: ./_assets/SQL_Server_Backup.png