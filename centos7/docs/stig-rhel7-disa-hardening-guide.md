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
CentOS 7 Installer  creates he installer's default partitioning scheme creates separate logical volumes for /, /boot, and swap.


<!--stackedit_data:
eyJoaXN0b3J5IjpbNTE0Mjc2OTEyXX0=
-->