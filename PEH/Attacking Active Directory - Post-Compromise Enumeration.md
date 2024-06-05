- We.ve Compromised a user. Now What?
- There are a few tools that offer quick, and efficient enumeration
	- Bloodhound
	- Plumhound
	- Ldapdomaindump
	- PingCastle

##   Domain Enumeration with ldapdomaindump
- https://github.com/dirkjanm/ldapdomaindump
- `mkdir marvel.local`
- `sudo python3 /opt/ldapdomaindump/ldapdomaindump.py ldaps://10.0.2.15 -u 'MARVEL\pparker' -p passw0rd@123`  --- and -o for output
##   Domain Enumeration with Bloodhound
- neo4j:P@ssword
- `sudo neo4j console` -- required to run bloodhound
- `sudo bloodhound`
- git clone https://github.com/fox-it/BloodHound.py
- cd BloodHound.py
- sudo pip3 install .
- `sudo python3 -m bloodhound -d MARVEL.local -u pparker -p passw0rd@123 -ns 10.0.2.15 -c all`
- The above command will fetch data in json format. Then upload the file in bloodhound

## Domain Enumeration with Plumhound
- https://github.com/PlumHound/PlumHound.git
- `sudo python3 PlumHound.py --easy -p P@ssw0rd`
- `sudo python3 PlumHound.py -x tasks/default.tasks -p P@ssw0rd`
