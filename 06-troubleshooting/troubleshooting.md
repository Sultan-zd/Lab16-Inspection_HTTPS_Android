# 🔧 Dépannage — Lab 16

## FAQ et solutions aux problèmes courants

---

### 1. `unable to connect to remote frida-server`

**Cause** : Frida (PC) ne peut pas joindre frida-server (appareil).

**Solutions** :

```powershell
# a) Vérifier que l'appareil est connecté
adb devices                       # → doit afficher "device"

# b) Vérifier que frida-server tourne
adb shell ps | grep frida         # → doit afficher le processus

# c) Si frida-server ne tourne pas, le relancer
adb shell "/data/local/tmp/frida-server -l 0.0.0.0 &"

# d) Versions alignées ?
frida --version                   # → 16.5.9 (PC)
# Le frida-server doit être de la même version

# e) Port forwarding
adb forward tcp:27042 tcp:27042
adb forward tcp:27043 tcp:27043
```

---

### 2. Objection crash au spawn / l'app crash

**Cause** : l'app a des protections anti-tampering ou anti-Frida.

**Solutions** :

```powershell
# a) Essayer l'attach au lieu du spawn
# Ouvrir l'app manuellement, puis :
objection -g com.example.app explore
# Dans le shell :
android sslpinning disable

# b) Simplifier la commande (pas de startup-command complexe)
objection -g com.example.app explore
# Puis exécuter les commandes une par une

# c) Vérifier le nom de package exact
frida-ps -Uai | Select-String "example"
```

---

### 3. Le proxy ne voit aucun trafic

**Cause** : le trafic ne passe pas par le proxy.

**Solutions** :

| Vérification | Action |
|-------------|--------|
| Proxy Wi-Fi | Paramètres > Wi-Fi > Proxy manuel → IP:port corrects ? |
| Même réseau | PC et téléphone sur le même Wi-Fi ? |
| Port ouvert | Pare-feu Windows désactivé pour le port 8080 ? |
| CA installée | Navigateur → https://example.com → pas d'avertissement ? |
| Proxy actif | Burp/mitmproxy bien lancé et listener actif ? |

```powershell
# Alternative USB si le Wi-Fi pose problème :
adb reverse tcp:8080 tcp:8080
# Puis proxy sur le téléphone → 127.0.0.1:8080
```

---

### 4. L'app fonctionne mais le proxy ne voit pas ses requêtes

**Cause** : l'app contourne le proxy système (sockets directs, Cronet, etc.).

**Solutions** :

| Méthode | Description |
|---------|-------------|
| **iptables (root)** | Rediriger tout le trafic via `iptables -t nat -A OUTPUT` |
| **VPN local** | Utiliser un outil comme `tun2socks` ou `Postern` |
| **Mode Wi-Fi entreprise** | Certains réseaux imposent le proxy via DHCP |
| **ProxyDroid (root)** | App qui force le proxy pour tout le trafic |

> Certaines apps (Flutter, React Native avec Cronet) ignorent la config proxy du système.

---

### 5. L'app utilise Cronet/Chromium ou du pinning natif

**Cause** : Objection cible les couches **Java** (OkHttp, TrustManager). Le pinning natif (BoringSSL, OpenSSL) n'est pas couvert.

**Solutions** :

```
┌──────────────────────────────────────────────────────────┐
│  Objection suffisant ?                                    │
│  → OkHttp, Conscrypt, TrustManager : OUI ✅             │
│  → BoringSSL natif, Cronet, Flutter : NON ❌             │
│                                                           │
│  Pour le pinning natif, voir Lab 15 :                    │
│  → Script Frida natif (SSL_get_verify_result, etc.)      │
│  → frida-trace -U -i "SSL_*" -i "X509_*"               │
└──────────────────────────────────────────────────────────┘
```

---

### 6. L'app reste en échec malgré le message de succès d'Objection

**Cause possible** :

- L'app a plusieurs clients HTTP (certains non patchés)
- Les classes réseau n'étaient pas encore chargées au moment du bypass
- Obfuscation des packages

**Solutions** :

```
# a) Naviguer dans l'app puis relancer le bypass
android sslpinning disable

# b) Chercher les classes réseau chargées
android hooking search classes okhttp
android hooking search classes conscrypt
android hooking search classes trust

# c) Vérifier si l'app a des packages ombrés (obfusqués)
android hooking search classes pin
android hooking search classes certificate
android hooking search classes verify
```

---

### 7. Obfuscation / packages renommés

**Cause** : le code de l'app est obfusqué (ProGuard, R8, DexGuard). Les classes n'ont plus leurs noms standard.

**Solutions** :

```
# a) Chercher des patterns connus
android hooking search classes pin
android hooking search classes trust
android hooking search classes cert
android hooking search classes ssl
android hooking search classes verify

# b) Lister toutes les classes et chercher manuellement
android hooking list classes

# c) Si Objection ne trouve pas les classes, utiliser un script
#    Frida personnalisé (voir Lab 15)
```

---

### 8. `Process crashed: Frida`

**Cause** : incompatibilité ou conflit avec l'app.

**Solutions** :

```powershell
# a) Mettre à jour Frida et Objection
pip install --upgrade objection frida frida-tools

# b) Redéployer frida-server (même version)
adb push frida-server /data/local/tmp/
adb shell chmod 755 /data/local/tmp/frida-server
adb shell "/data/local/tmp/frida-server -l 0.0.0.0 &"

# c) Essayer l'attach avec un délai
# Attendre 10 secondes après le lancement de l'app, puis attacher
```

---

### 9. Erreur `device not found` avec adb

```powershell
# a) Vérifier le câble USB (essayer un autre câble/port)
# b) Réinstaller les drivers ADB
# c) Redémarrer le serveur ADB
adb kill-server
adb start-server
adb devices
```

---

### 10. Le certificat CA n'est pas reconnu par l'app

**Cause** : Android 7+ ignore les CA utilisateur par défaut.

**Solutions** :

| Solution | Condition |
|----------|-----------|
| **Objection bypass** | Neutralise les TrustManagers → contourne la restriction |
| **Installer en CA système** | Nécessite root (Magisk + module MagiskTrustUserCerts) |
| **Patcher l'APK** | Modifier le `network_security_config.xml` de l'APK |

> En général, le bypass Objection suffit pour contourner cette restriction.

---

## Checklist de diagnostic rapide

```
□ adb devices → "device" (pas "unauthorized" ou vide)
□ frida-server tourne (adb shell ps | grep frida)
□ Versions alignées (frida PC = frida-server Android)
□ Proxy actif et listener sur le bon port
□ Proxy Wi-Fi configuré sur le téléphone (IP + port)
□ CA installée (test navigateur → pas d'erreur SSL)
□ Package correct (frida-ps -Uai | grep <nom>)
□ Objection lancé (spawn ou attach)
□ "android sslpinning disable" exécuté
□ Naviguer dans l'app pour générer du trafic
```
