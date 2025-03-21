# Sette opp server
```
laste ned server22 --> velge desktop experience --> velge custom --> laste ned også ta ut usb-en. 
```

# Installere alt, og sette opp statisk ip med Powershell
```
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
 
Install-WindowsFeature -Name DNS -IncludeManagementTools
 
Install-WindowsFeature -Name DHCP -IncludeManagementTools
 
Install-WindowsFeature -Name Hyper-V -IncludeManagementTools
 
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.xxx.xxx -PrefixLength 24 -DefaultGateway 192.168.xxx.1
```
# Legge til en ny ADDSForest, og Sette opp en DHCP Scope
```
Install-ADDSForest -DomainName "YourDomain.dm" -InstallDns -Force
 
Add-DhcpServerInDC -DnsName "YourDomain.dm" -IPAddress 192.168.xxx.xxx
 
Add-DhcpServerv4Scope -Name "DittScopeNavn" -StartRange 192.168.xxx.xxx -EndRange 192.168.xxx.xxx -SubnetMask 255.255.255.0 -State Active
 
Restart-Service dhcpserver
 
Get-DhcpServerv4Scope
```
# Sette opp ett OU med brukere

```
New-ADOrganizationalUnit -Name "IT Department" -Path "DC=yourdomain,DC=dm"
 
Redircmp "OU=IT Department,DC=yourdomain,DC=dm"
 
Get-ADOrganizationalUnit -Filter * | Select Name, DistinguishedName
 
Redirusr "OU=IT Department,DC=yourdomain,DC=dm"
 
New-ADUser -Name "Test User" -SamAccountName "test.user" -UserPrincipalName "test.user@yourdomain.dm" `
-AccountPassword (ConvertTo-SecureString "P@ssw0rd!" -AsPlainText -Force) -Enabled $true

Get-ADUser -Identity "test.user" | Select DistinguishedName
```

# Legge til mange brukere via Powershell og CSV fil

```
Definer stien til CSV-filen
$csvPath = "C:\Users\Administrator\Documents\xxx.csv"
 
Les CSV-filen

$users = Import-Csv -Path $csvPath
foreach ($user in $users) {
    # Definer brukerkontoegenskapene
    $firstName = $user.FirstName
    $lastName = $user.LastName
    $userName = $user.UserName
    $password = $user.Password
    $ou = $user.OU
    
    Opprett brukerkontoen
    New-ADUser -GivenName $firstName 
               -Surname $lastName 
               -Name "$firstName $lastName" 
               -SamAccountName $userName 
               -UserPrincipalname "$userName@example.com" 
               -AccountPassword (ConvertTo-SecureString $password -AsPlainText -Force) 
               -Path $ou 
               -Enabled $true 
               -PassThru

```
