# 📎 Snippets de référence — Lab 16

## Commandes ADB essentielles

```powershell
# ─── Connexion ──────────────────────────────────────────
adb devices                                                # Liste appareils
adb shell                                                  # Shell interactif
adb shell getprop ro.product.cpu.abi                       # Architecture CPU
adb shell getprop ro.build.version.release                 # Version Android
adb shell getprop ro.build.version.sdk                     # API level

# ─── Transfert de fichiers ──────────────────────────────
adb push <local> <remote>                                  # PC → appareil
adb pull <remote> <local>                                  # Appareil → PC

# ─── Réseau ─────────────────────────────────────────────
adb reverse tcp:8080 tcp:8080                              # Reverse port
adb forward tcp:27042 tcp:27042                            # Forward port
adb reverse --list                                         # Lister les reverses
adb forward --list                                         # Lister les forwards
adb reverse --remove-all                                   # Supprimer les reverses
adb forward --remove-all                                   # Supprimer les forwards

# ─── Apps ───────────────────────────────────────────────
adb shell pm list packages                                 # Toutes les apps
adb shell pm list packages -3                              # Apps tierces
adb shell pm path com.example.app                          # Chemin de l'APK
adb shell am force-stop com.example.app                    # Forcer l'arrêt
```

---

## Commandes Frida essentielles

```powershell
# ─── Connexion ──────────────────────────────────────────
frida-ps -U                                                # Processus (USB)
frida-ps -Uai                                              # Apps installées
frida-ps -D <serial>                                       # Appareil spécifique

# ─── Injection ──────────────────────────────────────────
frida -U -f com.example.app -l script.js --no-pause        # Spawn
frida -U -n "AppName" -l script.js                         # Attach

# ─── Traçage ────────────────────────────────────────────
frida-trace -U -i "SSL_*" com.example.app                  # Trace native
frida-trace -U -j "*ssl*" com.example.app                  # Trace Java
```

---

## Commandes Objection essentielles

```powershell
# ─── Lancement ──────────────────────────────────────────
objection -g com.example.app explore                       # Shell interactif
objection -g com.example.app explore \
  --startup-command "android sslpinning disable"           # Bypass au spawn
objection --serial XXXXX -g com.example.app explore        # Appareil spécifique

# ─── Dans le shell Objection ────────────────────────────
android sslpinning disable                                 # Bypass SSL
android root disable                                       # Bypass root detect
android hooking search classes <keyword>                   # Chercher classes
android hooking list class_methods <class>                 # Lister méthodes
android hooking watch class <class>                        # Surveiller appels
jobs list                                                  # Jobs actifs
env                                                        # Chemins de l'app
exit                                                       # Quitter
```

---

## Commandes proxy

### Burp Suite

| Action | Procédure |
|--------|-----------|
| Configurer le listener | Proxy > Options > Add > Port 8080, All interfaces |
| Désactiver l'intercept | Proxy > Intercept > Intercept is off |
| Voir l'historique | Proxy > HTTP history |
| Filtrer par domaine | HTTP history > Filtre > Host contains |
| Exporter certificat CA | Proxy > Options > Import/Export CA certificate |

### mitmproxy

```powershell
# Modes de lancement
mitmproxy -p 8080                                          # Terminal interactif
mitmweb -p 8080                                            # Interface web
mitmdump -p 8080                                           # Headless / scriptable

# Options utiles
mitmproxy -p 8080 --mode transparent                       # Mode transparent
mitmdump -p 8080 -w output.flow                            # Sauvegarder le flux
mitmdump -p 8080 -r input.flow                             # Relire un flux
```

---

## Installation rapide (one-liner)

```powershell
# Tout installer d'un coup :
pip install --upgrade objection frida frida-tools

# Vérifier :
objection --version && frida --version && adb version
```

---

## frida-server : déploiement rapide

```powershell
# Télécharger, décompresser, pousser, lancer :
adb push frida-server /data/local/tmp/
adb shell chmod 755 /data/local/tmp/frida-server
adb shell "/data/local/tmp/frida-server -l 0.0.0.0 &"
frida-ps -Uai
```

---

## Workflow complet (rappel)

```powershell
# 1. frida-server
adb shell "/data/local/tmp/frida-server -l 0.0.0.0 &"

# 2. Proxy
# Lancer Burp/mitmproxy, configurer Wi-Fi proxy sur le téléphone

# 3. Objection
objection -g com.example.app explore --startup-command "android sslpinning disable"

# 4. Naviguer dans l'app → trafic visible dans le proxy

# 5. Nettoyage
exit
adb shell "kill $(pidof frida-server)"
```
