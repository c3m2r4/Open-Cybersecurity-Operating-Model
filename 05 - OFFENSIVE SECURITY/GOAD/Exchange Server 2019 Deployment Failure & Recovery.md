#  Exchange Server 2019 Deployment Failure & Recovery

## GOAD / Ludus Exchange Environment

**Document Type:** Technical Troubleshooting Runbook  
**Environment:** GOAD / Ludus / VMware  
**Server:** `srv01`  
**Exchange Version:** Microsoft Exchange Server 2019  
**Active Directory Domain:** `sevenkingdoms.local`  
**Domain Controller / Global Catalog:** `kingslanding.sevenkingdoms.local`  
**Exchange Organization:** `sevenkingdoms`

---

## 1. Purpose

This runbook documents the troubleshooting and successful remediation of an Exchange Server 2019 installation on `srv01` within a GOAD/Ludus Active Directory lab.

It is intended to provide a repeatable procedure for diagnosing Exchange Setup readiness failures involving:

- Active Directory connectivity
    
- Domain membership
    
- Exchange administrative privileges
    
- .NET Framework 4.8
    
- Visual C++ 2013 Redistributable
    
- IIS URL Rewrite
    
- UCMA 4.0 Runtime
    
- Exchange schema preparation
    
- Exchange Active Directory preparation
    
- Exchange MSI installation
    

---

# 2. Initial Problem

Exchange Setup initially reported multiple readiness failures.

### Initial errors

#### Active Directory domain

> The user isn't logged on to an Active Directory domain.

#### .NET Framework

> This computer requires .NET Framework 4.8.

#### Exchange permissions

> You must be a member of the 'Organization Management' role group or a member of the 'Enterprise Admins' group to continue.

#### First Exchange server installation

> You must use an account that's a member of the Organization Management role group to install or upgrade the first Mailbox server role in the topology.

Similar errors were reported for:

- Mailbox
    
- Client Access
    
- Frontend Transport
    
- Cafe
    

#### Invalid credentials

> Active Directory operation failed ... The supplied credential for `THE-EYRIE\vagrant` is invalid.

#### Active Directory connectivity

> Either Active Directory doesn't exist, or it can't be contacted.

#### UCMA

> This computer requires the Microsoft Unified Communications Managed API 4.0, Core Runtime 64-bit.

#### Visual C++ 2013

> Visual C++ 2013 Redistributable Package is a required component.

---

# 3. Environment Assessment

The first step was to determine whether the server actually had .NET Framework 4.8 installed.

### Command

```powershell
Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\NET Framework Setup\NDP\v4\Full' |
Select-Object Release, Version
```

### Result

```text
Release  Version
-------  -------
528049   4.8.03761
```

### Conclusion

.NET Framework 4.8 was already installed.

Therefore:

**Status: RESOLVED / NOT A CURRENT BLOCKER**

---

# 4. Verify Domain Membership

The server's domain membership was checked using:

```powershell
(Get-CimInstance Win32_ComputerSystem) |
Select-Object Name,Domain,PartOfDomain
```

### Result

```text
Name       Domain              PartOfDomain
----       ------              ------------
THE-EYRIE  sevenkingdoms.local True
```

### Conclusion

`srv01` was correctly joined to:

```text
sevenkingdoms.local
```

The initial "not logged on to an Active Directory domain" error was therefore not caused by the machine being completely outside the domain.

---

# 5. Identify the Correct Exchange Installation Account

The initial Exchange Setup log referenced:

```text
THE-EYRIE\vagrant
```

This was suspicious because the actual AD domain was:

```text
sevenkingdoms.local
```

Later Exchange Setup showed:

```text
Logged on user: SEVENKINGDOMS\administrator
```

This was a significant improvement because the automated Ludus installation was now executing Exchange preparation using the intended privileged domain account.

### Operational lesson

When Exchange reports an invalid credential:

