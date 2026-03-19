# MobSF_Analyse_dynamique

#  Lab — Analyse de Sécurité Mobile Android
### MobSF + DIVA : Analyse Statique & Dynamique

##  Objectifs

- Configurer un émulateur Android rootable **sans Google Play Store**
- Déployer **MobSF** via Docker connecté à l'émulateur
- Réaliser une **analyse statique** (manifest, permissions, secrets hardcodés)
- Conduire une **analyse dynamique** (Frida, interception SSL, logs runtime)
- Identifier des vulnérabilités OWASP Mobile sur l'application **DIVA**

##  Étape 1 — Création de l'émulateur AVD 

1. Ouvrez **Android Studio** → `Tools` → `AVD Manager` → `Create Virtual Device`
2. Choisissez un modèle : **Pixel 5** ou **Pixel 6**
3. Dans **System Image** :
   - Sélectionnez **Google APIs** — API **30** — architecture **x86_64**
   -  **SANS** la mention « Google Play »
   - Cliquez `Download` si nécessaire → `Next` → `Finish`
4. Nommez l'AVD : `MobSF_DIVA_API_30`

<img width="1339" height="976" alt="image" src="https://github.com/user-attachments/assets/46884199-c93d-4b4c-af2b-518a15d46e5c" />


> ** Important :** L'image ne doit PAS contenir le Google Play Store. C'est une condition indispensable pour que MobSF puisse rooter l'émulateur.


##  Étape 2 — Cloner MobSF 

```bash
git clone https://github.com/MobSF/Mobile-Security-Framework-MobSF.git
cd Mobile-Security-Framework-MobSF
```

<img width="896" height="323" alt="image" src="https://github.com/user-attachments/assets/6deb723d-6c3e-41b7-9b0e-f2867ac46b93" />


##  Étape 3 — Lancer l'émulateur avec le script MobSF 

**Windows (PowerShell) :**
```powershell
scripts\start_avd.ps1 MobSF_DIVA_API_30
```

<img width="1112" height="344" alt="image" src="https://github.com/user-attachments/assets/2848af1c-5c74-4e5e-9799-677cd669da44" />


### Vérification ADB

Dans un nouveau terminal :
```bash
adb devices
```

<img width="777" height="119" alt="image" src="https://github.com/user-attachments/assets/6200a8ed-ac1f-4501-88ee-abe03de74c91" />


##  Étape 4 — Déployer MobSF via Docker 

L'émulateur doit rester en fonctionnement. Dans un nouveau terminal :

```bash
docker pull opensecurity/mobile-security-framework-mobsf:latest
```

<img width="1129" height="493" alt="image" src="https://github.com/user-attachments/assets/03e46145-af9e-44f4-abd9-07c3e7afbeba" />


Puis lancez le conteneur :

```bash
docker run -it --rm \
  -p 8000:8000 \
  -e MOBSF_ANALYZER_IDENTIFIER=emulator-5554 \
  opensecurity/mobile-security-framework-mobsf:latest
```
<img width="1814" height="552" alt="image" src="https://github.com/user-attachments/assets/8ee8220b-730b-4402-abf0-a8980f851da9" />

Ouvrir le navigateur : **http://127.0.0.1:8000**

 Login : `mobsf` 
 
 Mot de passe:| `mobsf` 
 
 
<img width="1912" height="733" alt="image" src="https://github.com/user-attachments/assets/c8182ecb-1c4e-45e0-ac42-0f3e6929f80d" />


> ** Ordre obligatoire :** L'émulateur doit être lancé **avant** MobSF, sinon la connexion ADB échoue.



##  Étape 5 — Télécharger l'APK DIVA

**DIVA** (Damn Insecure and Vulnerable App) est l'application de test de référence : 13 challenges volontairement vulnérables.

- Site officiel : http://www.payatu.com/damn-insecure-and-vulnerable-app/
- GitHub : https://github.com/payatu/diva-android

Téléchargez `diva-beta.apk` et gardez-le sur votre bureau.



##  Étape 6 — Analyse Statique + Dynamique

### Analyse Statique

1. Dans MobSF → **Upload & Analyze** → sélectionnez `diva.apk`
2. Attendez la fin de l'analyse
3. 
<img width="1428" height="741" alt="image" src="https://github.com/user-attachments/assets/bd61e975-9a40-416b-a3a9-8583a826d829" />

Le rapport inclut :
- Manifest Android (permissions, composants exposés)
- Décompilation Java / Smali
- Détection de secrets hardcodés
- Score de sécurité global

<img width="1913" height="844" alt="image" src="https://github.com/user-attachments/assets/69e731ac-2da3-4697-8df0-8aa2b1955de8" />

### Analyse Dynamique

1. Dans le rapport → cliquez **Dynamic Analysis**
2. MobSF installe DIVA, déploie Frida Server et configure le proxy HTTPS

   <img width="1896" height="892" alt="image" src="https://github.com/user-attachments/assets/cb31edd9-88c0-4b65-b91e-7cf57ee6a425" />

   <img width="1909" height="884" alt="image" src="https://github.com/user-attachments/assets/77477f20-0079-4481-b1fa-88e2c0970ca5" />


3. Sur l'émulateur, lancez **DIVA** et explorez les challenges

   <img width="1862" height="950" alt="image" src="https://github.com/user-attachments/assets/cd4d65be-efaf-41aa-81ba-106712519f8a" />


| Module | Rôle |
|--------|------|
| Runtime Logs | Logs Logcat en temps réel |
| Network Traffic | Interception HTTP/HTTPS + déchiffrement SSL |
| Frida / Hooking | Hook de méthodes Java au runtime |
| File Monitor | Fichiers créés / modifiés par l'app |
| Intent Monitor | Intents émis et reçus |

---

##  Challenges DIVA

| # | Challenge | Vulnérabilité OWASP Mobile |
|---|-----------|---------------------------|
| 1 | Insecure Logging | M9 — Données en clair dans Logcat |
| 2 | Hardcoded Credentials | M9 — Secrets dans le code source |
| 3–4 | Insecure Data Storage | M9 — Fichiers / SharedPrefs non chiffrés |
| 5 | Input Validation | M4 — Injection SQL |
| 6–8 | Access Control | M1 — Intents non protégés |
| 9 | Insecure Network | M5 — Trafic HTTP non chiffré |
| 10 | Binary Protections | M7 — Absence d'obfuscation |

---

## Export des résultats

Depuis MobSF → **Generate Report** (en bas du rapport) :

- **PDF** — rapport statique + dynamique consolidés
- **JSON** — données brutes pour intégration SIEM
- **HTML** — visualisation directe dans le navigateur

---

##  Récapitulatif des commandes

```bash
# Cloner MobSF
git clone https://github.com/MobSF/Mobile-Security-Framework-MobSF.git

# Démarrer l'émulateur (Linux)
./scripts/start_avd.sh MobSF_DIVA_API_30

# Démarrer l'émulateur (Windows PowerShell)
scripts\start_avd.ps1 MobSF_DIVA_API_30

# Vérifier la connexion ADB
adb devices

# Télécharger l'image Docker MobSF
docker pull opensecurity/mobile-security-framework-mobsf:latest

# Lancer MobSF
docker run -it --rm -p 8000:8000 \
  -e MOBSF_ANALYZER_IDENTIFIER=emulator-5554 \
  opensecurity/mobile-security-framework-mobsf:latest
```




*Lab réalisé dans le cadre du module Sécurité Mobile — 4CIR*
