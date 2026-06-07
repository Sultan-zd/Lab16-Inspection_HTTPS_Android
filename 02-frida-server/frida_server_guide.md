# 📱 Étape 2 — Préparer l'appareil et démarrer frida-server

## Objectif

Déployer `frida-server` sur l'appareil Android pour permettre à Objection (et Frida) de s'y connecter via USB.

---

## 2.1 — Activer le Débogage USB

1. **Paramètres** > **À propos du téléphone** > **Numéro de build** → appuyer 7 fois
2. **Paramètres** > **Options développeur** > **Débogage USB** → ✅ Activé
3. Brancher le câble USB au PC
4. Accepter l'empreinte RSA sur l'appareil

### Vérification

```powershell
adb devices
# Résultat attendu :
# List of devices attached
# XXXXXXXXXX    device
```

> ⚠️ Si l'état est `unauthorized`, acceptez l'empreinte sur l'appareil. Si `offline`, débranchez et rebranchez.

---

## 2.2 — Identifier l'architecture CPU

```powershell
adb shell getprop ro.product.cpu.abi
# Résultat courant : arm64-v8a
```

| Architecture | Binaire frida-server |
|-------------|---------------------|
| `arm64-v8a` | `frida-server-X.Y.Z-android-arm64` |
| `armeabi-v7a` | `frida-server-X.Y.Z-android-arm` |
| `x86_64` | `frida-server-X.Y.Z-android-x86_64` |
| `x86` | `frida-server-X.Y.Z-android-x86` |

---

## 2.3 — Télécharger frida-server

1. Aller sur : **https://github.com/frida/frida/releases**
2. Chercher la **même version** que `frida --version` sur le PC (ex: `16.5.9`)
3. Télécharger `frida-server-16.5.9-android-<arch>.xz`
4. Décompresser avec **7-Zip** (Windows) ou `xz -d` (Linux/macOS)

> ⚠️ **CRITIQUE** : la version du frida-server doit correspondre **exactement** à la version du client Frida sur le PC.

```powershell
# Vérifier la version PC :
frida --version    # → 16.5.9

# Le fichier téléchargé doit être :
# frida-server-16.5.9-android-arm64.xz
```

---

## 2.4 — Pousser et lancer frida-server

```powershell
# Pousser le binaire sur l'appareil
adb push frida-server /data/local/tmp/

# Rendre exécutable
adb shell chmod 755 /data/local/tmp/frida-server

# Lancer frida-server (en arrière-plan)
adb shell "/data/local/tmp/frida-server -l 0.0.0.0 &"
```

> **Note** : `-l 0.0.0.0` permet l'écoute sur toutes les interfaces. En USB direct, vous pouvez aussi utiliser `-l 127.0.0.1` (plus sécurisé).

---

## 2.5 — Port forwarding (optionnel)

Si Frida/Objection ne détecte pas l'appareil, activez le port forwarding :

```powershell
adb forward tcp:27042 tcp:27042
adb forward tcp:27043 tcp:27043
```

| Port | Usage |
|------|-------|
| `27042` | Canal de contrôle Frida |
| `27043` | Premier port de session |

---

## 2.6 — Vérification

```powershell
# Lister les apps avec Frida
frida-ps -Uai
```

### Résultat attendu

```
  PID  Name                    Identifier
-----  ----------------------  ----------------------------------
 1234  Chrome                  com.android.chrome
 5678  Settings                com.android.settings
  ...  ...                     ...
```

> Si `frida-ps -Uai` échoue :
> 1. Vérifiez que frida-server tourne : `adb shell ps | grep frida`
> 2. Vérifiez les versions : `frida --version` vs version du serveur
> 3. Essayez le port forwarding (section 2.5)

---

## 2.7 — Arrêter frida-server (fin de session)

```powershell
adb shell "kill $(pidof frida-server)"

# Vérifier l'arrêt :
adb shell ps | grep frida    # → aucun résultat
```

---

## Résumé

| Étape | Commande | Résultat attendu |
|-------|---------|------------------|
| ADB | `adb devices` | `device` |
| CPU | `adb shell getprop ro.product.cpu.abi` | `arm64-v8a` |
| Push | `adb push frida-server /data/local/tmp/` | Transfert OK |
| Chmod | `adb shell chmod 755 ...` | Permissions OK |
| Start | `adb shell "/data/local/tmp/frida-server -l 0.0.0.0 &"` | Serveur lancé |
| Test | `frida-ps -Uai` | Liste des apps |

> ✅ frida-server est opérationnel. Passer à l'étape 3 (Proxy + CA).
