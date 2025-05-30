# (Windows Server) a project for automating and documenting the setup of an IT infrastructure for a school. 

## All values are parameterized with variables, 
### Configuration Variables 
```powershell
# --- System Configuration ---
$newName = "Kuben"                     
$interfaceAlias = "Ethernet" 

# --- IP Configuration ---
$ip = "192.168.17.10"                  
$prefixlength = 24                     
$defaultgw = "192.168.17.1"            

# --- Domain & Active Directory ---
$domainName = "osloskolen.local"       # Domenenavn (f.eks. en skole/bedrift)
$newOU = "elever"                      # Organisasjonsenhet (OU) der brukere/PC-er skal plasseres i AD
$parentOU = "kuben"                    
$csvPath = "C:\Users\eksempel_brukere.csv"  # Filsti til en CSV-fil med brukerdata (for bulk-import)
$OUPath = "OU=elever,OU=kuben,DC=osloskolen,DC=local"

# --- Domain Controller Info ---
$DCenhet = "osloskolen"                # Første del av domenenavn (osloskolen.local)
$DCroot = "local"                      # Toppnivådomene (TLD) (osloskolen.local)
```

## Rename server
```powershell
Rename-Computer -NewName $newName -Force -PassThru | Restart-Computer -Force
```
## Configure static IP address
```powershell
New-NetIPAddress -InterfaceAlias $interfaceAlias -IPAddress $ip -PrefixLength $prefixlength -DefaultGateway $defaultgw
```

## Server Initial Setup & Role Installation
```powershell
# Install core Windows features
Install-WindowsFeature -Name AD-Domain-Services, DNS, DHCP, Hyper-V, Web-Server -IncludeManagementTools -IncludeAllSubFeatures
```

# Active Directory & DHCP Configuration
```powershell
# Promote server to Domain Controller
Install-ADDSForest -DomainName $domainName -InstallDns -Force -SafeModeAdministratorPassword $safeModePwd
```
## DHCP Server configuration with Enterprise Admin credential
```powershell
Add-DhcpServerv4Scope -Name "StudentNetwork" -StartRange 192.168.17.26 -EndRange 192.168.17.200 -SubnetMask 255.255.255.0 -State Active
Set-DhcpServerv4OptionValue -DnsServer $ip -Router $defaultgw -DnsDomain $domainName
Restart-Service dhcpserver
```

## Organizational Unit Management
```powershell
# Create OU structure
New-ADOrganizationalUnit -Name $parentOU -Path "DC=$DCenhet,DC=$DCroot"
New-ADOrganizationalUnit -Name $newOU -Path "OU=$parentOU,DC=$DCenhet,DC=$DCroot"
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

$userCreatedSuccessfully = $false

while (-not $userCreatedSuccessfully) {
    try {
        $password = Read-Host "Enter password" -AsSecureString
        
        New-ADUser -Name "$firstName $lastName" `
                   -GivenName $firstName `
                   -Surname $lastName `
                   -SamAccountName $username `
                   -UserPrincipalName "$username@$domainName" `
                   -AccountPassword $password `
                   -Path "OU=$newOU,OU=$parentOU,DC=$DCenhet,DC=$DCroot" `
                   -Enabled $true `
                   -ErrorAction Stop

        Write-Host "User '$username' created successfully."
        $userCreatedSuccessfully = $true
    }
    catch {
        if ($_.Exception.Message -like "*password*" -or $_.Exception.Message -like "*policy*" -or $_.Exception.Message -like "*policy requirements*") {
            Write-Warning "Bad password, try another one. Error: $($_.Exception.Message)"
        }
        else {
            Write-Warning "An error occurred: $($_.Exception.Message). Please try entering the password again."
        }
    }
}
```
## OPNsense brannmur-oppsett
Installasjon
Last ned OPNsense-installasjonsmediet.

Installer på en USB-enhet.

Start fra USB-enheten og fullfør installasjonen.

Grunnleggende konfigurasjon
Nettverksoppsett: Konfigurer WAN og LAN-grensesnitt.

DHCP-server: Aktiver DHCP på LAN-grensesnittet.

Sikkerhet
Brannmurregler: Tillat utgående trafikk fra LAN, blokkere innkommende trafikk til WAN.

Integrasjon mellom Windows Server og OPNsense
Fysisk tilkobling: Koble OPNsense til samme nettverk som Windows Server.

IP-adressering: Konfigurer statisk IP for Windows Server innenfor LAN-segmentet.

Dette oppsettet sikrer et robust og sikkert nettverksmiljø med både Windows Server og OPNsense.

```
```

## 🔔 Important Notes  
- **Test i isolert miljø:** Alltid test skriptene i et sandbox-miljø før produksjonsbruk  
- **Passordhåndtering:** Bytt alle standardpassord (spesielt `opnsense`/`installer` i OPNsense)  
- **IP-konflikter:** Sjekk at DHCP-scope ikke overlapper med statiske IP-adresser  
- **Versjonsavhengighet:** Skriptene er testet på Windows Server 2022 – andre versjoner kan kreve modifikasjoner  
- **Backup:** Ta full backup av OPNsense-konfigurasjon og Active Directory før endringer
- **Dokumentasjon** Dokumenter alt sammen, passord, bruker navn og ip addresser.


## 🔒 Security Notes  
**Kritiske sikkerhetstiltak for OPNsense + Windows Server:**  

1. **Active Directory:**  
   - Bruk **Least Privilege**-prinsippet for AD-brukere  
   - Implementer **Account Lockout Policy** mot brute force-angrep
  
2. **OPNsense Best Practices:**
 - Deaktiver ubrukte tjenester
 - Blokker private nettverk på WAN-grensesnittet
   
3. **Automatiske oppdateringer:**  
- Konfigurer **Auto-update** for OPNsense (`System → Firmware → Settings`)  
- Aktiver **Windows Update** for serveren via Group Policy  

4. **Logging og overvåkning:**  
- Konfigurer **Windows Event Log**-overvåkning for AD-endringer

**⚠️ Disclaimer:**  
Denne konfigurasjonen er kun en utgangsmodell. Administratoren må selv  
verifisere at oppsettet oppfyller organisasjonens sikkerhetskrav.


## ✅ Produksjonsklar-sjekkliste  
- [x] Endret standardpassord for OPNsense og AD-administrator  
- [x] Verifisert at brannmurregler ikke tillater uautorisert WAN-tilgang  
- [x] Testet DHCP-failover mellom OPNsense og Windows Server  
- [x] Konfigurert automatiske sikkerhetsoppdateringer
