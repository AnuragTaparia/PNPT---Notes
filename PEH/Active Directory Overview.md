

![[Active Directory Components.png]]

# Physical AD Components
### Domain Controller
- A domain controller is a server with AD DS (**Active Directory Domain Services**) server role installed that has specifically been promoted to a Domain Controller
- Domain Controllers:
	- Has a copy of the AD DS directory store
	- Provide authentication and authorization services
	- Replicate updates to other domain controllers in the domain and forest
	- Allow administrative access to manage user accounts and network resources
### AD DS Data Store
- The AD DS data store contains the database files and processes that store and manage directory information for users, services, and applications.
- The AD DS data store:
	- Consists of the Ntds.dit file
	- Is stores by default in the %Systemroot%\NTDS folder on all domain controllers
	- Is accessible only through the domain controller processes and protocols
# Logical AD Components
### AD DS Schema
- Defines every type of object that can be stored in the directory
- Enforces rules regarding object creation and configuration
- ![[AD DS Schema example.png]]
### Domains
- Domains are used to group and manage objects in an organisation
- Domains:
	- An administrative boundary for applying polices to groups of objects
	- A replication boundary for replacing data between domain controllers
	- An authentication and authorization boundary that provides a way to limit the scope of access to resources
### Domain Trees
- A Domain tree is a hierarchy of domains in AD DS
- All domains in the tree:
	- Share a contiguous namespace with the parent domain
	- Can have additional child domains
	- By default create a two-way transitive trust with other domains
### Forests
- A forest is a collection of one or more domain trees
- Forests:
	- Share a common schema
	- Share a common configuration partition
	- Share a common global catalog o enable searching
	- Enable trusts between all domains in the forest
	- Share the Enterprise Admins and Schema Admins groups
### Organizational Units (OUs)
-  OUs are Active Directory containers that can contain users, groups, computers, and other OUs.
- OUs are used to:
	- Represent your organization hierarchically and logically
	- Manage a collection o objects in a consistent way
	- Delegate permissions to administer groups of objects
	- Apply policies
### Trusts
- Trusts provide a mechanism for users to gain access to resources in another domain
- ![[Types of Trusts.png]]
- All domains in a forest trust all other domains in the forest
- Trusts can extend outside the forest
### Objects
![[Objects.png]]