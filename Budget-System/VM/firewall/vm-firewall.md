FireWall Setting
===

###### tags: `VM` `FireWall` 

| Items                            | Default　Listen Port | AP-PRD | DB-PRD |
| -------------------------------- | -------------------- | ------ | ------ |
| Weblogic Admin Server            | 7001                 | V      |        |
| Foundation Services Java Web App | 28080                | V      |        |
| Calculation Manager Java Web App | 8500                 | V      |        |
| Essbase Agent                    | 1423                 |        | V      |
| Essbase Apps(ESSSVR)             | 32768-33768          |        | V      |
| OPMN                             | 6711, 6712           |        | V      |
| Essbase Administration Services  | 10080                | V      |        |
| Provider Services                | 13080                | V      |        |
| Financial Reporting Java Web App | 8200                 | V      |        |
| FR RMI Services                  | 8205-8228            | V      |        |
| Planning Java Web App            | 8300                 | V      |        |
| Planning RMI Server              | 11333                | V      |        |
| Oracle HTTP Server               | 443                  | V      |        |

---
### IEC1-HPBAP-PRD

```powershell
New-NetFirewallRule -DisplayName "Weblogic Admin Server Allow 7001" -Direction Inbound -Protocol TCP -LocalPort 7001 -Action Allow
New-NetFirewallRule -DisplayName "Foundation Services Java Web App Allow 28080" -Direction Inbound -Protocol TCP -LocalPort 28080 -Action Allow
New-NetFirewallRule -DisplayName "Calculation Manager Java Web App Allow 8500" -Direction Inbound -Protocol TCP -LocalPort 8500 -Action Allow
New-NetFirewallRule -DisplayName "Essbase Administration Services Allow 10080" -Direction Inbound -Protocol TCP -LocalPort 10080 -Action Allow
New-NetFirewallRule -DisplayName "Provider Services Allow 13080" -Direction Inbound -Protocol TCP -LocalPort 13080 -Action Allow
New-NetFirewallRule -DisplayName "Financial Reporting Java Web App Allow 8200" -Direction Inbound -Protocol TCP -LocalPort 8200 -Action Allow
New-NetFirewallRule -DisplayName "FR RMI Services Allow 8205-8228" -Direction Inbound -Protocol TCP -LocalPort 8205-8228 -Action Allow
New-NetFirewallRule -DisplayName "Planning Java Web App Allow 8300" -Direction Inbound -Protocol TCP -LocalPort 8300 -Action Allow
New-NetFirewallRule -DisplayName "Planning RMI Server Allow 11333" -Direction Inbound -Protocol TCP -LocalPort 11333 -Action Allow
New-NetFirewallRule -DisplayName "Oracle HTTP Server Allow 443" -Direction Inbound -Protocol TCP -LocalPort 443 -Action Allow
```
### IEC1-HPBDB-PRD
```powershell
New-NetFirewallRule -DisplayName "Essbase Agent Allow 1423" -Direction Inbound -Protocol TCP -LocalPort 1423 -Action Allow
New-NetFirewallRule -DisplayName "Essbase Apps(ESSSVR) Allow 32768-33768" -Direction Inbound -Protocol TCP -LocalPort 32768-33768 -Action Allow
New-NetFirewallRule -DisplayName "OPMN Allow 6711, 6712" -Direction Inbound -Protocol TCP -LocalPort 6711, 6712 -Action Allow
```


## References