1. Verify the logged-on account.
    
2. Verify the domain.
    
3. Verify that the account is valid.
    
4. Verify required AD/Exchange permissions.
    
5. Avoid creating arbitrary local accounts as a workaround.
    

---

# 6. Exchange Prerequisite Installation

The Ludus Exchange role was inspected to understand what it was doing.

The role contained:

```text
tasks/ludus-exchange-2019-install.yml
```

with the schema preparation command:

```powershell
Setup.exe /IAcceptExchangeServerLicenseTerms /PrepareSchema
```

This confirmed that **Ludus was delegating Exchange preparation to Microsoft's Exchange Setup**, rather than manually controlling individual schema files.

The prerequisite phase subsequently completed the following tasks.

### IIS 6 Compatibility

```text
Install IIS 6 Compatibility Features
```

Status:

```text
changed: [srv01]
```

### .NET Framework

```text
Install .NET Framework
```

Status:

```text
changed: [srv01]
```

The server was subsequently rebooted.

### Visual C++ 2013

```text
Install Visual C++ Redistributable for Visual Studio 2013
```

Status:

```text
changed: [srv01]
```

The server was rebooted.

### IIS URL Rewrite

```text
The IIS URL Rewrite Module is required with Exchange Server 2016 CU22 and Exchange Server 2019 CU11 or later
```

Status:

```text
changed: [srv01]
```

### UCMA 4.0

```text
Install Unified Communications Managed API 4.0 Runtime.
```

Status:

```text
changed: [srv01]
```

The server was rebooted.

### Exchange Windows Features

```text
Install Windows features ADLDS Exchange Transport Hub
```

Status:

```text
ok: [srv01]
```

### Prerequisite conclusion

The Ludus prerequisite stage completed successfully.

---

# 7. Exchange ISO Mount

The Exchange ISO was successfully mounted:

```text
TASK [ludus_exchange : Mount Exchange ISO]
changed: [srv01]
```

Exchange Setup subsequently identified the mounted media as:

```text
E:\
```

and used:

```text
E:\exchangeserver.msi
```

for the actual Exchange installation.

---

# 8. Active Directory Schema Preparation

Ludus executed:

```powershell
Setup.exe /IAcceptExchangeServerLicenseTerms /PrepareSchema
```

Exchange began importing schema definitions using `ldifde.exe`.

The setup log showed operations such as:

```text
PostWindows2003_schema20.ldf
PostWindows2003_schema21.ldf
...
PostWindows2003_schema44.ldf
...
```

The schema preparation targeted:

```text
kingslanding.sevenkingdoms.local
```

and:

```text
CN=Schema,CN=Configuration,DC=sevenkingdoms,DC=local
```

---

# 9. Verify Schema Import Success

Each successful LDIF operation returned:

```text
Process C:\Windows\system32\ldifde.exe finished with exit code 0.
```

`exit code 0` confirmed successful processing of the individual schema import.

The schema process progressed through numerous LDIF files.

A file count was performed:

```powershell
(Get-ChildItem "C:\Windows\Temp\ExchangeSetup\Setup\Data\PostWindows2003_schema*.ldf").Count
```

Result:

```text
100
```

### Important clarification

The Ansible role itself did **not** contain a hardcoded loop of 100 schema files.

Instead, Exchange Setup handled the schema preparation:

```powershell
Setup.exe /PrepareSchema
```

The Exchange build determined which schema files were required.

---

# 10. Schema Preparation Completion

The final setup log showed:

```text
User specified parameters:
-LdapFileName:'Setup\Data\SchemaVersion.ldf'
```

followed by:

```text
Process C:\Windows\system32\ldifde.exe finished with exit code 0.
```

Then:

```text
Ending processing install-ExchangeSchema
Finished executing component tasks.
Ending processing Install-ExchangeOrganization
```

Most importantly:

```text
The Exchange Server setup operation completed successfully.
```

