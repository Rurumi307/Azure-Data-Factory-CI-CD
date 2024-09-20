# PSTools

[[_TOC_]]

## 使用PSTools 排查 nt authority\network service 執行bat問題

### 使用Admintistrator 開啟CMD

![PIC][OpenCMD] 

### 切換目錄至 D:\PSTools

```cmd
D:
cd D:\PSTools
```
![PIC][01] 

### 使用network service 開啟cmd

```cmd
psexec -i -u "nt authority\network service" cmd
```
原先的視窗會卡住,並開啟新的視窗    
![PIC][02]   

可以使用whoami 確認身分  
![PIC][03]  

### 切換至需要查bat的目錄下排查問題  
主要是發現TSMTP不同USER會不同,才排查問題  
![PIC][04] 

### 調整檔案

#### **setDate.bat**
最上面添加
```cmd
for /f "tokens=2 delims==" %%i in ('wmic os get LocalDateTime /value') do set datetime=%%i
IF [%TMSTP%] == [] SET TMSTP=%datetime:~0,14%
SET date=%datetime:~0,4%/%datetime:~4,2%/%datetime:~6,2%
```

## References
[OpenCMD]: ./_assets/CMD_Run_Administrator.png
[01]: ./_assets/CDPSTools.png
[02]: ./_assets/Usenetworkservice.png
[03]: ./_assets/whoami.png
[04]: ./_assets/TMSTP.png