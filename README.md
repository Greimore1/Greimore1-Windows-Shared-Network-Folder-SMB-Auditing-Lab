Windows Shared Network Folder with SMB Access and Auditing

A hands-on infrastructure project to securely share a folder on a Windows Server VM, access it from a Linux host via SMB, enforce IP-based firewall restrictions, and enable file access audit logging.
Environment Setup

    Windows Server: Windows Server 2019 (hosted in VirtualBox)

    Linux Host: EndeavourOS (bare metal)

    Networking: Host-only Adapter (vboxnet0)

    Shared Folder Location: C:\Shares\SharedFolder

Folder and User Configuration
1. Create the Shared Folder

    Path: C:\Shares\SharedFolder

    Shared using Advanced Sharing

    Share name: SharedFolder

2. Set NTFS and Share Permissions

    User: smbuser

    NTFS Permissions: Full Control

    Share Permissions: Full Control

3. Create Local User smbuser

    Created via compmgmt.msc

    Password set with "Password never expires"

    Member of the Users group only

Screenshot References: 2.png, 3.png
Network Configuration
4. VirtualBox Adapter

    Adapter: Host-only

    Name: vboxnet0

5. Static IP Address (Windows Server)

    IP: 192.168.56.2

    Subnet: 255.255.255.0

    Gateway/DNS: Not required (host-only network)

Screenshot References: 6.png, 5.png
Firewall Configuration
Allow SMB from Linux Host

New-NetFirewallRule -DisplayName "Allow SMB from Host" -Direction Inbound -Protocol TCP -LocalPort 445 -RemoteAddress 192.168.56.1 -Action Allow

Allow Ping (Optional)

Enable-NetFirewallRule -Name FPS-ICMP4-*

Linux SMB Access
6. Mount the Shared Folder

sudo mount -t cifs //192.168.56.2/SharedFolder /mnt/sharedfolder \
  -o credentials=/etc/smb-credentials,vers=3.0,uid=$(id -u),gid=$(id -g),rw,file_mode=0777,dir_mode=0777

Contents of /etc/smb-credentials:

username=smbuser
password=YourPassword
domain=WORKGROUP

Screenshot Reference: 4.png
Audit Logging
7. Enable File Access Auditing

auditpol /set /category:"Object Access" /subcategory:"File System" /success:enable /failure:enable

8. Add Audit Entry to Folder

    Right-click folder > Properties > Security > Advanced > Auditing

    Principal: smbuser

    Type: Success

    Applies to: This folder, subfolders and files

    Permissions: Full Control

Screenshot Reference: 1.png
9. View Logs

    Open eventvwr.msc

    Navigate to: Windows Logs > Security

    Filter by Event ID: 4663

    Confirm access details including user, file path, and access type

Files Accessed
From Linux Host:

echo "test" >> /mnt/sharedfolder/testlog.txt

Successfully logs an Event ID 4663 with:

    User: smbuser

    Access Type: WriteData (or AddFile)

    Object Name: C:\Shares\SharedFolder\testlog.txt
    

Screenshot Reference: 1.png



Summary

Feature	Implemented

Shared folder	Yes

NTFS/share permissions	Yes

Host-only networking	Yes

Static IP on Windows	Yes

Linux mount	Yes

Write access from Linux	Yes

Audit logging	Yes

Event log tracking	Yes
