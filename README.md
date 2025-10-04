# 🔍 Detection Lab – Active Directory & Splunk

## 🎯 Doelstelling  
Het doel van dit project is het opzetten van een gecontroleerde **Detection Lab-omgeving** waarin cyberaanvallen kunnen worden gesimuleerd en gedetecteerd.  
Binnen deze omgeving worden logbestanden verzameld, geanalyseerd en gebruikt om realistische aanvalsscenario’s te onderzoeken met behulp van **Splunk** als SIEM-systeem.  

De labomgeving bestaat uit een **Active Directory-domein**, een **Splunk-server** voor loganalyse, en een **Kali Linux-machine** waarmee aanvallen worden uitgevoerd.  
Dit project biedt praktische ervaring in zowel **Blue Team (defensieve)** als **Red Team (aanvallende)** concepten.

---

## 🧠 Vaardigheden die zijn ontwikkeld
- Inzicht in **SIEM-concepten** en het analyseren van Windows-telemetrie in Splunk.  
- Beheersing van **Active Directory-beheer** (gebruikers, OUs, domeinen, authenticatie).  
- Leren **detecteren van brute-force aanvallen** via RDP en het correleren van logs.  
- Toepassen van **Sysmon** voor uitgebreide systeemlogging.  
- Uitvoeren van **Atomic Red Team tests** gekoppeld aan MITRE ATT&CK-technieken.  
- Configureren van **virtuele netwerken** in VirtualBox (NAT-netwerk, IP-planning).  
- Ontwikkelen van een onderzoeksgerichte mindset en analytische denkvaardigheden in cybersecurity.  

---

## 🛠️ Gebruikte tools en technologieën
- **VirtualBox 7.x** – virtualisatieplatform  
- **Windows Server 2022** – Active Directory Domain Controller  
- **Windows 10** – doelwitmachine met Sysmon en Splunk Universal Forwarder  
- **Ubuntu Server 22.04** – Splunk Enterprise server  
- **Splunk Enterprise** – SIEM voor logverzameling en analyse  
- **Sysmon (met Olaf Hartong config)** – geavanceerde Windows logging  
- **Kali Linux** – aanvalsmachine (Crowbar, rockyou.txt)  
- **Atomic Red Team** – simulatie van MITRE ATT&CK-technieken  

---

## 🧩 Labarchitectuur

| Rol | Hostnaam | OS | IP-adres | Omschrijving |
|------|-----------|----------|---------------|--------------|
| Splunk Server | `splunk` | Ubuntu Server 22.04 | `192.168.10.10/24` | Bevat Splunk Web (`:8000`) en ontvangt logs via poort `9997` |
| Domain Controller | `ADDC01` | Windows Server 2022 | `192.168.10.7/24` | Active Directory + DNS |
| Target Workstation | `TARGET-PC` | Windows 10 | `192.168.10.11/24` | Doelwitmachine met Sysmon en Splunk Universal Forwarder |
| Attacker | `kali` | Kali Linux | `192.168.10.250/24` | Aanvalsmachine (RDP-bruteforce en Atomic Red Team) |

💡 **VirtualBox NAT-netwerk:**  
Aangemaakt als `AD-Project` met IPv4-prefix `192.168.10.0/24` (DHCP ingeschakeld).

**Ref 1: Netwerkdiagram (plaats screenshot hier)**  

---

## ⚙️ Voorwaarden
- VirtualBox geïnstalleerd (SHA-256 checksum gecontroleerd).  
- Beschikbare ISO’s:
  - Windows 10 (Media Creation Tool)
  - Windows Server 2022 (Evaluation)
  - Ubuntu Server 22.04
  - Kali Linux (OVA aanbevolen)  
- Minimaal **16 GB RAM** aanbevolen (je kunt DC/Target later terugzetten naar 2 GB).

---

## 🚀 Projectstappen

### **Stap 1 – Opzetten van de virtuele machines**
1. Installeer VirtualBox 7.x.  
2. Maak de volgende VMs aan:  
   - **Windows 10** – 1 CPU, 4 GB RAM, 50 GB disk  
   - **Windows Server 2022** – 1 CPU, 4 GB RAM, 50 GB disk  
   - **Ubuntu Server (Splunk)** – 2 CPU’s, 8 GB RAM, 100 GB disk  
   - **Kali Linux** – geïmporteerde OVA-file  

