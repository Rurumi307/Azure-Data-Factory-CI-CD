# SQL Server Permission

[[_TOC_]]

# Create Login

```SQL
USE [master]
GO

/* For security reasons the login is created disabled and with a random password. */
/****** Object:  Login [username]    Script Date: 2024/5/14 下午 04:06:43 ******/
CREATE LOGIN [username] WITH PASSWORD=N'password', DEFAULT_DATABASE=[BSDB], DEFAULT_LANGUAGE=[us_english], CHECK_EXPIRATION=ON, CHECK_POLICY=ON
GO

ALTER LOGIN [username] DISABLE
GO
```

# Table Permission

```SQL
use [dbname]
GO
GRANT DELETE ON [schema].[tablename] TO [username]
GRANT UPDATE ON [schema].[tablename] TO [username]
GRANT INSERT ON [schema].[tablename] TO [username] 
GRANT SELECT ON [schema].[tablename] TO [username]
GO
```
