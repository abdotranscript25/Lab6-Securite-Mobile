# Rapport d'analyse statique - ProjetWS

## Informations générales
- **Date d'analyse :** 2026-04-20
- **Analyste :** El Hachimi Abdelhamid
- **APK analysé :** app-debug.apk
- **Package :** com.example.projetws
- **Version :** 1.0
- **SHA256 :** 55ea8b021a8480b2c5a575e27f2a9dcb120c7d7961b69a6ad201639de76e2007
- **Outils utilisés :** MobSF v4.5.0 (Docker)

## Résumé exécutif

L'analyse statique de l'application ProjetWS (Gestion d'étudiants) a révélé un score de sécurité de **41/100**.

Les principales vulnérabilités concernent :
- **Communications non chiffrées (HTTP)** : les données des étudiants circulent en clair
- **Mode débogage activé** : l'application est débuggable en production
- **Backup activé** : les données peuvent être extraites via ADB

Le niveau de risque global est évalué comme **ÉLEVÉ**.

## Vulnérabilités critiques

| # | Vulnérabilité | Sévérité | Impact |
|---|---------------|----------|--------|
| 1 | Trafic en clair (HTTP) | ÉLEVÉE | Interception des données des étudiants |
| 2 | Mode débogage activé | ÉLEVÉE | Reverse engineering facilité |
| 3 | Backup activé | MOYENNE | Extraction de la base de données |
| 4 | Certificat de debug | ÉLEVÉE | Signature non sécurisée |
| 5 | Secrets hardcodés | MOYENNE | Exposition IP du serveur |

## Recommandations prioritaires

1. **Sécuriser les communications réseau** : Remplacer HTTP par HTTPS et désactiver `usesCleartextTraffic`
2. **Désactiver le mode débogage** : Mettre `android:debuggable="false"` en production
3. **Désactiver le backup** : Mettre `android:allowBackup="false"`
4. **Utiliser un certificat de production** : Signer l'APK avec un certificat release
5. **Nettoyer les secrets hardcodés** : Retirer les URLs HTTP du code source

## Annexes

### Permissions demandées
- `android.permission.INTERNET`
- `android.permission.ACCESS_NETWORK_STATE`

### Composants exportés
- `MainActivity` (exportée)
- `ProfileInstallReceiver` (exporté)

### URLs trouvées dans le code
- `http://192.168.0.162/projet/ws/createEtudiant.php`
- `http://192.168.0.162/projet/ws/loadEtudiant.php`
