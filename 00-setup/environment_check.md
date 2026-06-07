# ⚙️ Vérification de l'environnement — Lab 16

> Checklist à exécuter **avant** de commencer le lab.

---

## 1. Python et pip

```powershell
python --version          # ✅ Python 3.8+ requis (recommandé 3.11+)
pip --version             # ✅ pip 24.x
```

> **Windows** : si `python` n'est pas reconnu, vérifiez le PATH ou utilisez `py --version`.

---

## 2. ADB (Android Debug Bridge)

```powershell
adb version               # ✅ Android Debug Bridge version 1.0.41
```

> **Installation** : [Android Platform Tools](https://developer.android.com/tools/releases/platform-tools)  
> Décompresser et ajouter le dossier au PATH.

---

## 3. Appareil Android

```powershell
adb devices               # ✅ → <serial>   device
```

### Prérequis appareil

- [x] Android 8.0+ (API 26+)
- [x] **Options développeur** activées
- [x] **Débogage USB** activé
- [x] Câble USB connecté
- [x] Empreinte ADB acceptée sur l'appareil

> **Activer les Options développeur** : Paramètres > À propos du téléphone > Numéro de build (appuyer 7 fois).

---

## 4. Architecture CPU

```powershell
adb shell getprop ro.product.cpu.abi    # ✅ → arm64-v8a (ou armeabi-v7a, x86, x86_64)
```

> Cette information est nécessaire pour télécharger le bon binaire `frida-server`.

---

## 5. Frida (client PC)

```powershell
frida --version                                    # ✅ 16.5.9
python -c "import frida; print(frida.__version__)" # ✅ 16.5.9
```

---

## 6. Objection

```powershell
objection --version        # ✅ 1.11.0+
```

> **Windows** : si `objection` n'est pas reconnu, ajoutez le dossier Scripts au PATH :
> ```
> %USERPROFILE%\AppData\Roaming\Python\Python311\Scripts
> ```

---

## 7. Proxy TLS

### Burp Suite

- [x] Burp Suite installé (Community ou Pro)
- [x] Proxy listener configuré (ex: `0.0.0.0:8080`)

### mitmproxy (alternative)

```powershell
mitmproxy --version        # ✅ mitmproxy 10.x
```

---

## 8. Connectivité réseau

- [x] PC et téléphone sur le **même réseau Wi-Fi**
- [x] IP du PC notée (ex: `192.168.X.Y`)
- [x] Port proxy noté (ex: `8080`)
- [x] Pas de pare-feu bloquant le port proxy

```powershell
# Vérifier l'IP du PC :
ipconfig                   # Windows
# ou
ifconfig                   # macOS/Linux
```

---

## Résumé des versions

| Composant | Version attendue | Commande de vérification |
|-----------|-----------------|-------------------------|
| Python | 3.8+ | `python --version` |
| pip | 24.x | `pip --version` |
| ADB | 1.0.41 | `adb version` |
| Frida | 16.5.9 | `frida --version` |
| Objection | 1.11.0+ | `objection --version` |
| Burp / mitmproxy | Latest / 10.x | GUI / `mitmproxy --version` |

---

## ✅ Validation

Tous les composants ci-dessus doivent retourner un résultat valide avant de passer à l'Étape 1.

> ⚠️ **CRITIQUE** : la version de `frida` (PC) doit correspondre **exactement** à la version du `frida-server` sur l'appareil.
