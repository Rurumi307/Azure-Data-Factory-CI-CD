# MSAD 設定 

[[_TOC_]]

# 建立使用者目錄(MSAD)
## Access Oracle Hyperion Shared Services Console as System Administrator
登入[Budget-QAS][Budget-QAS]  
Select Navigate > Administer > Shared Services Console.  
![Add MSAD][001]  
[Launching Shared Services Console][R001]

## Configure User Directories
Select Administration > Configure User Directories  
![Add MSAD][002]

    The Provider Configuration tab opens. This screen lists all configured user directories, including Native Directory.
### Click New

![Add MSAD][003]  

### Select Directory Type
![Add MSAD][004]

### Click Next
![Add MSAD][005]  

### 填寫MSAD連線資訊
[Connection Information Screen][R002]  
![Add MSAD][006]

| Label      | Value                                                                                     |
| ---------- | ----------------------------------------------------------------------------------------- |
| 名稱       | Budget System AD                                                                          |
| 主機名稱   | iec.inventec                                                                              |
| 連接埠     | 389                                                                                       |
| SSL 已啟用 |                                                                                           |
| 基礎 DN    | DC=iec,DC=inventec                                                                        |
| ID 屬性    | objectguid                                                                                |
| 大小上限   | 0                                                                                         |
| 受信任的   | v                                                                                         |
| 匿名繫結   |                                                                                           |
| 使用者 DN  | CN=IEC1-HBPAdmin,OU=Special Accounts,OU=IEC1,OU=Inventec,DC=iec,DC=inventec |
| 密碼       | password                                                                                  |

### 填寫MSAD 使用者組態
	設定群組結點 e.g. ## TAO Budget System Group 預算系統群組
![Add MSAD][007]  
**登入屬性:** `sAMAccountName`  
**篩選至有限的使用者:** `(&(objectclass=user)(memberOf=CN=\\#\\# TAO Budget System Group 預算系統群組,OU=Other,OU=HR,OU=TAO,OU=Inventec,DC=iec,DC=inventec))`  

### 填寫MSAD 群組組態
	這邊不做任何設定
![Add MSAD][008]


## Restart Hyperion Foundation Services

Windows + R 輸入 Services  
![Resart][009]  

於服務清單重啟Hyperion Foundation Services服務。  
![Resart][010]

## Restart Essbase

VM: IEC1-HPBDB-QAS 
Windows -> Oracle EPM System -> Stop Essbase -> Start Essbase  
![Resart][013]  

# 網域群組申請

HR首頁 -> 員工自助服務區 -> 網路單位表單 -> 電子郵件表單 -> 群組管理申請單

## 網域群組申請範例
![AD Group][011]

# 網域群組添加成員
HR首頁 -> 員工自助服務區 -> 網路單位系統 -> 郵件群組管理新增成員
![AD Group][012]

## References

[001]: ./_assets/1_SharedServicesConsole.png
[002]: ./_assets/2_ConfigureUserDirectories.png
[003]: ./_assets/3_AddMicrosoftActiveDirectory_MSAD.png
[004]: ./_assets/4_SelectDirectoryType.png
[005]: ./_assets/5_ClickNext.png
[006]: ./_assets/MSAD_ConnInfo.png
[007]: ./_assets/MSAD_UserConfigurationScreen.png
[008]: ./_assets/MSAD_GroupConfigurationScreen.png
[009]: ./_assets/win+r_services.png
[010]: ./_assets/_Restart_Hyperion_Foundation_Services.png
[011]: ./_assets/AD_Group.png
[012]: ./_assets/AD_Group_AddUser.png
[013]:./_assets/_Restart_Essbase.png

[Budget-QAS]: https://budget-qas.inventec.com:19000/workspace/index.jsp

[R001]: https://docs.oracle.com/en/applications/enterprise-performance-management/11.2/epmsc/logon.html
[R002]: https://docs.oracle.com/en/applications/enterprise-performance-management/11.2/epmsc/configuring_oid_active_directory_and_other_ldap-based_user_directories.html