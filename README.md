# 🛡️ SIEM & EDR – Wazuh sur AWS

<h1>📌 Présentation du projet</h1>

Ce projet consiste à mettre en place une infrastructure de sécurité SIEM & EDR basée sur Wazuh, déployée dans un environnement Cloud AWS.
L’objectif est de superviser des endpoints Linux et Windows, de centraliser les logs de sécurité et de détecter des comportements malveillants tels que :

- Attaques par force brute,

- Echecs d’authentification,

- Création de comptes utilisateurs,

- Elévation de privilèges.

Ce travail a été réalisé dans le cadre du module Virtualisation et Cloud Computing – Sécurité des endpoints et supervision SIEM.



<h1>🏗️ Architecture de l’environnement</h1>

L’infrastructure repose sur trois instances EC2 déployées dans le même VPC afin de simuler un réseau d’entreprise.

🔹 Instances utilisées :

+ Wazuh Server
  - OS : Ubuntu 22.04 LTS
  - Type : t3.large
  - Rôle : Manager, Indexer et Dashboard Wazuh

+ Linux Client
  - OS : Ubuntu 22.04 LTS
  - Rôle : Endpoint supervisé (agent Wazuh)

+ Windows Client
  - OS : Windows Server 2022
  - Rôle : Endpoint supervisé (agent Wazuh)



<h1>🔐 Configuration réseau et sécurité</h1>
Les Security Groups AWS ont été configurés pour autoriser uniquement les flux nécessaires :
| Port | Protocole | Description                         |
| ---- | --------- | ----------------------------------- |
| 1514 | TCP       | Communication agent → serveur Wazuh |
| 1515 | TCP       | Enrôlement automatique des agents   |
| 443  | TCP       | Accès HTTPS au Wazuh Dashboard      |
| 22   | TCP       | Accès SSH (Linux)                   |
| 3389 | TCP       | Accès RDP (Windows)                 |



<h1>⚙️ Installation de Wazuh</h1>

🖥️ Serveur Wazuh :
- Installation All-in-One via le script officiel Wazuh
- Génération automatique des identifiants administrateur
- Accès au Dashboard via HTTPS

💻 Agents Wazuh
- Linux : installation via wget et dpkg
- Windows : installation via PowerShell (Invoke-WebRequest)
- Vérification de l’état Active des agents dans le Dashboard


<h1>🔎 Scénarios de sécurité testés</h1>
<h3>🐧 Linux – Attaque SSH par force brute</h3>

- Simulation de connexions SSH avec un utilisateur inexistant
- Détection automatique par Wazuh

Alerte générée :
- Rule ID 5710 – Attempt to login using a non-existent user
- Technique MITRE ATT&CK : T1110 – Brute Force

<h3>Windows – Authentification et gestion des comptes</h3>
🔸 Échecs de connexion
- Tentatives de login avec des identifiants invalides

Alerte générée :
- Rule ID 60122 – Logon failure (Event ID 4625)

🔸 Création d’un utilisateur et élévation de privilèges
- Création d’un utilisateur local
- Ajout au groupe Administrateurs

Alertes générées :
- Rule ID 60154 – Administrators group changed
- Rule ID 60170 – Users group changed

Ces alertes permettent de détecter une tentative de persistance ou compromission interne.


<h1>🧠 Threat Hunting</h1>

Des recherches proactives ont été réalisées pour identifier des comportements suspects :

- Linux : analyse de /var/log/auth.log (tentatives SSH échouées)

- Windows : surveillance des Event ID 4625 (échecs de login)

- Windows : détection de création de comptes via logs d’audit et Sysmon
