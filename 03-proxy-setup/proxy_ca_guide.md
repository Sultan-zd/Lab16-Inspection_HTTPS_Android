# 🌐 Étape 3 — Configurer le proxy et installer la CA

## Objectif

Mettre en place un proxy TLS (Burp Suite ou mitmproxy) sur le PC et installer le certificat CA sur l'appareil Android pour intercepter le trafic HTTPS.

---

## 3.1 — Lancer le proxy sur le PC

### Option A : Burp Suite

1. Ouvrir **Burp Suite** (Community ou Pro)
2. **Proxy** > **Options** > **Proxy Listeners**
3. Modifier ou ajouter un listener :
   - **Bind to port** : `8080`
   - **Bind to address** : `All interfaces` (ou l'IP spécifique du PC)
4. Cliquer **OK** et s'assurer que le listener est actif ✅

| Paramètre | Valeur |
|-----------|--------|
| Port | `8080` |
| Adresse | `0.0.0.0` (All interfaces) |
| Intercept | `Off` (pour observer le flux sans le bloquer) |

### Option B : mitmproxy

```powershell
# Lancer mitmproxy (mode terminal interactif)
mitmproxy -p 8080

# Ou en mode web UI :
mitmweb -p 8080

# Ou en mode dump (headless) :
mitmdump -p 8080
```

| Mode | Commande | Interface |
|------|---------|-----------|
| Terminal | `mitmproxy -p 8080` | CLI interactive |
| Web | `mitmweb -p 8080` | Navigateur (http://127.0.0.1:8081) |
| Dump | `mitmdump -p 8080` | Stdout (scriptable) |

---

## 3.2 — Noter l'adresse IP du PC

```powershell
# Windows :
ipconfig
# Chercher : IPv4 Address. . . . : 192.168.X.Y

# macOS/Linux :
ifconfig
# ou
ip addr show
```

> Notez l'adresse IP du PC sur le réseau local (ex: `192.168.1.100`).

---

## 3.3 — Configurer le proxy Wi-Fi sur le téléphone

1. **Paramètres** > **Wi-Fi** > appui long sur le réseau connecté > **Modifier**
2. **Options avancées** > **Proxy** > **Manuel**
3. Renseigner :
   - **Nom d'hôte proxy** : `192.168.X.Y` (IP du PC)
   - **Port proxy** : `8080`
4. **Enregistrer**

> ⚠️ Le téléphone et le PC doivent être sur le **même réseau Wi-Fi**.

### Alternative USB (sans Wi-Fi)

```powershell
adb reverse tcp:8080 tcp:8080
```

Puis sur le téléphone, configurer le proxy vers `127.0.0.1:8080`.

---

## 3.4 — Installer le certificat CA sur le téléphone

### Burp Suite

1. Sur le téléphone, ouvrir le navigateur
2. Naviguer vers : **http://burp**
3. Cliquer **CA Certificate** pour télécharger le certificat
4. Ouvrir le fichier téléchargé
5. **Paramètres** > **Sécurité** > **Installer depuis le stockage**
6. Nommer le certificat (ex: `Burp CA`) et installer

### mitmproxy

1. Sur le téléphone, ouvrir le navigateur
2. Naviguer vers : **http://mitm.it**
3. Cliquer **Android** (icône du robot vert)
4. Ouvrir le fichier téléchargé
5. Installer comme certificat CA utilisateur

---

## 3.5 — Remarques importantes

### SSL Pinning vs. Certificat CA

```
┌─────────────────────────────────────────────────────────────┐
│  Le bypass SSL pinning (Objection) et la CA du proxy sont   │
│  COMPLÉMENTAIRES, pas équivalents :                         │
│                                                             │
│  CA installée → l'OS accepte les certificats du proxy       │
│  Objection    → l'app ignore ses propres vérifications      │
│                                                             │
│  → Il faut LES DEUX pour un MITM complet et propre         │
└─────────────────────────────────────────────────────────────┘
```

| Composant | Rôle |
|-----------|------|
| **CA proxy** | L'OS Android fait confiance aux certificats signés par le proxy |
| **Objection** | L'app ignore son propre épinglage de certificat |

### Network Security Config (Android 7+)

Depuis Android 7.0 (Nougat), les CA utilisateur ne sont **plus** approuvées par défaut pour les apps. Le patch Objection neutralise cette restriction en patchant les `TrustManagers` côté Java.

### Apps restrictives

Si l'app ignore la config proxy (sockets directs, Cronet), consultez la section Troubleshooting.

---

## 3.6 — Validation rapide

1. Sur le téléphone, ouvrir le navigateur
2. Visiter **https://example.com**
3. Vérifier dans le proxy :

| Proxy | Vérification |
|-------|-------------|
| **Burp** | Onglet **HTTP history** → requête GET vers `example.com` visible |
| **mitmproxy** | La requête apparaît dans la liste du terminal ou du web UI |

> ✅ Si la requête apparaît dans le proxy sans erreur SSL, la CA est correctement installée.

> ⚠️ L'app cible pourra encore refuser les connexions tant que le SSL pinning n'est pas désactivé (Étape 4 avec Objection).

---

## Résumé

| Étape | Action | Résultat |
|-------|--------|---------|
| Proxy | Lancer Burp/mitmproxy sur `0.0.0.0:8080` | Proxy actif |
| IP | Noter l'IP du PC | `192.168.X.Y` |
| Wi-Fi | Proxy manuel sur le téléphone | Trafic redirigé |
| CA | Installer certificat depuis http://burp ou http://mitm.it | CA installée |
| Test | Visiter https://example.com | Requête visible |

> ✅ Le proxy est opérationnel. Passer à l'étape 4 (Objection bypass).
