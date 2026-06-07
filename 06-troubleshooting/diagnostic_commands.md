# 🔍 Commandes de diagnostic — Lab 16

## Diagnostic environnement

```powershell
# ─── Versions ────────────────────────────────────────────
python --version                                           # Python 3.11.x
pip --version                                              # pip 24.x
adb version                                                # 1.0.41
frida --version                                            # 16.5.9
objection --version                                        # 1.11.0+
python -c "import frida; print(frida.__version__)"         # 16.5.9

# ─── Appareil ───────────────────────────────────────────
adb devices                                                # → device
adb shell getprop ro.product.cpu.abi                       # → arm64-v8a
adb shell getprop ro.build.version.sdk                     # → API level (ex: 33)
adb shell getprop ro.build.version.release                 # → Android version (ex: 13)
```

---

## Diagnostic frida-server

```powershell
# ─── Vérifier que frida-server tourne ────────────────────
adb shell ps | grep frida
adb shell "pidof frida-server"                             # → PID ou vide

# ─── Relancer si nécessaire ─────────────────────────────
adb shell "kill $(pidof frida-server)" 2>$null
adb shell "/data/local/tmp/frida-server -l 0.0.0.0 &"

# ─── Port forwarding ───────────────────────────────────
adb forward tcp:27042 tcp:27042
adb forward tcp:27043 tcp:27043
adb forward --list                                         # Vérifier les forwards

# ─── Test connexion Frida ───────────────────────────────
frida-ps -U                                                # Liste processus (USB)
frida-ps -Uai                                              # Liste apps installées
```

---

## Diagnostic réseau / proxy

```powershell
# ─── IP du PC ───────────────────────────────────────────
ipconfig                                                   # Windows
# ifconfig ou ip addr show                                 # Linux/macOS

# ─── Test connectivité ──────────────────────────────────
# Depuis le téléphone : naviguer vers http://<IP_PC>:8080
# → doit répondre (page Burp ou erreur mitmproxy = OK)

# ─── Redirection USB ───────────────────────────────────
adb reverse tcp:8080 tcp:8080
adb reverse --list                                         # Vérifier

# ─── Pare-feu Windows ──────────────────────────────────
# Vérifier que le port 8080 n'est pas bloqué
netstat -an | findstr 8080                                 # Le port doit être en LISTENING
```

---

## Diagnostic Objection

```powershell
# ─── Lister les apps ────────────────────────────────────
frida-ps -Uai | Select-String "example"

# ─── Spawn test ─────────────────────────────────────────
objection -g com.example.app explore --startup-command "android sslpinning disable"

# ─── Attach test ────────────────────────────────────────
objection -g com.example.app explore
# Dans le shell :
android sslpinning disable
```

### Commandes de diagnostic dans le shell Objection

```
# ─── Chercher les classes réseau ────────────────────────
android hooking search classes ssl
android hooking search classes trust
android hooking search classes pin
android hooking search classes okhttp
android hooking search classes conscrypt
android hooking search classes certificate
android hooking search classes verify

# ─── Jobs actifs ────────────────────────────────────────
jobs list

# ─── Environnement de l'app ─────────────────────────────
env

# ─── Connexion agent ────────────────────────────────────
ping
reconnect
```

---

## Commandes de nettoyage

```powershell
# ─── Arrêter frida-server ───────────────────────────────
adb shell "kill $(pidof frida-server)"

# ─── Supprimer frida-server ─────────────────────────────
adb shell rm /data/local/tmp/frida-server

# ─── Supprimer les forwards ─────────────────────────────
adb forward --remove-all
adb reverse --remove-all

# ─── Vérifier le nettoyage ──────────────────────────────
adb shell ps | grep frida                                  # → vide
adb shell ls /data/local/tmp/frida-server                  # → No such file
```
