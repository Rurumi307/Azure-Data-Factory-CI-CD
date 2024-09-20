# SSL 設定 

[[_TOC_]]

Planning 系統使用 SSL terminated at Oracle HTTP Server 的設定架構:  
HTTPS requests ------> OHS ------> Application Server  

## SSL 設定步驟
IEC1-HPBAP-QAS Hyperion系統安裝於D:\Hyperion

### Step 1: 建置wallet 以儲存供OHS使用之憑證檔案
憑證將透過ORAPKI utility將XXX.pfx(PKCS格式)檔案轉換成JKS格式後再轉換成wallet。

#### 1. 使用keytool 將PKCS 轉換為 JAVA KEYSTORE  
至D:\Hyperion\jdk\bin 資料夾開啟CMD，執行下列命令  
```cmd
keytool -v -importkeystore -srckeystore D:\SSL\STAR-inventec-com-20240131\STAR_inventec.com\IIS\STAR_inventec.com.pfx -srcstoretype PKCS12 -destkeystore D:\SSL\Hyperion-SSL-202406.jks -deststoretype JKS
```  

```cmd
keytool -v -importkeystore -srckeystore D:\SSL\STAR-inventec-com-20240131\STAR_inventec.com\IIS\STAR_inventec.com.pfx -srcstoretype PKCS12 -destkeystore D:\SSL\Hyperion-SSL-20240627.jks -deststoretype pkcs12
```  
需要輸入金鑰密碼:[Key-infra]
#### 2.使用ORAPKI 將JAVA KEYSTORE 轉換為 WALLET
至D:\Hyperion\oracle_common\bin資料夾
``` cmd
cd D:\Hyperion\oracle_common\bin
```
**設定JAVA_HOME變數:**
```cmd
set JAVA_HOME=D:\Hyperion\jdk
```
#### 3.創立wallet，並設定wallet密碼 [password] 

```cmd
orapki wallet create -wallet D:\Hyperion\user_projects\epmsystem1\httpConfig\ohs\config\fmwconfig\components\OHS\instances\ohs_component\keystores\default_2407 -auto_login -pwd $RFFV5tgb6yhn 
```

<!-- 
```cmd
orapki wallet create -wallet D:\Hyperion\user_projects\epmsystem1\httpConfig\ohs\config\fmwconfig\components\OHS\instances\ohs_component\keystores\default_Sara -pwd ^YHN7ujm8ik, -auto_login
```

```cmd
orapki wallet jks_to_pkcs12 -wallet D:\Hyperion\user_projects\epmsystem1\httpConfig\ohs\config\fmwconfig\components\OHS\instances\ohs_component\keystores\default_Sara -pwd ^YHN7ujm8ik, -keystore D:\SSL\Hyperion-SSL-20240627.jks -jkspwd 2881072128797
``` -->

---
<!-- ``` cmd
orapki wallet create -wallet D:\Hyperion\user_projects\epmsystem1\httpConfig\ohs\config\fmwconfig\components\OHS\instances\ohs_component\keystores\default_Sara -pwd [password] -auto_login 
```

```
orapki wallet add -wallet D:\Hyperion\user_projects\epmsystem1\httpConfig\ohs\config\fmwconfig\components\OHS\instances\ohs_component\keystores\default_Sara -pwd [password] -dn "CN=`budget-qas.inventec.com`" -keysize 1024 -self_signed -validity 365
``` -->
<!-- 
產生憑證簽署要求 (CSR)，並將其新增至您的公事包：
``` cmd
orapki wallet add -wallet D:\Hyperion\user_projects\epmsystem1\httpConfig\ohs\config\fmwconfig\components\OHS\instances\ohs_component\keystores\default_Sara -pwd [password] -dn "CN=`iec.inventec`" -keysize 1024 -self_signed -validity 365
```
將根目錄與中繼憑證新增至受信任 Keystore
``` cmd
orapki wallet add -wallet D:\Hyperion\user_projects\epmsystem1\httpConfig\ohs\config\fmwconfig\components\OHS\instances\ohs_component\keystores\default_Sara -trusted_cert -cert D:\SSL\STAR-inventec-com-20240131\STAR_inventec.com\Apache\STAR_inventec.com.crt -pwd [password] 
``` -->
#### 4.將JKS檔案轉換成wallet
```cmd
orapki wallet jks_to_pkcs12 -wallet D:\Hyperion\user_projects\epmsystem1\httpConfig\ohs\config\fmwconfig\components\OHS\instances\ohs_component\keystores\default_2407 -pwd $RFFV5tgb6yhn -keystore D:\SSL\Hyperion-SSL-202407.jks -jkspwd [infra_pwd] 
```

