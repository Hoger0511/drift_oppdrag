# Windows Server Configuration Scripts
# All values are parameterized with variables

## Configuration Variables 
```powershell
$newName = "Kuben"
$interfaceAlias = "Ethernet"
$ip = "192.168.17.10"
$prefixlength = "24"
$defaultgw = "192.168.17.1"
$domainName = "osloskolen.local"  # osloskolen.local
$newOU = "elever"
$parentOU = "kuben"
$csvPath = "C:\Path\to\eksempel_brukere.csv"
```
## Server Initial Setup & Role Installation
```powershell
# Install core Windows features
Install-WindowsFeature -Name AD-Domain-Services, DNS, DHCP, Hyper-V, Web-Server -IncludeManagementTools -IncludeAllSubFeatures

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
Add-DhcpServerv4Scope -Name "StudentNetwork" -StartRange 192.168.17.26 -EndRange 192.168.17.200 -SubnetMask 255.255.255.0 -State Active
Set-DhcpServerv4OptionValue -DnsServer 192.168.17.10 -Router 192.168.1.1 -DnsDomain "osloskolen.local"
Restart-Service dhcpserver
```

## Organizational Unit Management
```powershell
# Create OU structure
New-ADOrganizationalUnit -Name "$parentOU" -Path "DC=osloskolen,DC=local"
New-ADOrganizationalUnit -Name "$newOU" -Path "OU=$parentOU,DC=osloskolen,DC=local"

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

## OPNsense brannmur-oppsett
```
Installasjon
Last ned OPNsense-installasjonsmediet.

Install på en USB-enhet.

Start fra USB-enheten og fullfør installasjonen.

Grunnleggende konfigurasjon
Nettverksoppsett: Konfigurer WAN og LAN-grensesnitt.

DHCP-server: Aktiver DHCP på LAN-grensesnittet.

Sikkerhet
Brannmurregler: Tillat utgående trafikk fra LAN, blokkere innkommende trafikk til WAN.

Integrasjon mellom Windows Server og OPNsense
Fysisk tilkobling: Koble OPNsense til samme nettverk som Windows Server.

IP-adressering: Konfigurer statisk IP for Windows Server innenfor LAN-segmentet.

DNS og routing: Konfigurer DNS og statiske ruter i OPNsense.

Dette oppsettet sikrer et robust og sikkert nettverksmiljø med både Windows Server og OPNsense.

```
## 🔔 Important Notes  
- **Test i isolert miljø:** Alltid test skriptene i et sandbox-miljø før produksjonsbruk  
- **Passordhåndtering:** Bytt alle standardpassord (spesielt `opnsense`/`installer` i OPNsense)  
- **IP-konflikter:** Sjekk at DHCP-scope ikke overlapper med statiske IP-adresser  
- **Versjonsavhengighet:** Skriptene er testet på Windows Server 2022 – andre versjoner kan kreve modifikasjoner  
- **Backup:** Ta full backup av OPNsense-konfigurasjon og Active Directory før endringer

## 🔒 Security Notes  
**Kritiske sikkerhetstiltak for OPNsense + Windows Server:**  
1. **Brannmurregler:**  
   - Tillat **kun** nødvendige porter (f.eks. RDP:3389, DNS:53)  
   - Blokker ICMP-ping fra WAN-grensesnittet  
   - Implementer "Default Deny"-policy for innkommende trafikk  

2. **Active Directory:**  
   - Bruk **Least Privilege**-prinsippet for AD-brukere  
   - Implementer **Account Lockout Policy** mot brute force-angrep
  
3. **OPNsense Best Practices:**

 - Deaktiver ubrukte tjenester
 - Blokker private nettverk på WAN-grensesnittet
   
4. **Automatiske oppdateringer:**  
- Konfigurer **Auto-update** for OPNsense (`System → Firmware → Settings`)  
- Aktiver **Windows Update** for serveren via Group Policy  

5. **Logging og overvåkning:**  
- Konfigurer **Windows Event Log**-overvåkning for AD-endringer

**⚠️ Disclaimer:**  
Denne konfigurasjonen er kun en utgangsmodell. Administratoren må selv  
verifisere at oppsettet oppfyller organisasjonens sikkerhetskrav.

## ✅ Produksjonsklar-sjekkliste  
- [x] Endret standardpassord for OPNsense og AD-administrator  
- [x] Verifisert at brannmurregler ikke tillater uautorisert WAN-tilgang  
- [x] Testet DHCP-failover mellom OPNsense og Windows Server  
- [x] Konfigurert automatiske sikkerhetsoppdateringer



