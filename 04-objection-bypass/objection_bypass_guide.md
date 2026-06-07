# 🛡️ Étape 4 — Lancer l'app avec Objection et désactiver le SSL Pinning

## Objectif

Utiliser Objection pour injecter des hooks Frida dans l'application cible et désactiver le SSL pinning, permettant au proxy de capturer le trafic HTTPS en clair.

---

## 4.1 — Identifier le package de l'app cible

```powershell
# Lister toutes les apps installées
frida-ps -Uai

# Filtrer (PowerShell) :
frida-ps -Uai | Select-String -Pattern "example"
frida-ps -Uai | Select-String -Pattern "ssl,okhttp,https"

# Filtrer (Linux/macOS) :
frida-ps -Uai | grep -i "example"
```

### Résultat attendu

```
  PID  Name                    Identifier
-----  ----------------------  ----------------------------------
    -  Example App             com.example.app
    -  Chrome                  com.android.chrome
```

> Notez le **Identifier** (ex: `com.example.app`) — c'est le nom de package utilisé par Objection.

---

## 4.2 — Stratégie : Spawn vs Attach

Objection propose deux méthodes d'injection :

| Méthode | Quand l'utiliser | Commande |
|---------|-----------------|----------|
| **Spawn** | L'app fait le pinning très tôt (au démarrage) — **recommandé** | `objection -g <pkg> explore --startup-command "..."` |
| **Attach** | Le spawn crash ou l'app a un processus déjà en cours | `objection -g <pkg> explore` puis commande dans le shell |

### Comment choisir ?

```
┌─────────────────────────────────────────────────────────────┐
│  1. Essayer SPAWN d'abord (couvre la majorité des cas)      │
│  2. Si crash → essayer ATTACH                               │
│  3. Si les deux échouent → diagnostiquer (voir §4.5)       │
└─────────────────────────────────────────────────────────────┘
```

---

## 4.3 — Méthode A : Injection au démarrage (Spawn) — Recommandé

```powershell
objection -g com.example.app explore --startup-command "android sslpinning disable"
```

### Ce qui se passe en interne

1. Objection demande à Frida de **spawner** (relancer) l'app
2. L'app est **suspendue** avant l'exécution du code principal
3. Objection injecte les hooks SSL pinning
4. L'app reprend son exécution avec le pinning neutralisé

### Logs attendus

```
Using USB device `Samsung SM-XXXX`
Agent injected and calculation started
Hooking into com.example.app on <device>...

(agent) [ssl-pinning-bypass] Registering job <id>. Started TrustManager bypass
(agent) [ssl-pinning-bypass] TrustManager bypass applied
(agent) [ssl-pinning-bypass] OkHTTP CertificatePinner bypass applied

com.example.app on (samsung: 13.0) [usb] #
```

> Le prompt `#` indique que vous êtes dans le **shell interactif** d'Objection.

---

## 4.4 — Méthode B : Attache à une app déjà ouverte (Attach)

```powershell
# 1. Ouvrir l'app normalement sur le téléphone

# 2. Attacher Objection
objection -g com.example.app explore
```

### Dans le shell Objection

```
com.example.app on (samsung: 13.0) [usb] # android sslpinning disable
```

### Logs attendus

```
(agent) [ssl-pinning-bypass] Registering job <id>. Started TrustManager bypass
(agent) [ssl-pinning-bypass] TrustManager bypass applied
(agent) [ssl-pinning-bypass] OkHTTP CertificatePinner bypass applied
```

> ⚠️ Avec l'attach, certaines vérifications SSL faites à l'initialisation de l'app peuvent avoir déjà eu lieu. Si l'app est déjà en échec, fermez-la et réessayez avec le spawn.

---

## 4.5 — Ce que fait `android sslpinning disable`

Objection applique automatiquement des hooks Frida sur **4 couches** :