<!-- 
```cmd
orapki wallet jks_to_pkcs12 -wallet D:\Hyperion\user_projects\epmsystem1\httpConfig\ohs\config\fmwconfig\components\OHS\instances\ohs_component\keystores\default -pwd [password] -keystore D:\SSL\Hyperion-SSL-202406.jks -jkspwd [Key-infra]
``` -->

#### 5.將於以下位置產生wallet.p12檔案
`D:\Hyperion\user_projects\epmsystem1\httpConfig\ohs\config\fmwconfig\components\OHS\instances\ohs_component\keystores\default_2407`

![Step 1][Step1]  

### Step 2:停止ohs_component以及OHS服務。
於D:\Script\Service 執行stop_ohs_component.bat
``` CMD
D:\Script\Service\stop_ohs_component.bat
```

#### Windows + R 輸入 Services

![Step 2][Step2-1]  

#### 於服務清單停止OHS相關服務。
![Step 2][Step2-2]  
![Step 2][Step2-3]  

### Step 3:先備份原始ssl.conf檔案後再修改ssl.conf檔案

`D:\Hyperion\user_projects\epmsystem1\httpConfig\ohs\config\fmwconfig\components\OHS\ohs_component\ssl.conf`

```cmd
copy D:\Hyperion\user_projects\epmsystem1\httpConfig\ohs\config\fmwconfig\components\OHS\ohs_component\ssl.conf D:\Hyperion\user_projects\epmsystem1\httpConfig\ohs\config\fmwconfig\components\OHS\ohs_component\ssl_bk.conf
```

1. 預設Listen 4443，修改為`Listen 443`  
   ![Step 3][Step3-1]  
2. Path to Wallet為預設資料夾  
   ![Step 3][Step3-2]   
3. 將下圖文字貼到</VirtualHost>前，並修改最後一段port為19443  
   ![Step 3][Step3-3]  
   ```conf
   Include "mod_wl_ohs.conf"
   Include "epm_online_help.conf"
   Include "epm_rewrite_rules.conf"
   Include "epm.conf"
   Include "deflate.conf"
   ```

### Step 4:先備份原始httpd.conf檔案後再修改httpd.conf檔案

`D:\Hyperion\user_projects\epmsystem1\httpConfig\ohs\config\fmwconfig\components\OHS\ohs_component\httpd.conf`    
於設定檔內加入include "ssl.conf"  
![Step 3][Step4-1]  

### Step 5:啟用OHS服務以及ohs_component

#### 於服務清單啟用OHS相關服務
Windows + R 輸入 Services  
![Step 2][Step2-1]  

#### 執行start_ohs_component.bat
於D:\Script\Service 執行start_ohs_component.bat
``` CMD
D:\Script\Service\start_ohs_component.bat
```

### Step 6: 測試連線至[Budget-QAS][Budget-QAS]成功

## References

### [Oracle® Hyperion EPM System with SSL Enabled][R001]

[Step1]: ./_assets/wallet_p12_file.png
[Step2-1]: ./_assets/win+r_services.png
[Step2-2]: ./_assets/Stop_OHS_Service_1.png
[Step2-3]: ./_assets/Stop_OHS_Service_2.png
[Step3-1]: ./_assets/ssl_conf_set.png
[Step3-2]: ./_assets/Path_to_Wallet.png
[Step3-3]: ./_assets//Set_ifmodule_port.png
[Step4-1]: ./_assets/Set_httpd.png

[Budget-QAS]: https://budget-qas.inventec.com:19000/workspace/index.jsp

[R001]: https://www.oracle.com/middleware/technologies/pm-v1112-doc.html