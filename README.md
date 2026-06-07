# 🔐 Lab 16 — Inspection HTTPS Android : Désactivation du SSL Pinning avec Objection + Proxy (Burp/mitmproxy)

> **Outils** : Objection · Frida · Burp Suite / mitmproxy | **Cadre** : OWASP MASVS/MASTG | **Niveau** : Intermédiaire

---

## 📋 Table des matières

- [Avertissement légal](#-avertissement-légal)
- [Description](#-description)
- [Objectifs pédagogiques](#-objectifs-pédagogiques)
- [Prérequis](#-prérequis)
- [Architecture du lab](#-architecture-du-lab)
- [Méthodologie](#-méthodologie)
- [Déroulement des étapes](#-déroulement-des-étapes)
- [Livrables](#-livrables)
- [Résultats clés](#-résultats-clés)
- [Exercices pratiques](#-exercices-pratiques-barème)
- [Règles et éthique](#-règles-et-éthique)
- [Troubleshooting](#-troubleshooting)
- [FAQ rapide](#-faq-rapide)
- [Bonnes pratiques](#-bonnes-pratiques)
- [Glossaire](#-glossaire)
- [Ressources](#-ressources-complémentaires)
- [Auteur](#-auteur)

---

## ⚠️ Avertissement légal

> **N'appliquez ce lab que dans un cadre légal (appareils/apps vous appartenant ou audit autorisé).**
>
> Le contournement du SSL pinning sert à **analyser la sécurité réseau** d'une application, pas à compromettre des systèmes en production. Ne capturez pas de données sensibles réelles, ne redistribuez pas d'APK patchés, et retirez toujours le certificat CA et frida-server après vos tests.

---

## 📖 Description

Ce lab couvre la **désactivation du SSL pinning** sur Android via **Objection** (surcouche CLI de Frida) combinée à un **proxy TLS** (Burp Suite ou mitmproxy) pour capturer le trafic HTTPS en clair. Contrairement au Lab 15 (approche multi-couches avec scripts Frida), ce lab se concentre sur un workflow **simplifié et efficace** basé sur une seule commande Objection.

| Aspect | Détail |
|--------|--------|
| **Outil principal** | Objection 1.11.0 (CLI Python basée sur Frida) |
| **Outil secondaire** | Frida 16.5.9 (runtime d'instrumentation) |
| **Proxy TLS** | Burp Suite Community/Pro ou mitmproxy |
| **Type d'analyse** | Instrumentation dynamique — bypass SSL pinning |
| **Cible** | Application Android avec SSL pinning (OkHttp/TrustManager) |
| **Environnement** | Appareil Android (API 26+) avec Débogage USB |
| **Référentiel** | OWASP MASVS v2.0 — MASVS-NETWORK |

### Différence avec le Lab 15

| Aspect | Lab 15 | Lab 16 (ce lab) |
|--------|--------|-----------------|
| **Focus** | Scripts Frida JS multi-couches | Objection CLI uniquement |
| **Scripts manuels** | 5 scripts (universel, OkHttp, WebView, natif, enum) | Aucun script à écrire |
| **Bypass natif** | ✅ BoringSSL/OpenSSL | ❌ Java seulement |
| **Complexité** | Intermédiaire-Avancé | Intermédiaire |
| **Commande clé** | `frida -U -f ... -l script.js` | `android sslpinning disable` |

---

## 🎯 Objectifs pédagogiques

- ✅ **Installer** Objection (via pipx ou pip) et vérifier la configuration Frida
- ✅ **Déployer** frida-server sur l'appareil Android (versions alignées)
- ✅ **Configurer** un proxy TLS (Burp Suite / mitmproxy) et installer le certificat CA
- ✅ **Lancer** l'app avec Objection et exécuter `android sslpinning disable` (spawn ou attach)
- ✅ **Valider** la capture du trafic HTTPS en clair dans le proxy
- ✅ **Diagnostiquer** et dépanner les cas courants (crash, pas de trafic, natif)

---

## ⚙️ Prérequis

### Environnement

| Composant | Requis |
|-----------|--------|
| 💻 PC | Windows (PowerShell) / macOS / Linux |
| 🐍 Python | 3.8+ avec pip |
| 🔧 ADB | [Android Platform Tools](https://developer.android.com/tools/releases/platform-tools) dans le PATH |
| 📱 Android | 8.0+ avec Débogage USB activé |
| 🔑 Frida | Client PC + frida-server Android (**versions alignées**) |
| 🌐 Proxy | Burp Suite ou mitmproxy installé |

### Outils utilisés

| Outil | Version | Rôle |
|-------|---------|------|
| Objection | 1.11.0 | CLI de bypass SSL — commande unique |
| Frida CLI | 16.5.9 | Runtime d'instrumentation (dépendance) |
| frida-server | 16.5.9 | Agent sur l'appareil Android |
| Burp Suite | Latest | Proxy TLS avec GUI |
| mitmproxy | 10.x | Proxy TLS scriptable (alternative) |

### Vérifications rapides

```powershell
python --version          # Python 3.11.x
pip --version             # pip 24.x
adb version               # 1.0.41
adb devices               # → device
objection --version       # 1.11.0+
frida --version            # 16.5.9
```

---

## 📁 Architecture du lab

```
Lab16/
├── 📂 00-setup/                            # Configuration et périmètre
│   ├── scope.md                            # Périmètre d'analyse et autorisations
│   └── environment_check.md                # Vérification de l'environnement
│
├── 📂 01-install-objection/                # Étape 1 — Installation Objection + Frida
│   └── objection_install_guide.md          # Guide complet (pipx / pip)
│
├── 📂 02-frida-server/                     # Étape 2 — Appareil et frida-server
│   └── frida_server_guide.md               # Déploiement et lancement
│
├── 📂 03-proxy-setup/                      # Étape 3 — Proxy TLS et certificat CA
│   └── proxy_ca_guide.md                   # Burp/mitmproxy + CA + validation
│
├── 📂 04-objection-bypass/                 # Étape 4 — Bypass SSL avec Objection
│   ├── objection_bypass_guide.md           # Guide principal (spawn/attach)
│   ├── spawn_vs_attach.md                  # Arbre de décision spawn vs attach
│   └── objection_commands.md               # Référence complète des commandes
│
├── 📂 05-validation/                       # Étape 5 — Validation
│   └── validation_guide.md                 # Preuves, livrables, nettoyage
│
├── 📂 06-troubleshooting/                  # Dépannage
│   ├── troubleshooting.md                  # FAQ et solutions (10 cas)
│   └── diagnostic_commands.md              # Commandes de diagnostic rapide
│
├── 📂 07-annexes/                          # Références
│   ├── snippets.md                         # Commandes ADB/Frida/Objection/Proxy
│   └── bonnes_pratiques.md                 # Best practices
│
├── analyse_info.txt                        # Métadonnées de traçabilité
├── commands.log                            # Log chronologique des commandes
├── checklist_fin.md                        # Checklist de clôture signée
└── README.md                               # Ce fichier
```

---

## 🔬 Méthodologie

```
┌──────────────┐   ┌──────────────┐   ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│   ÉTAPE 1    │──▶│   ÉTAPE 2    │──▶│   ÉTAPE 3    │──▶│   ÉTAPE 4    │──▶│   ÉTAPE 5    │
│  Objection   │   │ frida-server │   │ Proxy + CA   │   │  Objection   │   │  Validation  │
│  installer   │   │  déployer    │   │  configurer  │   │ SSL disable  │   │  + nettoyage │
└──────────────┘   └──────────────┘   └──────────────┘   └──────────────┘   └──────────────┘
```

| Phase | Actions | Outils | Livrables |
|-------|---------|--------|-----------|
| Setup | Vérifier Python, ADB, connexion | pip, ADB | `00-setup/` |
| Objection | Installer via pipx ou pip | pip/pipx | `01-install-objection/` |
| Appareil | Déployer frida-server, tester | ADB, Frida | `02-frida-server/` |
| Proxy | Burp/mitmproxy, CA, redirection | Proxy | `03-proxy-setup/` |
| Bypass | `android sslpinning disable` (spawn/attach) | Objection | `04-objection-bypass/` |
| Validation | Trafic HTTPS en clair dans le proxy | Proxy | `05-validation/` |

---

## 📝 Déroulement des étapes

### Étape 1 — Installer Objection et Frida (15 min)

```powershell
# Option isolée (recommandée) :
pip install --user pipx
pipx ensurepath
pipx install objection

# OU via pip classique :
pip install --upgrade objection frida frida-tools

# Vérification :
objection --version                                        # 1.11.0+
frida --version                                            # 16.5.9
python -c "import frida; print(frida.__version__)"         # 16.5.9
```

> **Windows** : si `objection` n'est pas reconnu, ajoutez `%USERPROFILE%\AppData\Roaming\Python\Python311\Scripts` au PATH.

→ Guide : [`01-install-objection/objection_install_guide.md`](./01-install-objection/objection_install_guide.md)

### Étape 2 — Préparer l'appareil et frida-server (15 min)

```powershell
adb devices                                                # → device
adb shell getprop ro.product.cpu.abi                       # → arm64-v8a
# Télécharger frida-server depuis https://github.com/frida/frida/releases
adb push frida-server /data/local/tmp/
adb shell chmod 755 /data/local/tmp/frida-server
adb shell "/data/local/tmp/frida-server -l 0.0.0.0 &"
# Port forwarding (optionnel) :
adb forward tcp:27042 tcp:27042
adb forward tcp:27043 tcp:27043
# Vérification :
frida-ps -Uai                                             # → liste apps
```

→ Guide : [`02-frida-server/frida_server_guide.md`](./02-frida-server/frida_server_guide.md)

### Étape 3 — Configurer le proxy et installer la CA (20 min)

```powershell
# Lancer Burp (Proxy > Options > 0.0.0.0:8080)
# OU mitmproxy :
mitmproxy -p 8080

# Proxy Wi-Fi du téléphone → IP_PC:8080
# CA : http://burp ou http://mitm.it depuis le navigateur du téléphone
# Alternative USB :
adb reverse tcp:8080 tcp:8080
```

> ⚠️ SSL pinning désactivé + CA installée = les **deux** sont nécessaires pour un MITM complet.

→ Guide : [`03-proxy-setup/proxy_ca_guide.md`](./03-proxy-setup/proxy_ca_guide.md)

### Étape 4 — Objection : désactiver le SSL pinning (20 min)

```powershell
# Identifier le package :
frida-ps -Uai | Select-String -Pattern "example"

# SPAWN (recommandé — patcher avant l'init) :
objection -g com.example.app explore --startup-command "android sslpinning disable"

# ATTACH (si crash — app déjà ouverte) :
objection -g com.example.app explore
# Puis dans le shell :
android sslpinning disable
```

#### Résultat attendu

```
(agent) [ssl-pinning-bypass] Registering job. Started TrustManager bypass
(agent) [ssl-pinning-bypass] TrustManager bypass applied
(agent) [ssl-pinning-bypass] OkHTTP CertificatePinner bypass applied
```

#### Commandes utiles dans le shell Objection

```
help android sslpinning
android hooking search classes pin
android hooking search classes okhttp
android hooking search classes trust
```

→ Guide : [`04-objection-bypass/objection_bypass_guide.md`](./04-objection-bypass/objection_bypass_guide.md)

### Étape 5 — Validation (15 min)

- Naviguez dans l'app (login, API, recherche) pour générer du trafic
- Dans Burp/mitmproxy, vérifiez :
  - ✅ Les requêtes HTTPS de l'app apparaissent
  - ✅ Les réponses sont visibles sans alerte SSL côté app
  - ✅ URL, headers et body sont lisibles
- Dans Objection, surveillez la console pour les hooks déclenchés

→ Guide : [`05-validation/validation_guide.md`](./05-validation/validation_guide.md)

---

## 📦 Livrables

| # | Fichier | Description | Statut |
|---|---------|-------------|--------|
| 1 | `00-setup/scope.md` | Périmètre et autorisations | ✅ |
| 2 | `00-setup/environment_check.md` | Vérification environnement | ✅ |
| 3 | `01-install-objection/objection_install_guide.md` | Installation Objection + Frida | ✅ |
| 4 | `02-frida-server/frida_server_guide.md` | Déploiement frida-server | ✅ |
| 5 | `03-proxy-setup/proxy_ca_guide.md` | Configuration proxy + CA | ✅ |
| 6 | `04-objection-bypass/objection_bypass_guide.md` | Guide bypass SSL (spawn/attach) | ✅ |
| 7 | `04-objection-bypass/spawn_vs_attach.md` | Arbre de décision spawn vs attach | ✅ |
| 8 | `04-objection-bypass/objection_commands.md` | Référence commandes Objection | ✅ |
| 9 | `05-validation/validation_guide.md` | Validation et nettoyage | ✅ |
| 10 | `06-troubleshooting/troubleshooting.md` | FAQ et solutions (10 cas) | ✅ |
| 11 | `06-troubleshooting/diagnostic_commands.md` | Commandes de diagnostic | ✅ |
| 12 | `07-annexes/snippets.md` | Snippets ADB/Frida/Objection/Proxy | ✅ |
| 13 | `07-annexes/bonnes_pratiques.md` | Bonnes pratiques | ✅ |
| 14 | `analyse_info.txt` | Métadonnées traçabilité | ✅ |
| 15 | `commands.log` | Log chronologique | ✅ |
| 16 | `checklist_fin.md` | Checklist de clôture | ✅ |

---

## 📊 Résultats clés

### Synthèse

| Métrique | Valeur |
|----------|--------|
| **Couches SSL patchées (Objection)** | 4 (SSLContext, TrustManager, Conscrypt, OkHttp) |
| **Scripts manuels requis** | 0 (tout via Objection CLI) |
| **Modes d'injection testés** | 2 (spawn, attach) |
| **Outils utilisés** | 3 (Objection, Frida, Proxy TLS) |
| **Taux de bypass (Java)** | 100% |

### Ce que patch `android sslpinning disable`

```
COUCHE 1 — SSLContext.init()                  ██████████████████  Patché ✅
COUCHE 2 — TrustManagerImpl.checkTrusted      ██████████████████  Patché ✅
COUCHE 3 — TrustManagerImpl.verifyChain       ██████████████████  Patché ✅
COUCHE 4 — OkHTTP CertificatePinner.check     ██████████████████  Patché ✅
COUCHE 5 — Pinning natif (BoringSSL)          ░░░░░░░░░░░░░░░░░░  Non couvert ❌
```

> Pour le pinning natif, combinez avec un script Frida dédié (voir Lab 15).

### Logs typiques

| Log | Source | Signification |
|-----|--------|---------------|
| `[ssl-pinning-bypass] TrustManager bypass applied` | Objection | TrustManager neutralisé |
| `[ssl-pinning-bypass] OkHTTP CertificatePinner bypass applied` | Objection | OkHttp bypass actif |
| `Registering job <id>` | Objection | Hook enregistré |

---

## 📝 Exercices pratiques (barème)

| Critère | Points | Description |
|---------|--------|-------------|
| **Setup complet** | 15 | Objection, Frida, ADB, frida-server fonctionnels |
| **Proxy + CA** | 15 | Proxy configuré, CA installée, navigateur OK |
| **Bypass spawn** | 25 | `--startup-command "android sslpinning disable"` + logs |
| **Bypass attach** | 15 | Fallback attach fonctionnel |
| **Validation proxy** | 20 | Capture HTTPS en clair (URL + headers + body) |
| **Dépannage** | 10 | Diagnostic d'un cas de blocage |
| **Total** | **100** | |

---

## ⚖️ Règles et éthique

> **⚠️ Ce lab s'inscrit dans un cadre STRICTEMENT pédagogique et défensif.**

### ✅ Autorisé

- Analyse sur appareil/émulateur **contrôlé**
- Applications de test (propriétaires ou installées avec autorisation)
- Utilisation d'Objection à des fins d'apprentissage
- Inspection du trafic de ses propres applications

### ❌ Interdit

- Interception de trafic d'applications de production sans autorisation
- Capture de credentials ou données sensibles réelles
- Redistribution d'APK patchés
- Attaque Man-in-the-Middle sur des tiers
- Publication de données interceptées

---

## 🔧 Troubleshooting

<details><summary><b>unable to connect to remote frida-server</b></summary>

`adb devices` → `device`. `adb shell ps | grep frida` → actif. Versions alignées. `adb forward tcp:27042 tcp:27042`.
</details>

<details><summary><b>Objection crash au spawn</b></summary>

Essayer l'attach : ouvrir l'app → `objection -g <pkg> explore` → `android sslpinning disable`. Vérifier le nom de package exact avec `frida-ps -Uai`.
</details>

<details><summary><b>Pas de trafic dans le proxy</b></summary>

Vérifier proxy Wi-Fi (IP/port). Vérifier CA installée (navigateur test). Vérifier pare-feu. Certaines apps contournent le proxy → `adb reverse` ou VPN local.
</details>

<details><summary><b>L'app utilise du pinning natif (BoringSSL)</b></summary>

Objection cible uniquement Java. Pour le natif, utiliser un script Frida dédié (voir Lab 15) ou `frida-trace -U -i "SSL_*"`.
</details>

<details><summary><b>L'app reste en échec malgré le succès Objection</b></summary>

Naviguer dans l'app puis relancer `android sslpinning disable`. Chercher : `android hooking search classes okhttp`. L'app peut avoir plusieurs clients HTTP.
</details>

→ FAQ complète : [`06-troubleshooting/troubleshooting.md`](./06-troubleshooting/troubleshooting.md)

---

## ❓ FAQ rapide

**« Pourquoi Objection plutôt qu'un script Frida ? »**
> Objection offre une commande unique (`android sslpinning disable`) sans écrire de JavaScript. Idéal pour un bypass rapide.

**« Faut-il installer la CA si Objection bypasse le pinning ? »**
> **Oui.** Objection neutralise les checks de l'app, mais le proxy a besoin d'une CA pour générer des certificats valides. Les deux sont complémentaires.

**« Spawn ou attach ? »**
> **Spawn** d'abord (patcher avant l'init). Si crash → **attach** (ouvrir l'app, puis attacher Objection).

**« Que faire si l'app a du pinning natif ? »**
> Objection ne couvre que Java. Combinez avec un script Frida natif (Lab 15) pour BoringSSL/OpenSSL.

**« Tout en une ligne ? »**
> `objection -g com.example.app explore --startup-command "android sslpinning disable"` — puis naviguez dans l'app.

---

## 💡 Bonnes pratiques

| Pratique | Raison |
|----------|--------|
| Versions Frida alignées (client = server) | Éviter erreurs de protocole |
| Spawn d'abord, attach si crash | Approche progressive |
| Ne surchargez pas `--startup-command` | Une commande à la fois |
| Vérifiez avec le navigateur avant l'app | Isoler proxy vs pinning |
| Minimisez les logs en session réelle | Certaines apps détectent un IO verbeux |
| Nettoyez après le test | CA retirée, frida-server arrêté |

---

## 📋 Checklist rapide

- [x] Objection, Frida, frida-server installés et versions alignées
- [x] Appareil connecté (`adb devices` → `device`)
- [x] frida-server lancé ; `frida-ps -Uai` liste les apps
- [x] Proxy TLS opérationnel (Burp / mitmproxy)
- [x] Certificat CA installé sur l'appareil
- [x] `android sslpinning disable` exécuté (spawn ou attach)
- [x] Requêtes HTTPS de l'app visibles en clair dans le proxy
- [x] Points de blocage traités (Cronet/natif → Lab 15 si besoin)
- [x] CA et frida-server retirés après tests

---

## 📖 Glossaire

| Terme | Définition |
|-------|------------|
| **SSL Pinning** | Épinglage de certificat — l'app n'accepte qu'un certificat spécifique |
| **Objection** | CLI Python de bypass mobile basée sur Frida |
| **Frida** | Framework d'instrumentation dynamique (injection de code) |
| **TrustManager** | Interface Java pour valider les certificats TLS |
| **CertificatePinner** | Classe OkHttp pour l'épinglage de certificat |
| **CA** | Certificate Authority — signe les certificats |
| **MitM** | Man-in-the-Middle — interception légitime du trafic |
| **Spawn** | Relancer l'app avec injection avant l'init |
| **Attach** | Se connecter à un processus déjà en cours |
| **Conscrypt** | Implémentation TLS d'Android (basée sur BoringSSL) |
| **Network Security Config** | Config XML Android pour les règles de sécurité réseau |

---

## 📚 Ressources complémentaires

- 📘 [Objection — GitHub](https://github.com/sensepost/objection)
- 📗 [Objection — Wiki](https://github.com/sensepost/objection/wiki)
- 📕 [Frida — Documentation officielle](https://frida.re/docs/home/)
- 📙 [Frida GitHub — Releases](https://github.com/frida/frida/releases)
- 📔 [OWASP MASTG — Network Testing](https://mas.owasp.org/MASTG/)
- 📓 [Burp Suite](https://portswigger.net/burp)
- 📒 [mitmproxy](https://mitmproxy.org/)
- 📖 [Android Platform Tools](https://developer.android.com/tools/releases/platform-tools)

---

## 👤 Auteur

| | |
|---|---|
| **Analyste** | Étudiant Sécurité |
| **Cours** | Sécurité Mobile — Lab 16 |
| **Date** | 2026-06-08 |
| **Durée** | 2h30 |

---

<div align="center">

**⚡ Lab réalisé dans un cadre strictement pédagogique et défensif ⚡**

*Sécurité Mobile — Inspection HTTPS Android : Désactivation du SSL Pinning avec Objection + Proxy*

</div>
