# VM 設定 

[[_TOC_]]
## 顧問提供VM建議配置
![pic][01]
## 目前實際配置  
`更新日期: 2024/9/18`  
| 環境           | 伺服器         | CPU | RAM  | C     | D     | E     |
| -------------- | -------------- | --- | ---- | ----- | ----- | ----- |
| AP Server      | IEC1-HPBAP-PRD | 16  | 64GB | 100GB | 300GB |       |
| Essbase Server | IEC1-HPBDB-PRD | 16  | 64GB | 100GB | 1TB   | 500GB |
| DB Server      | IEC1-HPBRE-PRD | 4   | 32GB | 100GB | 100GB | 250GB |
| AP Server      | IEC1-HPBAP-QAS | 16  | 64GB | 100GB | 100GB |       |
| Essbase Server | IEC1-HPBDB-QAS | 16  | 64GB | 100GB | 1TB   | 500GB |
| DB Server      | IEC1-HPBRE-QAS | 4   | 32GB | 100GB | 100GB | 250GB |


## Modules

| Module                         | Description       |
| ------------------------------ | ----------------- |
| [磁碟管理][vm-disk-management] | 磁碟管理          |
| [防火牆設定][vm-firewall]      | 防火牆設定        |
| [SID重複][vm-sid-duplicate]    | SID重複           |
| [遠端連線][vm-remote-desktop]  | 遠端連線Port:3389 |

## 聯絡窗口
`Yu.Mack 游博宇 IEC1`  
`Wu.Harold 吳俊翰 IEC1`



## References
[01]: ./_assets/image.png
[vm-disk-management]: ./disk/vm-disk-management.md
[vm-firewall]: ./firewall/vm-firewall.md
[vm-sid-duplicate]: ./sid/vm-sid-duplicate.md
[vm-remote-desktop]: ./remote-desktop/vm-remote-desktop.md