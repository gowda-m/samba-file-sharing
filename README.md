Samba File Sharing Between Linux and Windows
Project Overview

This project demonstrates the configuration of Samba file sharing on a Linux server and access from a Windows client. It covers Samba installation, user authentication, Linux ACL permissions, SELinux security contexts, share configuration, and file-access validation.

The project is based on a practical lab environment using Linux command-line tools and Windows File Explorer.

Technologies Used
Linux Server (RHEL-based distribution)
Samba
Windows Client
Access Control Lists (ACL)
SELinux
Linux user and permission management
Network file sharing


1. Install Samba

Install the Samba server and client packages.

dnf install samba samba-client samba-common -y


Screenshot:

!Samba installation

2. Create a Linux User

Create a dedicated account for Samba access.

useradd -s /sbin/nologin user1
passwd user1
smbpasswd -a user1
smbpasswd -e user1

useradd creates the Linux account.
/sbin/nologin prevents normal interactive shell login.
passwd sets the Linux account password.
smbpasswd -a adds the account to Samba's password database.
smbpasswd -e enables the Samba account.

Screenshot:

!Linux user creation

3. Install ACL Utilities

Install the ACL package to manage additional file and directory permissions.

dnf install acl -y


Screenshot:

!ACL package installation

4. Configure SELinux

Assign the Samba share SELinux type to the shared directory.

dnf install policycoreutils-python-utils -y

semanage fcontext -a -t samba_share_t "/opt/shared_folder(/.*)?"

restorecon -Rv /opt/shared_folder


Verify the security context:

ls -Zd /opt/shared_folder


The directory should have the samba_share_t type.

Screenshot:

!SELinux context configuration

5. Configure Directory ACL Permissions

Create the shared directory:

mkdir -p /opt/shared_folder


Grant user1 read, write, and directory traversal permissions:

setfacl -m u:user1:rwx /opt/shared_folder

setfacl -R -m u:user1:rwX /opt/shared_folder

setfacl -m d:u:user1:rwX /opt/shared_folder


Verify the permissions:

getfacl /opt/shared_folder


The default ACL helps ensure that newly created content inherits the intended permissions, subject to normal file-creation rules.

Screenshot:

!Folder ACL permissions

6. Configure Samba Shares

Edit the Samba configuration file:

vi /etc/samba/smb.conf


Example share configuration:

[shared_folder]
    path = /opt/shared_folder
    browseable = yes
    read only = no
    valid users = user1


Save the file and validate the configuration:

testparm


The share name in this example is shared_folder. Use the same name when connecting from Windows.

Screenshot:

!Samba configuration

7. Start and Validate Samba

Enable Samba at boot and start the service:

systemctl enable --now smb
systemctl restart smb
systemctl status smb


Confirm that the service is active.

Screenshot:

!Samba service status

8. Verify Samba Shares

List the available Samba shares from the Linux server:

smbclient -L localhost -U user1


Enter the Samba password when prompted. Confirm that shared_folder appears in the results.

Screenshot:

!Samba share verification

9. Access Samba from Windows

Obtain the Linux server's IP address:

hostname -I


On the Windows client:

Open File Explorer.
Enter the following path in the address bar, replacing SERVER-IP with the Linux server's IP address:

\\SERVER-IP\shared_folder

Enter the Samba username and password when prompted.
Confirm that the shared directory opens successfully.

Screenshot:

!Windows network share

10. Test File Access

Create a test file from Windows in the shared directory. Then verify that it appears on Linux:

ls -l /opt/shared_folder


This validates file sharing between the Windows client and the Linux server. Confirm write access using an actual test before reporting the result.

Screenshot:

!Windows file access test

Troubleshooting
Samba service status
systemctl status smb

Samba configuration validation
testparm

Share listing
smbclient -L localhost -U user1

ACL permissions
getfacl /opt/shared_folder

SELinux context
ls -Zd /opt/shared_folder

Firewall

If firewalld is active, allow Samba traffic:

firewall-cmd --permanent --add-service=samba
firewall-cmd --reload


This hands on project demonstrates Linux-to-Windows file sharing using Samba, with access control managed through Linux ACLs and SELinux.

The repository includes configuration steps, validation commands, and screenshots documenting the implementation.
