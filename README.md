# LAB 6 : Analyse statique d'un APK avec MobSF

## 📌 Objectif du lab

Analyser la sécurité d'une application Android (ProjetWS) en utilisant **MobSF (Mobile Security Framework)**, un outil open-source d'analyse automatisée de sécurité mobile. L'analyse statique permet d'identifier les vulnérabilités potentielles sans exécuter l'application.

---

## 🎯 Ce que nous avons appris

| Concept | Définition |
|---------|-------------|
| **MobSF** | Mobile Security Framework - framework d'analyse automatisée de sécurité mobile |
| **Security Score** | Note de 0 à 100 évaluant la sécurité globale de l'APK |
| **Static Analysis** | Analyse du code et des ressources sans exécution de l'application |
| **Manifest Analysis** | Analyse du fichier AndroidManifest.xml (permissions, composants exportés) |
| **Hardcoded Secrets** | Informations sensibles (clés API, URLs, mots de passe) dans le code |
| **Network Security Config** | Configuration de sécurité réseau de l'application |

---

## 🛠️ Environnement et outils utilisés

| Élément | Valeur |
|---------|--------|
| **VM** | Mobexler |
| **Outil d'analyse** | MobSF v4.5.0 (via Docker) |
| **APK analysé** | ProjetWS (app-debug.apk) |
| **Package** | com.example.projetws |
| **Version** | 1.0 |

---

## 📊 Résultat global

| Indicateur | Valeur |
|------------|--------|
| **Security Score** | **41/100** |
| **Vulnérabilités HIGH** | 7 |
| **Vulnérabilités WARNING** | 5 |
| **Vulnérabilités INFO** | 2 |
| **Points SECURE** | 1 |

---

## 🐳 Configuration et lancement de MobSF

### Lancement avec Docker

```bash
# Télécharger l'image Docker MobSF
sudo docker pull opensecurity/mobile-security-framework-mobsf

# Lancer le conteneur
sudo docker run -it -p 8000:8000 opensecurity/mobile-security-framework-mobsf
```

### Accès à l'interface web

Une fois le conteneur démarré, accéder à : `http://127.0.0.1:8000`

| Information | Valeur |
|-------------|--------|
| Identifiant | `mobsf` |
| Mot de passe | `mobsf` |

## 📂 Analyse de l'APK

### Étapes réalisées

1. Upload de l'APK via l'interface web
2. Lancement de l'analyse (durée : 2-5 minutes)
3. Génération du rapport avec score de sécurité

### Informations de l'APK

| Élément | Valeur |
|---------|--------|
| App Name | Projet WS |
| Package | `com.example.projetws` |
| Version | 1.0 |
| SHA256 | `55ea8b021a8480b2c5a575e27f2a9dcb120c7d7961b69a6ad201639de76e2007` |
| Min SDK | 24 (Android 7.0) |
| Target SDK | 36 |

---

## 📊 Résultats de l'analyse

### Security Score

| Indicateur | Valeur |
|------------|--------|
| **Security Score** | **41/100** |
| HIGH | 7 |
| WARNING | 5 |
| INFO | 2 |
| SECURE | 1 |

---

## 🔍 Manifest Analysis

| # | Problème | Sévérité | Description |
|---|----------|----------|-------------|
| 1 | Min SDK 24 (Android 7.0) | INFO | Application installable sur version ancienne sans mises à jour |
| 2 | Trafic en clair (HTTP) | HIGH | `usesCleartextTraffic=true` - communications non chiffrées |
| 3 | Network Security Config | INFO | Configuration réseau personnalisée |
| 4 | Mode débogage activé | HIGH | `debuggable=true` - reverse engineering facilité |
| 5 | Backup activé | WARNING | `allowBackup=true` - extraction des données via ADB |
| 6 | Receiver exporté | WARNING | `ProfileInstallReceiver` accessible par d'autres applications |

---

## 🌐 Network Security

| # | Domaine | Sévérité | Description |
|---|---------|----------|-------------|
| 1 | `192.168.0.162`, `10.0.2.2` | HIGH | Configuration réseau autorisant le trafic en clair vers ces domaines |

---

## 🔏 Certificate Analysis

| # | Problème | Sévérité | Description |
|---|----------|----------|-------------|
| 1 | Debug certificate | HIGH | Application signée avec un certificat de debug |
| 2 | Signed Application | HIGH | Application signée (information) |

