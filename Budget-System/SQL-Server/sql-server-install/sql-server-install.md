# SQL Server Install

[[_TOC_]]

## 下載SQL Server iso安裝檔

[MS: SQL Server 2022 download][R001]  
會下載 `SQL2022-SSEI-Eval.exe`

### 執行SQL2022-SSEI-Eval.exe

- Check Language(英文) > Choose ISO

    ![SQL Server 2022 download][001]
    ![SQL Server 2022 download][002]  

會產生 `SQLServer2022-x64-ENU.iso`

## 下載 SQL Server Management Studio Management Studio

[MS-Learn: Download SQL Server Management Studio (SSMS)][R002]

### Choose your language

![SSMS download][003]

## Install SQL Server

[mssqltips: install-sql-server-2022][R005]

### 執行安裝檔(setup.exe)

- Click **SQLServer2022-x64-ENU** > **setup.exe**  
    ![Install SQLServer2022][004]

### Installation

- New SQL Server standalone installation or add features to an existing installation

    ![Install SQLServer2022][005]

### Edition

- Select Evalustion  
    Express為免費版，功能較少限制較多。  
    Evaluation為試用版，功能與Enterprise相同，180天後到期。  

    ![Install SQLServer2022][006]

### License Terms

- 勾選accept: I accept the license terms...  
    ![Install SQLServer2022][007]

### Feature Selection

- Select **Database Engine Services**  
    ![Install SQLServer2022][008]

### Instance Configuration

- Default instance 即可  
    ![Install SQLServer2022][009]

### Server Configuration

- Service Accounts  
    ![Install SQLServer2022][010]

- Collation  
  選擇需要相對應的, 這邊選擇Chinese_Taiwan_Stroke_CI_AS

    ![Install SQLServer2022][011]

### Database Engine Configuration

#### Server Configuration Authentication Mode

- Set Specify the password for the SQL Server system administrator(sa) account

    ![Install SQLServer2022][012]

- Add Current User

#### Data Directories

![Install SQLServer2022][013]

#### TempDB

![Install SQLServer2022][014]

#### MaxDOP

- Set CPU cores

    ![Install SQLServer2022][015]

#### Memory

![Install SQLServer2022][016]

#### FILESTREAM

![Install SQLServer2022][017]

### Ready to Install

![Install SQLServer2022][018]

### Complete

![Install SQLServer2022][019]

## Install Microsoft SQL Server Management Studio (SSMS)

### 執行安裝檔(SSMS-Setup-ENU.exe)

- Run **SSMS-Setup-ENU.exe**

    ![Install SSMS][020]

- Click **Install**

    ![Install SSMS][021]

    1. If asked to allow this app to make changes to your device, click **Yes**.
    2. Once the install has completed, if it says a restart is required, click **Restart**.

### Launch the SQL Server Management Studio App

![Install SSMS][022]

## 其他項目

### 公司阻擋安裝SQL Server程式

![Install SQLServer2022][023]

尋求相關人員設定 `Yu.Fish 余美儒 IEC1`

### 設定Firewall

Firewall Warning 情況下

[MS-Learn: Configure the Windows Firewall to allow SQL Server access][R003]

- Open Windows PowerShell Run as administrator

    ![Install SQLServer2022][024]

    ``` shell
    New-NetFirewallRule -DisplayName "SQLServer default instance" -Direction Inbound -LocalPort 1433 -Protocol TCP -Action Allow  
    New-NetFirewallRule -DisplayName "SQLServer Browser service" -Direction Inbound -LocalPort 1434 -Protocol UDP -Action Allow
    ```

### SQL Server 過期, 添加License

[Evaluation period has expired][R004]

申請License人員 `Hung.JamesJX 洪兆禧 IEC1`

## References

[001]: _assets/download-SQL2022.png
[002]: _assets/download-SQL2022-ISO.png
[003]: _assets/download-SSMS.png
[004]: _assets/Click-SQLServer2022-x64-ENU.png
[005]: _assets/SQLServer-Install-Installation.png
[006]: _assets/SQLServer-Install-Edition.png
[007]: _assets/SQLServer-Install-LicenseTerms.png
[008]: _assets/SQLServer-Install-FeatureSelection.png
[009]: _assets/SQLServer-Install-InstanceConfiguration.png
[010]: _assets/SQLServer-Install-ServerConfiguration.png
[011]: _assets/SQLServer-Install-ServerConfiguration-Collation.png
[012]: _assets/SQLServer-Install-DBEngineConfiguration.png
[013]: _assets/SQLServer-Install-DBEngineConfig-Check01.png
[014]: _assets/SQLServer-Install-DBEngineConfig-Check02.png
[015]: _assets/SQLServer-Install-DBEngineConfig-Check03.png
[016]: _assets/SQLServer-Install-DBEngineConfig-Check04.png
[017]: _assets/SQLServer-Install-DBEngineConfig-Check05.png
[018]: _assets/SQLServer-Install-ReadyToInstall.png
[019]: _assets/SQLServer-Install-Complete.png
[020]: _assets/SSMS-Install.png
[021]: _assets/SSMS-Install-Click.png
[022]: _assets/SSMS-Launch.png
[023]: _assets/SQLServer-Install-Fail-RuleCheckResult.png
[024]: _assets/SQLServer-Install-Firewall.png

[R003]: https://learn.microsoft.com/zh-tw/sql/sql-server/install/configure-the-windows-firewall-to-allow-sql-server-access?view=sql-server-ver16
[R004]: https://wiki.sqlfans.cn/mssql/mssql-case-evaluation-period.html
[R005]: https://www.mssqltips.com/sqlservertip/7313/install-sql-server-2022/

<!-- url references -->
[R001]: https://www.microsoft.com/zh-tw/evalcenter/download-sql-server-2022
[R002]: https://learn.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-ver16
