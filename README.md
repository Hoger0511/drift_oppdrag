# Windows Server Configuration Scripts
# All values are parameterized with variables

## Configuration Variables 
```powershell
$oldName = "Skole"
$newName = "Kuben"
$interfaceAlias = "Ethernet"
$ip = "192.168.1.100"
$prefixLength = 24
$defaultgw = "192.168.1.1"
$domainName = "$DCenhet.$DCroot"  # osloskolen.local
$newOU = "elever"
$parentOU = "kuben"
$csvPath = "C:\Path\to\users.csv"
```
## Server Initial Setup & Role Installation
```powershell
# Install core Windows features
Install-WindowsFeature -Name AD-Domain-Services, DNS, DHCP -IncludeManagementTools

# Configure static IP address
New-NetIPAddress -InterfaceAlias "$interfaceAlias" -IPAddress $ip -PrefixLength $prefixLength -DefaultGateway $defaultgw

# Rename server
Rename-Computer -NewName $newName -Force -PassThru | Restart-Computer -Force
```
## Active Directory & DHCP Configuration
```powershell
# Promote server to Domain Controller
Install-ADDSForest -DomainName $domainName -InstallDns -Force

# DHCP Server configuration
Add-DhcpServerInDC -DnsName $domainName -IPAddress $ip
Add-DhcpServerv4Scope -Name "StudentNetwork" -StartRange 192.168.1.100 -EndRange 192.168.1.200 -SubnetMask 255.255.255.0 -State Active
Restart-Service dhcpserver
```

## Organizational Unit Management
```powershell
# Create OU structure
New-ADOrganizationalUnit -Name "$newOU" -Path "OU=$parentOU,DC=$DCenhet,DC=$DCroot"

# Redirect default containers
Set-ADDomain -Identity $domainName -DefaultUserContainer "OU=$newOU,OU=$parentOU,DC=$DCenhet,DC=$DCroot"
```
## Add Multiple Accounts from CSV
```powershell
# Bulk user import script
$users = Import-Csv -Path $csvPath

foreach ($user in $users) {
    New-ADUser -GivenName $user.FirstName `
               -Surname $user.LastName `
               -Name "$($user.FirstName) $($user.LastName)" `
               -SamAccountName $user.UserName `
               -UserPrincipalName "$($user.UserName)@$domainName" `
               -AccountPassword (ConvertTo-SecureString $user.Password -AsPlainText -Force) `
               -Path "OU=$newOU,OU=$parentOU,DC=$DCenhet,DC=$DCroot" `
               -Enabled $true `
               -PassThru
}
```
## Single User Creation
```powershell
# Create individual user
New-ADUser -Name "Test User" -SamAccountName "test.user" `
  -UserPrincipalName "test.user@$domainName" `
  -AccountPassword (ConvertTo-SecureString "P@ssw0rd!" -AsPlainText -Force) `
  -Path "OU=$newOU,OU=$parentOU,DC=$DCenhet,DC=$DCroot" `
  -Enabled $true
```
## Interactive User Creation
```powershell
$firstName = Read-Host "Enter user's first name"
$lastName = Read-Host "Enter user's last name"
$username = Read-Host "Enter username"
$password = Read-Host "Enter temporary password" -AsSecureString

New-ADUser -Name "$firstName $lastName" `
           -GivenName $firstName `
           -Surname $lastName `
           -SamAccountName $username `
           -UserPrincipalName "$username@$domainName" `
           -AccountPassword $password `
           -Path "OU=$newOU,OU=$parentOU,DC=$DCenhet,DC=$DCroot" `
           -Enabled $true
```
⚠️ Important Notes:

All DC references use: DC=DCenhet,DC=DCenhet,DC=DCroot
OU structure follows: OU=newOU,OU=newOU,OU=parentOU
Default IP scheme: ip/ip/prefixLength
Domain controller identity: $identitet

🔑 Security Recommendations:

Rotate DHCP scope credentials regularly
Audit OU=$newOU permissions quarterly
Use JEA for AD management tasks
Enable LAPS for local admin passwords