---

## 💻 Code Analysis

| # | Problème | Sévérité | Référence MASVS | Localisation |
|---|----------|----------|-----------------|--------------|
| 1 | SSL Certificate Pinning | SECURE | MSTG-NETWORK-4 | OkHttp |
| 2 | Secrets hardcodés | WARNING | MSTG-STORAGE-14 | Fichiers contenant des clés/tokens |
| 3 | Random Number Generator faible | WARNING | MSTG-CRYPTO-6 | `java.util.Random` |
| 4 | IP Address disclosure | WARNING | MSTG-CODE-2 | `192.168.0.162` |
| 5 | Logs sensibles | INFO | MSTG-STORAGE-3 | `AddEtudiant.java`, `ListEtudiant.java` |

---

## 🔑 Hardcoded Secrets

| URL | Fichier | Risque |
|-----|---------|--------|
| `http://192.168.0.162/projet/ws/createEtudiant.php` | `AddEtudiant.java` | Élevé - Données en clair |
| `http://192.168.0.162/projet/ws/loadEtudiant.php` | `ListEtudiant.java` | Élevé - Données en clair |

### Domaines vérifiés

| Domaine | Statut | Observation |
|---------|--------|-------------|
| `192.168.0.162` | OK | IP locale (serveur XAMPP) |
| `github.com` | OK | Liens documentation légitime |

---

## 🚨 Top vulnérabilités

| # | Vulnérabilité | Sévérité | Impact |
|---|---------------|----------|--------|
| 1 | Trafic en clair (HTTP) | ÉLEVÉE | Interception des données des étudiants |
| 2 | Mode débogage activé | ÉLEVÉE | Reverse engineering facilité |
| 3 | Certificat de debug | ÉLEVÉE | Signature non sécurisée |
| 4 | Backup activé | MOYENNE | Extraction de la base de données |
| 5 | Secrets hardcodés | MOYENNE | Exposition IP du serveur |

---

## ✅ Recommandations prioritaires

1. **Sécuriser les communications réseau** : Remplacer HTTP par HTTPS et désactiver `usesCleartextTraffic`
2. **Désactiver le mode débogage** : Mettre `android:debuggable="false"` en production
3. **Désactiver le backup** : Mettre `android:allowBackup="false"`
4. **Utiliser un certificat de production** : Signer l'APK avec un certificat release
5. **Nettoyer les secrets hardcodés** : Retirer les URLs HTTP du code source

---


---

## 📸 Screenshots du laboratoire

| # | Description | Capture |
|---|-------------|---------|
| 1 | MobSF - Interface d'accueil | <img width="1599" height="809" alt="T2_1" src="https://github.com/user-attachments/assets/4c71a84a-0fe2-4375-aea9-8ad955d6c26f" />|
| 2 | MobSF - Upload de l'APK | |
| 3 | MobSF - Security Score 41/100  /  App Information |<img width="1596" height="772" alt="T2_3" src="https://github.com/user-attachments/assets/f2f30f4b-292f-401a-a39f-30ad5db0d0de" />|
| 5 | MobSF - Manifest Analysis (details) | <img width="1600" height="661" alt="T4" src="https://github.com/user-attachments/assets/3146e13b-f7c5-4ae1-90aa-d87593e9626d" />|
| 6 | MobSF - Network Security / Certificate Analysis |<img width="1431" height="568" alt="T4_2" src="https://github.com/user-attachments/assets/f75f7572-6e94-4891-886e-1af2b30f8c55" />|
| 7 | MobSF - Code Analysis | <img width="1413" height="505" alt="T4_3" src="https://github.com/user-attachments/assets/14a9acd5-3f95-4b8a-826e-03f5838bff04" />|
| 8 | MobSF - Hardcoded Secrets (URLs) | <img width="1405" height="445" alt="T4_1_1" src="https://github.com/user-attachments/assets/a9d6f833-33cf-4b5b-9291-0bdaccdc956f" />|
| 9 | MobSF - Domain Malware Check |<img width="1437" height="475" alt="T4_1" src="https://github.com/user-attachments/assets/c6cd8d25-2513-449c-98b3-757ae26717a0" />|


---

## 👤 Auteur

**El Hachimi Abdelhamid**  
Date : 2026-04-20  
Cours : Sécurité des applications mobiles

---
