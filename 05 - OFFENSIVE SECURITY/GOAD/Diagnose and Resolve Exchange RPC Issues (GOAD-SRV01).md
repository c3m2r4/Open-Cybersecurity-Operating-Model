# Diagnose and Resolve Exchange RPC Issues (GOAD-SRV01)

This plan outlines the steps to resolve the Exchange RPC Endpoint Mapper (1753) errors and `MapiExceptionNetworkError` occurring on `the-eyrie` (GOAD-SRV01).

## Findings
Our diagnostic scripts executed against `the-eyrie` revealed the root cause of the RPC and mailbox connection failures:
1. **Dual-Homed DNS Pollution**: `the-eyrie` has two network interfaces (`Ethernet1` at 192.168.56.21 and `Ethernet0` at 192.168.57.163). Because `Ethernet0` receives its IP via DHCP, Windows automatically registered this NAT IP in the Active Directory DNS zone. Consequently, `the-eyrie.sevenkingdoms.local` resolves to *both* IPs. Client requests and internal Exchange RPC calls are randomly routed to the NAT IP, which drops the traffic and causes the RPC 1753 failures.
2. **Service Outage**: The `MSExchangeTransport` service is currently in a `Stopped` state.

## User Review Required
> [!IMPORTANT]
> To execute this plan, I will run PowerShell commands on `GOAD-SRV01` and `GOAD-DC01` via `vmrun`. Please review the proposed changes below and click **Proceed** if you approve.

## Proposed Changes

### 1. Fix Interface DNS Registration (GOAD-SRV01)
Disable DNS registration for the NAT interface so it stops polluting the Active Directory DNS zone.
- Run on `the-eyrie`:
  ```powershell
  Set-DnsClient -InterfaceAlias "Ethernet0" -RegisterThisConnectionsAddress $false
  ipconfig /flushdns
  ```

### 2. Clean Up Active Directory DNS (GOAD-DC01)
Remove the invalid NAT IP from the AD DNS zone so all clients/services exclusively route to the internal `192.168.56.21` IP.
- Run on `kingslanding` (GOAD-DC01):
  ```powershell
  Remove-DnsServerResourceRecord -ZoneName "sevenkingdoms.local" -RRType "A" -Name "the-eyrie" -RecordData "192.168.57.163" -Force
  ```

### 3. Restart Exchange Services (GOAD-SRV01)
Restart core Exchange topology and RPC services to clear out old bindings, and start the Transport service.
- Run on `the-eyrie`:
  ```powershell
  Start-Service -Name "MSExchangeTransport"
  Restart-Service -Name "MSExchangeADTopology" -Force
  Restart-Service -Name "MSExchangeRPC" -Force
  ```

## Verification Plan

### Automated Tests
Once the remediation is complete, I will execute a final validation script as a Domain Admin (`sevenkingdoms\vagrant`) to ensure:
- `Resolve-DnsName the-eyrie.sevenkingdoms.local` returns ONLY `192.168.56.21`.
- `Test-NetConnection -ComputerName the-eyrie -Port 135` succeeds.
- `Get-MailboxDatabaseCopyStatus` shows the database is `Mounted` and healthy.
- `Test-MapiConnectivity` succeeds for the Exchange server.