This confirmed that Exchange schema preparation completed successfully.

---

# 11. Prepare Active Directory

After schema preparation, Exchange Setup automatically moved to:

```text
/PrepareAD
```

with:

```text
/OrganizationName:sevenkingdoms
```

The effective command was:

```powershell
/IAcceptExchangeServerLicenseTerms /PrepareAD /OrganizationName:sevenkingdoms /sourcedir:E:
```

The Exchange organization name was therefore:

```text
sevenkingdoms
```

The setup process was logged on as:

```text
SEVENKINGDOMS\administrator
```

---

# 12. Exchange MSI Installation

After AD preparation, Setup moved into the actual Exchange installation phase.

The setup log showed:

```text
Setup will run the task 'install-msipackage'
```

and:

```text
Installing MSI package 'E:\exchangeserver.msi'.
```

The installation target was:

```text
C:\Program Files\Microsoft\Exchange Server\V15
```

The following Exchange components were selected:

```text
AdminTools
Bridgehead
ClientAccess
Mailbox
FrontendTransport
Cafe
AdminToolsNonGateway
```

This confirmed that the process had moved beyond schema/AD preparation and was now installing the actual Exchange Server binaries.

---

# 13. Final Installation State

At this stage the deployment status was:

|Component|Status|
|---|---|
|Domain membership|✅|
|.NET Framework 4.8|✅|
|IIS 6 compatibility|✅|
|Visual C++ 2013|✅|
|IIS URL Rewrite|✅|
|UCMA 4.0|✅|
|Exchange Windows features|✅|
|Exchange ISO|✅|
|Exchange Schema preparation|✅|
|Exchange PrepareAD|✅|
|Exchange MSI installation|🔄 In progress|

---

# 14. Monitoring Commands

## Monitor Exchange setup

```powershell
Get-Content "C:\ExchangeSetupLogs\ExchangeSetup.log" -Tail 30 -Wait
```

## Monitor MSI installation

```powershell
Get-Content "C:\ExchangeSetupLogs\ExchangeSetup.msilog" -Tail 30 -Wait
```

## Check Exchange Setup process

```powershell
Get-Process setup -ErrorAction SilentlyContinue
```

## Check Windows Installer

```powershell
Get-Process msiexec -ErrorAction SilentlyContinue
```

## Check domain membership

```powershell
(Get-CimInstance Win32_ComputerSystem) |
Select-Object Name,Domain,PartOfDomain
```

## Check .NET Framework

```powershell
Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\NET Framework Setup\NDP\v4\Full' |
Select-Object Release,Version
```

---

# 15. Troubleshooting Decision Tree

```text
Exchange Setup fails
        |
        v
Is server domain joined?
        |
   No --+--> Join server to AD
        |
       Yes
        |
        v
Can server contact DC?
        |
   No --+--> Troubleshoot DNS/LDAP/Kerberos
        |
       Yes
        |
        v
Is correct privileged account being used?
        |
   No --+--> Use appropriate domain account
        |
       Yes
        |
        v
Check Exchange prerequisites
        |
        +--> .NET 4.8
        +--> VC++ 2013
        +--> UCMA 4.0
        +--> IIS URL Rewrite
        +--> Windows Features
        |
        v
Run Exchange Schema preparation
        |
        v
PrepareAD
        |
        v
Install Exchange MSI
```

---

# 16. Key Lessons Learned

### 1. Don't manually install Exchange when Ludus is managing it

The GOAD/Ludus role already performs:

- prerequisite installation
    
- ISO mounting
    
- schema preparation
    
- AD preparation
    
- Exchange installation
    

Manual `Setup.exe` execution can interfere with the intended automation workflow.

### 2. `exit code 0` is a successful LDIF operation

For example:

```text
ldifde.exe finished with exit code 0
```

means that particular schema import completed successfully.

### 3. Registry key warnings aren't automatically failures

