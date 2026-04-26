# Mobile Security Lab: Android Rooting & Integrity Analysis

L'objectif est d'analyser les mécanismes de rooting, l'impact sur l'isolation des données (sandboxing) et les protections d'intégrité modernes sur Android.

## 📌 1. Fiche Périmètre (Périmètre du Lab)
* **Auteur :** Noha
* **Support :** Émulateur Android Studio (AVD) - Google Pixel 6
* **Version OS :** Android 13.0 (Tiramisu) / API 33
* **Objectif :** Comprendre le processus de rooting et évaluer les impacts sur la confidentialité des données.
* **Données :** Utilisation exclusive de données fictives.
* **Réseau :** Environnement de test local et isolé.

---

## 🛠️ 2. Configuration & Méthodologie

### Phase A : Tentative de Rooting & Accès Système
Pour tester l'intégrité, l'émulateur a été lancé avec la partition système déverrouillée.

```bash
# Lancement de l'AVD en mode écriture
emulator -avd Pixel_6_API_33 -writable-system

# Tentative d'élévation de privilèges
adb root
adb remount
```

> **Observation :** Bien que `adb root` soit fonctionnel, la commande `remount` échoue en raison du mécanisme **dm-verity** actif sur API 33.

![Erreur Remount](./labsec2/1.png)

---

### Phase B : Audit des Propriétés Système
Utilisation de commandes de diagnostic pour vérifier l'état de sécurité réel de l'appareil émulé.

```bash
adb shell id
adb shell getprop ro.boot.veritymode
adb shell getprop ro.boot.vbmeta.device_state
```

![Analyse des Propriétés](./labsec2/2.png)

**Analyse des résultats :**
* **UID :** `0(root)` -> Privilèges maximum obtenus sur le démon ADB.
* **VerityMode :** `enforcing` -> La protection d'intégrité logicielle est active.
* **Impact :** L'accès root permet de briser le sandboxing applicatif (accès à `/data/data/`), validant ainsi un risque majeur pour la confidentialité des données.

---

## 🛡️ 3. Référentiel de Sécurité (OWASP MASVS)

Le lab s'appuie sur le standard **MASVS** (Mobile Application Security Verification Standard) :
1.  **STORAGE-1 :** Vérification du stockage des données sensibles (clés API, tokens). Le root permet de vérifier si ces données sont stockées en clair dans `shared_prefs`.
2.  **NETWORK-1 :** Analyse du trafic. Le rooting permet d'injecter des certificats pour auditer les flux TLS.

**Tests suggérés (MASTG) :**
* Examen des fichiers XML dans `/data/data/[package_name]/shared_prefs/`.
* Analyse des fuites d'informations via `adb logcat`.

---

## ⚠️ 4. Risques & Points de Vigilance
* **Intégrité non garantie :** Les modifications root peuvent biaiser les résultats de sécurité.
* **Surface d'attaque accrue :** Un appareil rooté est plus exposé aux malwares si le périmètre n'est pas maîtrisé.
* **Mélange de données :** Risque de fuite si des comptes personnels sont utilisés sur l'appareil de test.

---

## 🧹 5. Procédure de Reset (Hygiène Numérique)
La remise à zéro est obligatoire pour éviter toute contamination des tests futurs.

* **Action :** Wipe Data via Android Studio ou commande :
    ```bash
    emulator -avd Pixel_6_API_33 -wipe-data
    ```

