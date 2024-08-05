- [Basic Linux Privilege Escalation](https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/)
- [Linux Privilege Escalation](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Linux%20-%20Privilege%20Escalation.md)
- [Checklist - Linux Privilege Escalation](https://book.hacktricks.xyz/linux-unix/linux-privilege-escalation-checklist)
- [Sushant 747's Guide (Country dependant - may need VPN)](https://sushant747.gitbooks.io/total-oscp-guide/content/privilege_escalation_-_linux.html)
- [Course Resource](https://github.com/TCM-Course-Resources/Linux-Privilege-Escalation-Resources)
- 

## Initial Enumeration
- System Enumeration
	- `hostname`
	- `uname - a`
	- `cat /proc/version`
	- `cat /etc/issue`
	- `lscpu`
	- `ps aux`
- User Enumeration
	- `whoami`
	- `id`
	- `sudo -l`
	- `cat /etc/passwd` or `cat /etc/passwd | cut -d : -f 1`
	- `cat /etc/shadow`
	- `history`
	- `sudo su -` 
- Network Enumeration
	- `ifconfig` or `ip a`
	- `ip route`
	- `arp -a` or `ip neigh`
	- `netstat -ano`
- Password Hunting
	- `locate password | more` or `locate pass | more`-- for locating file
	- `grep --color=auto -rnw '/' -ie "PASSWORD=" --color=always 2> /dev/null`
	- `find / -name id_rsa 2> /dev/null`

## Exploring Automated Tools
- [LinPeas](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS)
- [LinEnum](https://github.com/rebootuser/LinEnum)
- [Linux Exploit Suggester](https://github.com/mzet-/linux-exploit-suggester)
- [Linux Priv Checker](https://github.com/sleventyeleven/linuxprivchecker)


## Escalation Path

#### Kernel Exploits
- What is kernel
![[kernel.png]]
- [Linux Kernel Exploits](https://github.com/lucyoa/kernel-exploits) resource

#### Passwords & File Permissions
- 