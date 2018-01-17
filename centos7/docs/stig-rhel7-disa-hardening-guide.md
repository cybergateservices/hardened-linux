## CentOS 7 Server Hardening Guide for STIG DISA RHEL7

The intention of this guide is to help you to harden your CentOS server using OpenSCAP tools. We are using the upstream DISA STIG security profile for Red Hat Enterprise Linux 7 which provides you a system which is equivalent to US Department of Defense systems which will satisfy majority of your secuio

This benchmark is a direct port of a SCAP Security Guide benchmark developed for Red Hat Enterprise Linux. It has been modified through an automated process by the CentOS  developers to remove specific dependencies on Red Hat Enterprise Linux and to work with CentOS. The result is a generally useful SCAP Security Guide benchmark with the following limitations:

* CentOS is not an exact copy of Red Hat Enterprise Linux. There may be configuration differences that produce false positives and/or false negatives. 

* CentOS has its own build system, compiler options, patchsets, and is a community supported, non-commercial operating system. CentOS does not inherit certifications or evaluations from Red Hat Enterprise Linux. As such, some configuration rules (such as those requiring FIPS 140-2 encryption) will continue to fail on CentOS.

It is recommended to visit [DISA STIG profile for Red Hat Enterprise Linux 7](https://static.open-scap.org/ssg-guides/ssg-rhel7-guide-stig-rhel7-disa.html)  to learn more about this profile before you implement it. 

This guide will not include any  command or configuration codes that you should execute or configure to achieve the security compliances in the SCAP security guide. Instead it  directs you to the appropriate links of the original guide which contains the detailed descriptions, rationales, commands, and remediation scripts that can be used to achieve a single goal by using  a manual execution or configuration pertaining to a specific task .  The guide includes a very high level  description  of what tasks to be done to meet a particular  compliances of the security policy. 

The prime intention of this guide is to provide you  an a automated solution to achieve the compliances in an idempotent and productive nature.   We will be using an Ansible role to automate  majority  of  remediation to meet the compliances required by the DISA STIG Profile. 

## SCAP vs OpenSCAP

[SCAP](https://scap.nist.gov/) or Security Content Automation Protocol is U.S. standard maintained by National Institute of Standards and Technology [NIST](https://www.nist.gov/). OpenSCAP project is a collection of open source tools for implementing and enforcing this standard. Please visit [OpenSCAP](https://www.open-scap.org/) web site to learn more.
## What We Do
In this guide we carry out the following tasks to meet our compliance. These list of hardening task were taken from  [DISA STIG profile for Red Hat Enterprise Linux 7](https://static.open-scap.org/ssg-guides/ssg-rhel7-guide-stig-rhel7-disa.html)

 1.  System Settings 
	 - Installing and Maintaining Software 
	 - File Permissions and Masks 
	 - SELinux 
	 - Account and Access Control 
	 - Network Configuration and Firewalls 
	 - Configure Syslog 
	 - System Accounting with auditd
 2. Services
	- Obsolete Services 
	- Base Services 
	- Cron and At Daemons 
	- SSH Server 
	- System Security Services Daemon 
	- X Window System 
	- Network Time Protocol 
	- Mail Server Software 
	- NFS and RPC 
	- FTP Server  
	- SNMP Server

To meet our compliance we will be carrying out our tasks here in a certain order. Also we will be doing some additional hardening steps which are not specified by the policy.  Some of such task are creating a kickstart installation with disk encryption, creating SSH  keys with high grade encryption etc.
## Kickastart Installation
It will not be possible and practical to change the partition during  remediation steps  on a production systems after having done your security scan. If you installing a system which you need to comply with  this STIG policy we  suggest you to use the [kickstart file](hardened-linux/centos7/ks/stig-centos7-ks.cfg) which we have included in this [GitHub project](https://github.com/cybergateservices/hardened-linux). This section of the document describes the sections of the kickstart file with reference to our security policy. 
### Disk Partitions
CentOS 7 Installer  creates creates separate logical volumes for  ```/, /boot, and swap.``` and you need to creating separate partitions for the following mount points.  For information and understand the rationals behind having separate portions of please refer to the [Disk Partitioning](https://static.open-scap.org/ssg-guides/ssg-rhel7-guide-stig-rhel7-disa.html#xccdf_org.ssgproject.content_group_disk_partitioning) section of the guide. 

 - [Ensure ``/tmp`` Located On Separate Partition](https://static.open-scap.org/ssg-guides/ssg-rhel7-guide-stig-rhel7-disa.html#xccdf_org.ssgproject.content_rule_partition_for_tmp)
 - [Ensure ``/var`` Located On Separate Partition](https://static.open-scap.org/ssg-guides/ssg-rhel7-guide-stig-rhel7-disa.html#xccdf_org.ssgproject.content_rule_partition_for_var)
 - [Ensure ``/var/log/audit Located`` On Separate Partition](https://static.open-scap.org/ssg-guides/ssg-rhel7-guide-stig-rhel7-disa.html#xccdf_org.ssgproject.content_rule_partition_for_var_log_audit)
 - [Ensure ``/home`` Located On Separate Partition](https://static.open-scap.org/ssg-guides/ssg-rhel7-guide-stig-rhel7-disa.html#xccdf_org.ssgproject.content_rule_partition_for_home) 

Additionally we will be creating separate partitions for ```/var, /var/log, /var/tmp, and /var/www``` also.

If you do not need encrypted storage you can avoid the code ```--encrypted --passphrase=PleaseChangeMe```. This  will give you an additional layer of protection.

The relevant kickstart code to achieve our goals  in an automated passion is below.
```
# Create primary system partition for /boot
part /boot --fstype=xfs --size=1024 --fsoptions="rw,nodev,noexec,nosuid"
# Create 30GB physical volume and encrypt it using LUKS
part pv.01  --fstype="lvmpv" --ondisk=vda --size=30720 --encrypted --passphrase=PleaseChangeMe
# Create the volume groups vg_os 
volgroup vg_os pv.01
# Create logical volumes. 
logvol /              --fstype="xfs" --size=6144 --vgname=vg_os --name=lv_root 
logvol /home          --fstype="xfs" --size=2048 --vgname=vg_os --name=lv_home    --fsoptions="rw,nodev,nosuid"
logvol /tmp           --fstype="xfs" --size=1024 --vgname=vg_os --name=lv_tmp     --fsoptions="rw,nodev,noexec,nosuid"
logvol /var           --fstype="xfs" --size=4096 --vgname=vg_os --name=lv_var     --fsoptions="rw,nosuid"
logvol /var/log       --fstype="xfs" --size=1024 --vgname=vg_os --name=lv_var-log --fsoptions="rw,nodev,noexec,nosuid"
logvol /var/log/audit --fstype="xfs" --size=512  --vgname=vg_os --name=lv_var-aud --fsoptions="rw,nodev,noexec,nosuid"
logvol /var/tmp       --fstype="xfs" --size=1024 --vgname=vg_os --name=lv_var-tmp --fsoptions="rw,nodev,noexec,nosuid"
logvol /var/www       --fstype="xfs" --size=1024 --vgname=vg_os --name=lv_var-www --fsoptions="rw,nodev,nosuid"
logvol swap           --fstype="swap" --size=512  --vgname=vg_os --name=lv_swap   --fsoptions="swap"
```
### Choosing an Anaconda Security Policy
During the installation you can choose the relevant security policy so that you can get several security requirement automated. In our kickstart file we will call RHLE7 STIG DISA upstream policy for CentOS 7 by adding the following codes.
```
# OSCAP Anaconda add-on for RHEL7 STIG
%addon org_fedora_oscap
        content-type = scap-security-guide
        profile = stig-rhel7-server-upstream
%end
```
## Installing the Operating System

Using the kickstart file directly from  this site you can install CentOS 7 in any virtual, baremetal, or cloud environment  that support kickstart  installation. At the end of this guide you will learn how to deploy a kickstart installing in  the following environments.

 - Using  [Oracle VirtualBox](https://www.virtualbox.org/) in Linux or Windows 
 - Using [Vulre](https://www.vultr.com/) cloud

Use the the following URL to call your kickstart file during installation. 
```
https://raw.githubusercontent.com/cybergateservices/hardened-linux/master/centos7/ks/stig-centos7-ks.cfg
```
Since the above URL is very long , you run in to problems  when you type  it in the installer as a kernel parameter in the sysntax of ```ks=URL``` . To make the life easier you can use the URL shortener [Git.io](https://git.io/)  to shorten above URL as below.
```
https://git.io/vNCMc
```
## Updating Software
To meet requirements enforced by our security policy we need to keep the software up to date using ```yum``` software management tool.

In this section we need to meet  the following requirements.

 - [Ensure ```gpgcheck``` Enabled In Main Yum Configuration](https://static.open-scap.org/ssg-guides/ssg-rhel7-guide-stig-rhel7-disa.html#xccdf_org.ssgproject.content_rule_ensure_gpgcheck_globally_activated)
 - [Ensure Software Patches Installed](https://static.open-scap.org/ssg-guides/ssg-rhel7-guide-stig-rhel7-disa.html#xccdf_org.ssgproject.content_rule_security_patches_up_to_date)
 - [Ensure YUM Removes Previous Package Versions](https://static.open-scap.org/ssg-guides/ssg-rhel7-guide-stig-rhel7-disa.html#xccdf_org.ssgproject.content_rule_clean_components_post_updating)
 - [Ensure ```gpgcheck``` Enabled for Local Packages ](https://static.open-scap.org/ssg-guides/ssg-rhel7-guide-stig-rhel7-disa.html#xccdf_org.ssgproject.content_rule_ensure_gpgcheck_local_packages)
 - [Ensure ```gpgcheck``` Enabled for Repository Metadata](https://static.open-scap.org/ssg-guides/ssg-rhel7-guide-stig-rhel7-disa.html#xccdf_org.ssgproject.content_rule_ensure_gpgcheck_repo_metadata)

## System and Software Integrity
Please refer to the [System and Software Integrity](https://static.open-scap.org/ssg-guides/ssg-rhel7-guide-stig-rhel7-disa.html#xccdf_org.ssgproject.content_group_integrity)  section of the guide to learn more about this.
## GNOME Desktop Environment 
Since this guide do not use the graphical interface we will not discuss it here. Please refer to the [GNOME Desktop Environment](https://static.open-scap.org/ssg-guides/ssg-rhel7-guide-stig-rhel7-disa.html#xccdf_org.ssgproject.content_group_gnome) section of the STIG guide if you have a requirement to use it. 
## Using Sudo
In Linux ```sudo``` provides the ability to delegate authority to certain users, groups of users, or system administrators. We need to meet the following requirements with ```sudo```

 - [Ensure Users Re-Authenticate for Privilege Escalation - sudo NOPASSWD](https://static.open-scap.org/ssg-guides/ssg-rhel7-guide-stig-rhel7-disa.html#xccdf_org.ssgproject.content_rule_sudo_remove_nopasswd)
 - [Ensure Users Re-Authenticate for Privilege Escalation - sudo !authenticate](https://static.open-scap.org/ssg-guides/ssg-rhel7-guide-stig-rhel7-disa.html#xccdf_org.ssgproject.content_rule_sudo_remove_no_authenticate)
## File Permissions and Masks
Traditional Unix security relies heavily on file and directory permissions to prevent unauthorized users from reading or modifying files to which they should not have access.  In this section of the guide we need to achieve the following requirements.
 - [Add ```nosuid``` Option to Removable Media Partitions](https://static.open-scap.org/ssg-guides/ssg-rhel7-guide-stig-rhel7-disa.html#xccdf_org.ssgproject.content_rule_mount_option_nosuid_removable_partitions)
 - [Add ```nosuid``` Option to ```/home```](https://static.open-scap.org/ssg-guides/ssg-rhel7-guide-stig-rhel7-disa.html#xccdf_org.ssgproject.content_rule_mount_option_home_nosuid)
 - [Disable ```modprobe``` Loading of USB Storage Driver ](https://static.open-scap.org/ssg-guides/ssg-rhel7-guide-stig-rhel7-disa.html#xccdf_org.ssgproject.content_rule_kernel_module_usb-storage_disabled)
 - [Disable the ```autofs``` Automounter](https://static.open-scap.org/ssg-guides/ssg-rhel7-guide-stig-rhel7-disa.html#xccdf_org.ssgproject.content_rule_service_autofs_disabled)
 - [Ensure All Files Are Owned by a User](https://static.open-scap.org/ssg-guides/ssg-rhel7-guide-stig-rhel7-disa.html#xccdf_org.ssgproject.content_rule_no_files_unowned_by_user)
 - [Ensure All World-Writable Directories Are Owned by a System Account](https://static.open-scap.org/ssg-guides/ssg-rhel7-guide-stig-rhel7-disa.html#xccdf_org.ssgproject.content_rule_dir_perms_world_writable_system_owned)
## Restrict Programs from Dangerous Execution Patterns
The recommendations in this section are designed to ensure that the system's features to protect against potentially dangerous program execution are activated.
### Enable ExecShield
[ExecShield](https://static.open-scap.org/ssg-guides/ssg-rhel7-guide-stig-rhel7-disa.html#xccdf_org.ssgproject.content_group_enable_execshield_settings) describes kernel features that provide protection against exploitation of memory corruption errors such as buffer overflows
 - [Enable Randomized Layout of Virtual Address Space](https://static.open-scap.org/ssg-guides/ssg-rhel7-guide-stig-rhel7-disa.html#xccdf_org.ssgproject.content_rule_sysctl_kernel_randomize_va_space)
## SELinux 
SELinux is an Access Control  feature of the Linux kernel which can be used to guard against misconfigured or compromised programs. This guide recommends that SELinux be enabled using the default (targeted) policy on every Red Hat system, unless that system has unusual requirements which make a stronger policy appropriate. Recommends to enable following rules.
 - [Ensure SELinux State is Enforcing](https://static.open-scap.org/ssg-guides/ssg-rhel7-guide-stig-rhel7-disa.html#xccdf_org.ssgproject.content_rule_selinux_state)
 - [Configure SELinux Policy](https://static.open-scap.org/ssg-guides/ssg-rhel7-guide-stig-rhel7-disa.html#xccdf_org.ssgproject.content_rule_selinux_policytype)
 - [Ensure No Device Files are Unlabeled by SELinux](https://static.open-scap.org/ssg-guides/ssg-rhel7-guide-stig-rhel7-disa.html#xccdf_org.ssgproject.content_rule_selinux_all_devicefiles_labeled)
 - [Map System Users To The Appropriate SELinux Role](https://static.open-scap.org/ssg-guides/ssg-rhel7-guide-stig-rhel7-disa.html#xccdf_org.ssgproject.content_rule_selinux_user_login_roles)
##  Account and Access Control 
This section introduces mechanisms for restricting access to accounts under Red Hat Enterprise Linux 7.
### Protect Accounts by Restricting Password-Based Login
Password-based login is vulnerable to guessing of weak passwords, and to sniffing and man-in-the-middle attacks against passwords entered over a network or at an insecure console.  This section enforce the following rules.
#### Restrict Root Login
Direct root logins should be allowed only for emergency use. Please  follow the [recommendation](https://static.open-scap.org/ssg-guides/ssg-rhel7-guide-stig-rhel7-disa.html#xccdf_org.ssgproject.content_group_root_logins) in the guide.
 - [Verify Only Root Has UID 0](https://static.open-scap.org/ssg-guides/ssg-rhel7-guide-stig-rhel7-disa.html#xccdf_org.ssgproject.content_rule_accounts_no_uid_except_zero)
### Verify Proper Storage and Existence of Password Hashes
In Linux password hashes are stored in ```/etc/shadow```. This file should be readable only by processes running with root credentials, preventing users from casually accessing others' password hashes and attempting to crack them. Click [here](https://static.open-scap.org/ssg-guides/ssg-rhel7-guide-stig-rhel7-disa.html#xccdf_org.ssgproject.content_group_password_storage) to learn more.
 - [Prevent Log In to Accounts With Empty Password](https://static.open-scap.org/ssg-guides/ssg-rhel7-guide-stig-rhel7-disa.html#xccdf_org.ssgproject.content_rule_no_empty_passwords)
 - [All GIDs referenced in /etc/passwd must be defined in /etc/group](https://static.open-scap.org/ssg-guides/ssg-rhel7-guide-stig-rhel7-disa.html#xccdf_org.ssgproject.content_rule_gid_passwd_group_same)
### Set Password Expiration Parameters 
Users should be forced to change their passwords, in order to decrease the utility of compromised passwords. However, the need to change passwords often should be balanced against the risk that users will reuse or write down passwords if forced to change them too often.  How it can be done is described [here](https://static.open-scap.org/ssg-guides/ssg-rhel7-guide-stig-rhel7-disa.html#xccdf_org.ssgproject.content_group_password_expiration).
 - [Set Password Minimum Age](https://static.open-scap.org/ssg-guides/ssg-rhel7-guide-stig-rhel7-disa.html#xccdf_org.ssgproject.content_rule_accounts_minimum_age_login_defs)
 - [Set Password Maximum Age](https://static.open-scap.org/ssg-guides/ssg-rhel7-guide-stig-rhel7-disa.html#xccdf_org.ssgproject.content_rule_accounts_maximum_age_login_defs)
 - [Set Existing Passwords Minimum Age](https://static.open-scap.org/ssg-guides/ssg-rhel7-guide-stig-rhel7-disa.html#xccdf_org.ssgproject.content_rule_accounts_password_set_min_life_existing)
 - [Set Existing Passwords Maximum Age](https://static.open-scap.org/ssg-guides/ssg-rhel7-guide-stig-rhel7-disa.html#xccdf_org.ssgproject.content_rule_accounts_password_set_max_life_existing)
 

 

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTg3NTc2MzI0NF19
-->