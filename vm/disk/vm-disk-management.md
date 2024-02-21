VM Disk Management
===

###### tags: `VM` `Disk` `Management`

分割磁碟(C、D、E)
---

### [MS-Learn: Disk Management Initialize new disks][R001]

- Right-Click Windows Icon > Disk Management  
![Disk Management][disk-management]
- Right-Click  Disk 1 > Online  
![Disk Management][Disk-management-online]
- Right-Click Disk 1 > Initialize Disk  
![Disk Management][disk-management-initialize-disk]
- 磁碟分割樣式 MBR 或 GPT（如果磁碟小於 2TB，則選擇 MBR。否則，選擇 GPT。）  
![Disk Management][disk-management-initialize-disk-patition]
- Right-Click the unallocated space on the drive > New Simple Volume  
![Disk Management][Disk-management-simple-volume]
- Next  
![Disk Management][Disk-management-simple-volume-wizard]
![Disk Management][Disk-management-simple-volume-size]
- Select D/E/F  
建議按照順序建立, C>D>E>F..., 如果有光碟機將光碟機一致其他名稱  
![Disk Management][Disk-management-simple-volume-path]
![Disk Management][Disk-management-simple-volume-format-partition]
- Finish  
![Disk Management][Disk-management-simple-volume-finish]

## References


[R001]: 
https://learn.microsoft.com/en-us/windows-server/storage/disk-management/initialize-new-disks

[disk-management]: ./_assets/Disk-management.png
[Disk-management-online]: ./_assets/Disk-management-online.png
[disk-management-initialize-disk]: ./_assets/Disk-management-initialize-disk.png
[disk-management-initialize-disk-patition]: ./_assets/Disk-management-initialize-disk-patition.png
[Disk-management-simple-volume]: ./_assets/Disk-management-simple-volume.png
[Disk-management-simple-volume-wizard]: ./_assets/Disk-management-simple-volume-wizard.png
[Disk-management-simple-volume-size]: ./_assets/Disk-management-simple-volume-size.png
[Disk-management-simple-volume-path]: ./_assets/Disk-management-simple-volume-path.png
[Disk-management-simple-volume-format-partition]: ./_assets/Disk-management-simple-volume-format-partition.png
[Disk-management-simple-volume-finish]: ./_assets/Disk-management-simple-volume-finish.png