# Windows Server Configuration Scripts

## Server Initial Setup & Role Installation
```powershell
# Install core Windows features
Install-WindowsFeature -Name AD-Domain-Services, DNS, DHCP, Hyper-V -IncludeManagementTools

# Configure static IP address (replace placeholders)
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.xxx.xxx -PrefixLength 24 -DefaultGateway 192.168.xxx.1
```

## Active Directory & DHCP Configuration
```powershell
# Promote server to Domain Controller (WARNING: Triggers reboot!)
Install-ADDSForest -DomainName "YourDomain.dm" -InstallDns -Force

# DHCP Server configuration
Add-DhcpServerInDC -DnsName "YourDomain.dm" -IPAddress 192.168.xxx.xxx
Add-DhcpServerv4Scope -Name "NetworkScope" -StartRange 192.168.xxx.100 -EndRange 192.168.xxx.200 -SubnetMask 255.255.255.0 -State Active
Restart-Service dhcpserver
```

## Sett up Vlan and Virtual switch
```powershell
# Hyper-V
New-VMSwitch -Name "VLAN10Switch" -NetAdapterName "Ethernet" -AllowManagementOS $true

```
## Organizational Unit Management
```powershell
# Create OU structure
New-ADOrganizationalUnit -Name "IT Department" -Path "DC=yourdomain,DC=dm"

# Redirect default containers
Redircmp "OU=IT Department,DC=yourdomain,DC=dm"
Redirusr "OU=IT Department,DC=yourdomain,DC=dm"
```

## Add Multiple Accounts from CSV
```powershell
# Bulk user import script
$csvPath = "C:\Path\to\users.csv"
$users = Import-Csv -Path $csvPath

foreach ($user in $users) {
    New-ADUser -GivenName $user.FirstName `
               -Surname $user.LastName `
               -Name "$($user.FirstName) $($user.LastName)" `
               -SamAccountName $user.UserName `
               -UserPrincipalName "$($user.UserName)@yourdomain.dm" `
               -AccountPassword (ConvertTo-SecureString $user.Password -AsPlainText -Force) `
               -Path $user.OU `
               -Enabled $true `
               -PassThru
}
```

**Required CSV Format** (`users.csv`):
```csv
FirstName,LastName,UserName,Password,OU
John,Doe,john.doe,SecurePass123,"OU=IT Department,DC=yourdomain,DC=dm"
Jane,Smith,jane.smith,Passw0rd!,"OU=IT Department,DC=yourdomain,DC=dm"
```

## Single User Creation
```powershell
# Create individual user
New-ADUser -Name "Test User" -SamAccountName "test.user" -UserPrincipalName "test.user@yourdomain.dm" `
  -AccountPassword (ConvertTo-SecureString "P@ssw0rd!" -AsPlainText -Force) -Enabled $true
```
## 
```
$firstname = Read-Host "Enter your name"
$surname = Read-Host "Enter your Surname"
Write-Host "hey, $firstname $surname."
```
⚠️ **Important Notes:**
- Replace all placeholders (`xxx`, `yourdomain.dm`) with actual values
- Maintain consistent domain names in UPNs and OU paths
- CSV file must use UTF-8 encoding
- Passwords must meet domain complexity requirements

🔑 **Security Recommendations:**
- Store CSV files encrypted at rest
- Delete CSV files after user creation
- Use certificate-based authentication for automated processes
- Implement account lockout policies