During first-time Exchange installation, messages such as:

```text
The registry key HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\ExchangeServer\V15\Setup wasn't found.
```

can simply indicate that Exchange isn't installed yet.

The surrounding Setup result must be evaluated before treating such messages as errors.

### 4. Monitor the actual Exchange log

The most useful troubleshooting file was:

```text
C:\ExchangeSetupLogs\ExchangeSetup.log
```

For MSI installation:

```text
C:\ExchangeSetupLogs\ExchangeSetup.msilog
```

These provide considerably more information than simply watching the Ansible task name.

---

# 17. Post-Installation Validation

After Ansible reports successful Exchange installation, perform the following checks.

### Check Exchange services

```powershell
Get-Service *Exchange* |
Select-Object Status,Name,DisplayName
```

### Check Exchange installation directory

```powershell
Test-Path "C:\Program Files\Microsoft\Exchange Server\V15"
```

Expected:

```text
True
```

### Check Exchange Management Shell

If available:

```powershell
Get-ExchangeServer
```

### Check Exchange server

```powershell
Get-ExchangeServer |
Format-List Name,Edition,AdminDisplayVersion
```

### Check Exchange organization

```powershell
Get-OrganizationConfig |
Format-List Name,OrganizationConfigVersion
```

### Check mailbox databases

```powershell
Get-MailboxDatabase
```

### Check Exchange services

```powershell
Test-ServiceHealth
```

All required Exchange services should report healthy.

---

# 18. Success Criteria

The deployment can be considered successful when all of the following are true:

-  `srv01` is joined to `sevenkingdoms.local`
    
-  Domain Controller is reachable
    
-  .NET Framework 4.8 installed
    
-  Visual C++ 2013 installed
    
-  IIS URL Rewrite installed
    
-  UCMA 4.0 installed
    
-  Exchange Windows features installed
    
-  Exchange ISO mounted
    
-  `/PrepareSchema` completed successfully
    
-  `/PrepareAD` completed successfully
    
-  Exchange MSI installation completed successfully
    
-  Exchange services are running
    
-  `Get-ExchangeServer` returns `srv01`
    
-  `Test-ServiceHealth` reports healthy services
    
-  Exchange Management Shell functions correctly
    

---

# 19. Incident Summary

**Problem:** Exchange Server 2019 installation failed readiness checks.

**Primary contributing factors:**

1. Missing Exchange prerequisites.
    
2. Initial AD/domain authentication issue.
    
3. Invalid/incorrect `THE-EYRIE\vagrant` credential reported by Exchange Setup.
    
4. Exchange Setup had not yet completed schema and AD preparation.
    

**Resolution:**

The Ludus Exchange role was allowed to perform its automated prerequisite and Exchange deployment workflow. Required prerequisites were installed, the Exchange ISO was mounted, the AD schema was successfully prepared, `/PrepareAD` was executed using `SEVENKINGDOMS\administrator`, and Exchange subsequently entered the actual MSI installation phase.

**Current state:** Exchange Server 2019 MSI installation is in progress on `srv01`.

---

## 20. Quick Recovery Checklist

For future Exchange deployment failures:

```text
1. Verify domain membership
2. Verify DNS/DC connectivity
3. Verify logged-on domain account
4. Verify required privileges
5. Verify .NET 4.8
6. Install VC++ 2013
7. Install UCMA 4.0
8. Install IIS URL Rewrite
9. Verify Exchange Windows features
10. Mount Exchange ISO
11. Allow Ludus to run /PrepareSchema
12. Allow Ludus to run /PrepareAD
13. Allow Ludus to install Exchange MSI
14. Monitor ExchangeSetup.log
15. Monitor ExchangeSetup.msilog
16. Run Test-ServiceHealth
17. Verify Exchange Server
```

**Do not manually repeat Exchange preparation commands unless the Ludus automation has actually failed and the logs identify a specific reason to do so.**