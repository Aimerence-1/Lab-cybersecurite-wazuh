# Lab de Cybersécurité — Wazuh SIEM + Kali Linux + Windows 10

> Simulation d'attaques réelles et détection en temps réel avec Wazuh SIEM

## Description

Ce lab permet de simuler une attaque cybersécurité en environnement contrôlé et d'observer sa détection en temps réel par le SIEM Wazuh. Il couvre l'installation complète d'un environnement de détection d'intrusions, la création de règles personnalisées et la simulation d'un reverse shell via Metasploit.

## Architecture
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   Kali Linux    │     │   Windows 10    │     │  Ubuntu 24.04   │
│   Attaquant     │────▶│    Victime      │────▶│  Wazuh SIEM     │
│ 192.168.74.128  │     │ 192.168.74.130  │     │ 192.168.74.131  │
└─────────────────┘     └─────────────────┘     └─────────────────┘
     Metasploit          Sysmon + Agent            Dashboard
     msfvenom             Wazuh                    Alertes


   Machine          Rôle                   OS                  IP 
 Wazuh Server     Analyseur SIEM       Ubuntu 24.04 LTS        192.168.74.131 
 Kali Linux      Attaquant             Kali Linux 2026.1       192.168.74.128 
 Windows 10       Victime              Windows 10 Pro 22H2     192.168.74.130 


## Technologies utilisées

- "Wazuh" 4.12 SIEM open source
- "Sysmon" Surveillance avancée Windows (Sysinternals)
- "Metasploit 6.4" Framework de pentest
- "msfvenom" Génération de payloads
- "VMware Workstation"  Virtualisation
- "MITRE ATT&CK"  Framework de détection



## 🚀 Prérequis

- VMware Workstation ou Player
- Minimum 16 Go de RAM
- Minimum 150 Go d'espace disque
- ISOs : Ubuntu 24.04, Kali Linux, Windows 10 22H2

## Installation rapide

### 1. Wazuh (Ubuntu)
bash
curl -sO https://packages.wazuh.com/4.12/wazuh-install.sh
sudo bash wazuh-install.sh -a

### 2. Sysmon (Windows 10 - PowerShell Admin)
powershell
mkdir C:\Sysmon
Invoke-WebRequest -Uri "https://download.sysinternals.com/files/Sysmon.zip" -OutFile "C:\Sysmon\sysmon.zip"
Expand-Archive -Path "C:\Sysmon\sysmon.zip" -DestinationPath "C:\Sysmon"
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig-export.xml" -OutFile "C:\Sysmon\sysmonconfig.xml"
C:\Sysmon\Sysmon64.exe -accepteula -i C:\Sysmon\sysmonconfig.xml

### 3. Agent Wazuh (Windows 10 - PowerShell Admin)
powershell
Invoke-WebRequest -Uri "https://packages.wazuh.com/4.x/windows/wazuh-agent-4.12.0-1.msi" -OutFile "C:\wazuh-agent.msi"
msiexec /i C:\wazuh-agent.msi /q WAZUH_MANAGER="192.168.74.131"
NET START WazuhSvc

## Règles de détection personnalisées

### Règle 100001 — Exécution depuis Downloads
xml
<rule id="100001" level="8">
  <if_group>sysmon_event1</if_group>
  <field name="win.eventdata.image" type="pcre2">(?i)\\Users\\.*\\Downloads\\.*\.exe</field>
  <description>Execution d un executable depuis Downloads</description>
  <mitre><id>T1204.002</id></mitre>
</rule>

### Règle 100002 — Connexion réseau depuis Downloads
xml
<rule id="100002" level="10">
  <if_group>sysmon_event3</if_group>
  <field name="win.eventdata.image" type="pcre2">(?i)\\Users\\.*\\Downloads\\.*\.exe</field>
  <field name="win.eventdata.initiated">true</field>
  <description>Connexion reseau sortante depuis Downloads</description>
  <mitre><id>T1071</id></mitre>
</rule>

## Techniques MITRE ATT&CK détectées

ID                Technique                           Outil                          Règle 

T1204.002        User Execution                 Malicious File| msfvenom             100001 
T1071         Application Layer Protocol (C2)      Meterpreter                       100002 
T1059        Command and Scripting Interpreter     PowerShell                        Sysmon 
T1105        Ingress Tool Transfer                  HTTP                             Sysmon 


## Résultats obtenus

- ✅ "162" événements Sysmon détectés
- ✅ "28" alertes de niveau 12 ou plus
- ✅ Règle "100001" déclenchée (T1204.002 - Malicious File)
- ✅ Règle "100002" déclenchée (T1071 - Application Layer Protocol)
- ✅ Session Meterpreter détectée en temps réel

## Avertissement éthique

Ce lab est destiné **uniquement à des fins éducatives** dans un environnement isolé et contrôlé. Ne jamais utiliser ces techniques sur des systèmes sans autorisation explicite. L'utilisation non autorisée est illégale.

##  Auteur

**Fèmi Aimérence BOKO**  
Étudiante en Cybersécurité — École des Métiers du Numérique 2026
