

Your active GOAD inventory intentionally uses:

```text
ansible_user=vagrant
ansible_password=vagrant
```

That is the **local provisioning account**, not the domain Administrator credential.

Your `inventory_disable_vagrant` shows the domain Administrator credentials. For the `sevenkingdoms.local` domain controller:

```text
administrator@sevenkingdoms.local
password: 8dCT-DJjgScp
```

So for your `the-eyrie` machine, **don't use `the-eyrie\vagrant` for Kerberos**. Use the domain account.

### On `the-eyrie`

Open a new PowerShell process:

```powershell
runas /user:SEVENKINGDOMS\Administrator powershell.exe
```

When prompted, enter the GOAD domain Administrator password from the inventory.

Then, **inside the new window**, run:

```powershell
whoami
$env:USERDNSDOMAIN
klist
```

You should now see a domain identity rather than:

```text
the-eyrie\vagrant
```

Then obtain a Kerberos ticket:

```powershell
klist get krbtgt
klist
```

After that:

```powershell
Test-WSMan the-eyrie.sevenkingdoms.local
```

### Then test Exchange

Still in that **domain-authenticated PowerShell window**:

```powershell
$session = New-PSSession `
  -ConfigurationName Microsoft.Exchange `
  -ConnectionUri http://the-eyrie.sevenkingdoms.local/PowerShell/ `
  -Authentication Kerberos
```

If `$session` comes back without an error:

```powershell
Import-PSSession $session -DisableNameChecking
```

Then:

```powershell
Get-ExchangeServer
```

### One correction to our earlier diagnosis

The DNS problem is now essentially resolved. Your latest:

```text
Resolve-DnsName ... -DnsOnly -NoHostsFile
```

returned only:

```text
192.168.56.21
```

So **leave DNS alone now**.

The current chain is:

```text
DNS                    ✅
Secure channel         ✅
WinRM listener         ✅
WinRM connectivity     ✅
Current logon          ❌ local vagrant
Kerberos ticket        ❌ none
Exchange PowerShell    ❌ fails with 0x8009030e
```

Getting the domain Administrator session working should move us past the `0x8009030e` error.

