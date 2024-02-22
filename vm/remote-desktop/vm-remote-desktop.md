VM Remote
===

###### tags: `VM` `Remote` `Firewall`

Enable Remote Access and Connect to a Virtual Machine
---

### [MS-support: How to use Remote Desktop][R001]

- **Start** > **Settings** > **System** > **Remote Desktop** and turn on **Enable Remote Desktop**  
![remote-desktop][remote-desktop-enable]

- **Advanced settings** > Configure Network Level Authentication  
_Network level authentication_ is used for authenticating Remote Desktop services, such as Windows RDP, and Remote Desktop Connection (RDP Client)
![remote-desktop][remote-desktop-enable-advanced-setting]

Add User Account 
---

- **Start** > **Settings** > **Account** > **Other users** > `add a work or school user` / `add someone else to this PC`  
![remote-desktop][remote-desktop-add-account]
- **enter account name** > **select Account type**  
![remote-desktop][remote-desktop-add-account-info]

Select users that can remotely access this PC
---

- **Start** > **Settings** > **System** > **Remote Desktop** > `Select users that can remotely access this PC`  
![remote-desktop][remote-desktop-remote-desktop-users]
- Add user Name or Domain  
![remote-desktop][remote-desktop-remote-desktop-users-add]
![remote-desktop][remote-desktop-remote-desktop-users-add-name]

Firewall Setting 
---

- **Start** > Type **`Windows Firewall`**  
![remote-desktop][remote-desktop-windows-firewall]
- **`allow apps to communicate through windows firewall`** > Check RDP  
![remote-desktop][remote-desktop-windows-firewall-allow]
![remote-desktop][remote-desktop-windows-firewall-check-RDP]
- Go back > **Advanced settings**  
![remote-desktop][remote-desktop-windows-firewall-advanced-settings]
- Inbound Rules  
![remote-desktop][remote-desktop-windows-firewall-inbound-rules]
- Scroll down to find a rule labeled RDP (or using port **_3389_**).  
![remote-desktop][remote-desktop-windows-firewall-inbound-rules-TCP]
- Double-click > Scope  
- Add Remote IP(IPv4)  
Make sure to include your current IP address in the list of allowed Remote IPs  
![remote-desktop][remote-desktop-windows-firewall-inbound-rules-TCP-remote-IP]  
You can use `cmd` type `ipconfig` get IPv4  

    ```cmd
    C:\Users\test>ipconfig
    ```

    ![remote-desktop][remote-desktop-cmd-ipv4]

## References

[R001]: https://support.microsoft.com/en-us/windows/how-to-use-remote-desktop-5fe128d5-8fb1-7a23-3b8a-41e636865e8c#ID0EDD=Windows_10
[R002]: https://support.microsoft.com/en-us/windows/add-or-remove-accounts-on-your-pc-104dc19f-6430-4b49-6a2b-e4dbd1dcdf32#WindowsVersion=Windows_10

[remote-desktop-enable]: ./_assets/remote-desktop-enable.png
[remote-desktop-enable-advanced-setting]: ./_assets/remote-desktop-enable-advanced-setting.png
[remote-desktop-add-account]: ./_assets/remote-desktop-add-account.png
[remote-desktop-add-account-info]: ./_assets/remote-desktop-add-account-info.png
[remote-desktop-remote-desktop-users]: ./_assets/remote-desktop-remote-desktop-users.png
[remote-desktop-remote-desktop-users-add]: ./_assets/remote-desktop-remote-desktop-users-add.png
[remote-desktop-remote-desktop-users-add-name]: ./_assets/remote-desktop-remote-desktop-users-add-name.png
[remote-desktop-windows-firewall]: ./_assets/remote-desktop-windows-firewall.png
[remote-desktop-windows-firewall-allow]: ./_assets/remote-desktop-windows-firewall-allow.png
[remote-desktop-windows-firewall-check-RDP]: ./_assets/remote-desktop-windows-firewall-check-RDP.png
[remote-desktop-windows-firewall-advanced-settings]: ./_assets/remote-desktop-windows-firewall-advanced-settings.png
[remote-desktop-windows-firewall-inbound-rules]: ./_assets/remote-desktop-windows-firewall-inbound-rules.png
[remote-desktop-windows-firewall-inbound-rules-TCP]: ./_assets/remote-desktop-windows-firewall-inbound-rules-TCP.png
[remote-desktop-windows-firewall-inbound-rules-TCP-remote-IP]: ./_assets/remote-desktop-windows-firewall-inbound-rules-TCP-remote-IP.png
[remote-desktop-cmd-ipv4]: ./_assets/remote-desktop-cmd-ipv4.png