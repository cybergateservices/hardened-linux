## CentOS 7 Server Hardening Guide for STIG DISA RHEL7

The intention of this guide is to help you to harden your CentOS server using OpenSCAP tools. We are using the upstream DISA STIG security profile for Red Hat Enterprise Linux 7 which provides required settings for US Department of Defense systems.

This benchmark is a direct port of a SCAP Security Guide benchmark developed for Red Hat Enterprise Linux. It has been modified through an automated process by the CentOS  developers to remove specific dependencies on Red Hat Enterprise Linux and to work with CentOS. The result is a generally useful SCAP Security Guide benchmark with the following limitations:

* CentOS is not an exact copy of Red Hat Enterprise Linux. There may be configuration differences that produce false positives and/or false negatives. 

* CentOS has its own build system, compiler options, patchsets, and is a community supported, non-commercial operating system. CentOS does not inherit certifications or evaluations from Red Hat Enterprise Linux. As such, some configuration rules (such as those requiring FIPS 140-2 encryption) will continue to fail on CentOS.

It is recommended to visit [DISA STIG profile for Red Hat Enterprise Linux 7](https://static.open-scap.org/ssg-guides/ssg-rhel7-guide-stig-rhel7-disa.html)  to learn more about this profile before you implement it. 

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
### Choosing anaconda security policy
During the installation you can choose the relevant security policy so that you can several security requirement automated. In our kickstart file we will call RHLE


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE5NDQ4OTQyNjVdfQ==
-->