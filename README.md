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

## Stap 1 – VMs aanmaken
- Installeer VirtualBox 7.x.  
- Maak VMs:
  - **Windows 10** — 1 CPU, 4 GB, 50 GB  
  - **Windows Server 2022** — 1 CPU, 4 GB, 50 GB  
  - **Ubuntu (Splunk)** — 2 CPU, 8 GB, 100 GB  
  - **Kali Linux** — OVA import

<img width="847" height="929" alt="image" src="https://github.com/user-attachments/assets/b5ce187e-fc63-45b2-89ab-610b20506374" />



## Stap 2 – Splunk installeren
- Installeer Ubuntu Server 22.04 en geef statisch IP `192.168.10.10`.  
- Installeer Splunk (`.deb`) en open `http://192.168.10.10:8000`.  
- Maak index `endpoint` aan en activeer receiving op poort `9997`.  

<img width="894" height="398" alt="image" src="https://github.com/user-attachments/assets/1774430d-17b0-4b79-94a0-35b948b7da9e" />


## Stap 3 – Sysmon & Forwarder
- Installeer Sysmon met `sysmon64.exe -i sysmonconfig.xml`.  
- Installeer Splunk Universal Forwarder op server en target.  
- Plaats `inputs.conf` in `C:\Program Files\SplunkUniversalForwarder\etc\system\local\`:
- 
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
Herstart Splunk Forwarder.

<img width="801" height="557" alt="image" src="https://github.com/user-attachments/assets/50b26d28-24a5-45a8-98ee-c009da52f9d4" />

## Stap 4 – Active Directory
1: Stel DC IP in op 192.168.10.7.
2: Installeer AD DS en promoteer naar domain controller mydfir.local.
3: Maak OUs/users: IT\JSmith, HR\TSmith.
4: Voeg in Windows 10 aan domein.

<img width="897" height="731" alt="image" src="https://github.com/user-attachments/assets/1e60876b-26bc-4545-9f58-4662e88008c6" />


## Stap 5 – RDP brute-force (Kali)
1. Stel Kali IP in: `192.168.10.250`.  
2. Installeer Hydra:
   
sudo apt install hydra -y

Maak een wachtwoordlijst passwords.txt (bijv. 20 regels of gebruik een subset van rockyou.txt):

voorbeeld: kopieer eerste 20 regels uit rockyou
gunzip -c /usr/share/wordlists/rockyou.txt.gz | head -n 20 > passwords.txt

Voer Hydra uit voor 1 gebruiker (TSmith) en log resultaten:

Hydra -l TSmith -P passwords.txt rdp://192.168.10.11 -t 4 -V -f -o hydra_rdp_results.tx

<img width="716" height="825" alt="image" src="https://github.com/user-attachments/assets/f7df66ab-f347-4c38-aa08-09306a5f3d3e" />

<img width="1006" height="743" alt="image" src="https://github.com/user-attachments/assets/07201b51-72df-45bf-93e6-4df5b7194cbd" />  hier zie je een failed login attempt

<img width="1018" height="758" alt="image" src="https://github.com/user-attachments/assets/c40df8e6-cf37-4971-822f-5107867489a1" /> hier zie je een succesvolle login attempt door de Brute Force attack


## Stap 6 – Atomic Red Team
In PowerShell (Admin):

powershell

Set-ExecutionPolicy Bypass -Scope CurrentUser
IEX (IWR "https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/install.ps1" -UseBasicParsing)

Install-AtomicRedTeam 
Invoke-AtomicTest T1136.001

<img width="945" height="778" alt="image" src="https://github.com/user-attachments/assets/448304fc-f2f1-4819-bbb6-49f9faeacb36" />


Controleer in Splunk:

index=endpoint "new local user"

Ik vond niks en dat betekent dat er een gap zit met de huidige instellingen. Dat vind ik nou het leuke aan ART, je kan de gaps vinden in het monitoren. 


## Samenvatting

Werkende AD + Splunk lab (4 VMs).

Sysmon → Splunk ingest (index endpoint).

RDP brute-force en Atomic-tests uitgevoerd en geanalyseerd.
