## Windows Server Backup Veiledning

### Navigering i Windows Server Backup

Windows Server Backup er et innebygd verktøy for sikkerhetskopiering og gjenoppretting av data. Her er en grunnleggende veiledning for å navigere og bruke det:

#### Åpne Windows Server Backup

1.  **Server Manager:** Åpne Server Manager.
2.  **Verktøy:** Klikk på "Verktøy" i øvre høyre hjørne.
3.  **Windows Server Backup:** Velg "Windows Server Backup" fra rullegardinmenyen.

#### Hovedvinduet

Windows Server Backup-konsollen har et venstre panel og et hovedpanel:

* **Venstre panel:**
    * **Lokal Server:** Viser serveren du er koblet til.
    * **Kalender:** Viser planlagte sikkerhetskopier.
* **Hovedpanel:**
    * **Oversikt:** Gir en oppsummering av sikkerhetskopieringsstatusen.
    * **Lokale sikkerhetskopier:** Lar deg konfigurere og administrere sikkerhetskopier for den lokale serveren.

#### Handlingsruten

Handlingsruten på høyre side gir tilgang til ulike oppgaver:

* **Sikkerhetskopiering en gang:** Starter en manuell sikkerhetskopiering.
* **Sikkerhetskopieringsplan...:** Oppretter en tidsplan for automatiske sikkerhetskopieringer.
* **Gjenopprett...:** Gjenoppretter data fra en tidligere sikkerhetskopi.
* **Alternativer:** Konfigurerer globale innstillinger for Windows Server Backup.

### PowerShell-koder for Windows Server Backup

Selv om Windows Server Backup har et grafisk brukergrensesnitt, kan du også bruke PowerShell til å automatisere sikkerhetskopierings- og gjenopprettingsprosesser. Her er noen nyttige PowerShell-kommandoer:

#### Installere Windows Server Backup (hvis ikke installert)

```powershell
Install-WindowsFeature -Name Windows-Server-Backup
```

#### Starte en engangssikkerhetskopiering

```powershell
# Sikkerhetskopierer hele serveren til en spesifisert plassering
Start-WBBackup -Policy {New-WBPolicy -FullSystemBackup -DestinationPath "Path_to_backup_location"} -Force
```

#### Opprette en sikkerhetskopieringsplan

```powershell
# Definerer sikkerhetskopieringspolicyen
$policy = New-WBPolicy
Add-WBVolume -Policy $policy -VolumeDriveLetter "C:" # Legg til volumet du vil sikkerhetskopiere
Set-WBSchedule -Policy $policy -ScheduleDaily -At 00:00 # Angi tidspunkt for sikkerhetskopiering

# Registrerer planen
Register-WBPolicy -Policy $policy
```

#### Gjenoppretting av data

```powershell
# Henter de siste sikkerhetskopiversjonene
$versions = Get-WBBackupSet -MachineName "ServerName"

# Starter gjenopprettingsprosessen
Start-WBRecovery -BackupSet $versions[0] -ItemToRecover "Path_to_item_to_recover" -RecoveryTarget "Path_to_recovery_location" -Force
```

Husk å erstatte plassholdere som "Path\_to\_backup\_location", "VolumeDriveLetter", "ServerName" og "Path\_to\_item\_to\_recover" med de faktiske verdiene for ditt system.