📸 *Ref 2: Screenshots van de VM-configuratie*  

---

### **Stap 2 – Installatie en configuratie van Splunk**
1. Installeer **Ubuntu Server 22.04**.  
2. Geef een statisch IP-adres (`192.168.10.10`) in het netplan-bestand.  
3. Installeer Splunk Enterprise (`.deb`-pakket).  
4. Start Splunk en log in op `http://192.168.10.10:8000`.  
5. Maak een nieuwe **index** aan met de naam `endpoint`.  
6. Activeer poort `9997` onder *Settings → Forwarding & Receiving → Configure Receiving*.  

📸 *Ref 3: Splunk Web interface met index configuratie*  

---

### **Stap 3 – Installatie Sysmon en Universal Forwarder**
1. Download **Sysmon** van Sysinternals en de **Olaf Hartong config** (`sysmonconfig.xml`).  
2. Installeer Sysmon:  
   ```bash
   sysmon64.exe -i sysmonconfig.xml
Installeer Splunk Universal Forwarder op zowel de server als het target.

Voeg een inputs.conf-bestand toe in:

perl
Code kopiëren
C:\Program Files\SplunkUniversalForwarder\etc\system\local\
met de volgende inhoud:

ini
Code kopiëren
[default]
host = TARGET-PC

[WinEventLog:Application]
index = endpoint
[WinEventLog:Security]
index = endpoint
[WinEventLog:System]
index = endpoint
[WinEventLog:Microsoft-Windows-Sysmon/Operational]
index = endpoint
Start de Splunk Forwarder Service opnieuw.

📸 Ref 4: Voorbeeld van logs zichtbaar in Splunk (index=endpoint)

Stap 4 – Configuratie van Active Directory
Stel op de Windows Server een statisch IP-adres in (192.168.10.7).

Installeer Active Directory Domain Services (AD DS) via Server Manager.

Promoot de server tot Domain Controller met domein:

lua
Code kopiëren
mydfir.local
Maak in Active Directory Users and Computers:

OU IT → Gebruiker Jenny Smith (JSmith)

OU HR → Gebruiker Terry Smith (TSmith)

Voeg de Windows 10-client toe aan het domein mydfir.local.

📸 Ref 5: Domeincontroller met gebruikers en OUs zichtbaar

Stap 5 – Aanvalssimulatie met Kali Linux
Stel een statisch IP-adres in (192.168.10.250).

Installeer Crowbar:

bash
Code kopiëren
sudo apt install crowbar -y
Maak een bestand passwords.txt met 20 wachtwoorden + het echte wachtwoord van TSmith.

Voer de brute-force uit:

bash
Code kopiëren
crowbar -b rdp -u TSmith -C passwords.txt -s 192.168.10.11/32
Bekijk in Splunk:

spl
Code kopiëren
index=endpoint TSmith
→ Controleer event IDs:

4625 → Mislukte logins

4624 → Geslaagde login

📸 Ref 6: Splunk resultaten – EventID 4625 & 4624 zichtbaar

Stap 6 – Atomic Red Team (ATT&CK-simulatie)
Open PowerShell als Administrator:

powershell
Code kopiëren
Set-ExecutionPolicy Bypass -Scope CurrentUser
Voeg een antivirus-uitsluiting toe voor C:\.

Installeer Atomic Red Team:

powershell
Code kopiëren
IEX (IWR "https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/install.ps1" -UseBasicParsing)
Install-AtomicRedTeam
Voer een test uit (bijv. lokaal account aanmaken):

powershell
Code kopiëren
Invoke-AtomicTest T1136.001
Controleer Splunk op detectie:

spl
Code kopiëren
index=endpoint "new local user"
📸 Ref 7: Voorbeeld Atomic Red Team testresultaten in Splunk

🧾 Samenvatting
✅ Voltooide onderdelen:

Volledige labomgeving met 4 virtuele machines

Actieve AD-domeinstructuur

Splunk ingest met Sysmon + Windows Event Logs

Aanvalsdetectie via RDP brute force

Atomic Red Team getest tegen MITRE ATT&CK

💡 Belangrijkste leerpunt:
Door de aanval uit te voeren en de logs in Splunk te analyseren, leer je niet alleen wat er gebeurt, maar ook hoe en waarom — de essentie van moderne cybersecurity-detectie.