| # | Cible | Technique |
|---|-------|-----------|
| 1 | `SSLContext.init()` | Injection d'un `TrustManager` permissif qui accepte tous les certificats |
| 2 | `TrustManagerImpl.checkTrustedRecursive()` | Bypass Conscrypt — valide toujours la chaîne |
| 3 | `TrustManagerImpl.verifyChain()` | Bypass de la vérification de la chaîne de certificats |
| 4 | `OkHTTP CertificatePinner.check()` | Neutralisation du check OkHttp — retour immédiat |

### Schéma de fonctionnement

```
┌──────────────────────────────────────────────────────────────────┐
│                     APPLICATION ANDROID                          │
│                                                                  │
│  Requête HTTPS ──►  SSLContext.init()  ◄── Hook Objection (1)   │
│                        │                                         │
│                     TrustManager  ◄──────── Hook Objection (2,3)│
│                        │                                         │
│                  CertificatePinner  ◄────── Hook Objection (4)  │
│                        │                                         │
│                   Connexion TLS ──────►  Proxy (Burp/mitmproxy) │
│                                                 │                │
│                                            Serveur réel          │
└──────────────────────────────────────────────────────────────────┘
```

---

## 4.6 — Commandes utiles dans le shell Objection

### Aide et diagnostic

| Commande | Description |
|----------|-------------|
| `help` | Afficher toutes les commandes disponibles |
| `help android sslpinning` | Aide spécifique au SSL pinning |
| `android hooking search classes pin` | Chercher les classes contenant "pin" |
| `android hooking search classes trust` | Chercher les classes contenant "trust" |
| `android hooking search classes okhttp` | Chercher les classes OkHttp |
| `android hooking search classes ssl` | Chercher les classes SSL |
| `android hooking search classes conscrypt` | Chercher les classes Conscrypt |

### Exploration de l'app

| Commande | Description |
|----------|-------------|
| `android hooking list classes` | Lister toutes les classes chargées |
| `android hooking list class_methods <class>` | Méthodes d'une classe |
| `android hooking watch class <class>` | Surveiller les appels à une classe |
| `android hooking watch class_method <method> --dump-args` | Hook une méthode avec ses arguments |
| `env` | Chemins de l'app (data, cache, etc.) |

### Gestion de session

| Commande | Description |
|----------|-------------|
| `jobs list` | Lister les jobs en cours (hooks actifs) |
| `jobs kill <id>` | Arrêter un job spécifique |
| `reconnect` | Reconnecter à l'agent Frida |
| `exit` | Quitter Objection |

---

## 4.7 — Variante : tout en une ligne

Si l'app tolère le spawn, cette commande unique suffit :

```powershell
objection -g com.example.app explore --startup-command "android sslpinning disable"
```

Ensuite, naviguez simplement dans l'app — tout le trafic HTTPS apparaît dans le proxy.

---

## 4.8 — Dépannage immédiat

| Problème | Solution |
|----------|---------|
| Objection crash au spawn | Essayer l'attach (§4.4) |
| `Failed to spawn` | App protégée → attach. Ou vérifier le nom de package |
| Pas de logs `[ssl-pinning-bypass]` | Relancer la commande dans le shell. L'app n'a peut-être pas encore chargé les classes réseau |
| L'app reste en échec malgré le bypass | Naviguez dans l'app puis relancez `android sslpinning disable`. Certaines apps ont plusieurs clients HTTP |
| `unable to connect to remote frida-server` | Voir Étape 2 — frida-server doit tourner |

---

## Résumé

| Aspect | Détail |
|--------|--------|
| **Spawn** | `objection -g <pkg> explore --startup-command "android sslpinning disable"` |
| **Attach** | `objection -g <pkg> explore` puis `android sslpinning disable` |
| **Couverture** | SSLContext, TrustManager, Conscrypt, OkHttp |
| **Logs de succès** | `TrustManager bypass applied` + `CertificatePinner bypass applied` |
| **Limitation** | Pas de bypass natif (BoringSSL) — voir Lab 15 si nécessaire |

> ✅ Le SSL pinning est désactivé. Passer à l'étape 5 (Validation).
