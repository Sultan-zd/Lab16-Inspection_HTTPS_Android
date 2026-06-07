# 📖 Référence complète des commandes Objection

## Commandes SSL Pinning

| Commande | Description |
|----------|-------------|
| `android sslpinning disable` | Désactive le SSL pinning (TrustManager + OkHttp + Conscrypt) |
| `help android sslpinning` | Aide sur les commandes SSL pinning |

---

## Commandes de hooking

### Recherche de classes

```
android hooking search classes ssl
android hooking search classes trust
android hooking search classes pin
android hooking search classes okhttp
android hooking search classes conscrypt
android hooking search classes certificate
android hooking search classes verify
```

### Lister les classes et méthodes

```
android hooking list classes                                # Toutes les classes chargées
android hooking list class_methods javax.net.ssl.SSLContext  # Méthodes d'une classe
android hooking list class_methods okhttp3.CertificatePinner
```

### Surveiller les appels

```
android hooking watch class javax.net.ssl.SSLContext
android hooking watch class_method javax.net.ssl.SSLContext.init --dump-args --dump-return
android hooking watch class okhttp3.CertificatePinner
```

---

## Commandes d'exploration

| Commande | Description |
|----------|-------------|
| `env` | Chemins de l'application (data, cache, files, etc.) |
| `android hooking list activities` | Lister les activités de l'app |
| `android hooking list services` | Lister les services |
| `android hooking list receivers` | Lister les receivers |
| `android intent launch_activity <activity>` | Lancer une activité spécifique |

---

## Commandes réseau

| Commande | Description |
|----------|-------------|
| `android sslpinning disable` | Bypass SSL pinning |
| `android proxy set <host> <port>` | Configurer un proxy pour l'app |

---

## Commandes système

| Commande | Description |
|----------|-------------|
| `android root disable` | Désactiver la détection de root |
| `android root simulate` | Simuler un environnement rooté |

---

## Gestion de session

| Commande | Description |
|----------|-------------|
| `jobs list` | Lister les hooks/jobs actifs |
| `jobs kill <id>` | Arrêter un job spécifique |
| `reconnect` | Reconnecter à l'agent |
| `exit` | Quitter Objection |
| `ping` | Tester la connexion à l'agent |

---

## Commandes fichiers

| Commande | Description |
|----------|-------------|
| `file download <remote> <local>` | Télécharger un fichier de l'appareil |
| `file upload <local> <remote>` | Uploader un fichier vers l'appareil |
| `file cat <remote>` | Afficher le contenu d'un fichier |
| `ls` | Lister les fichiers du répertoire courant |
| `pwd` | Répertoire courant |

---

## Commandes mémoire

| Commande | Description |
|----------|-------------|
| `memory dump all <file>` | Dump complet de la mémoire |
| `memory list modules` | Lister les modules chargés |
| `memory list exports <module>` | Exports d'un module |
| `memory search "<string>"` | Chercher une chaîne en mémoire |

---

## Options de lancement

| Flag | Description |
|------|-------------|
| `-g <package>` | Package cible |
| `explore` | Ouvrir le shell interactif |
| `--startup-command "<cmd>"` | Commande à exécuter au démarrage |
| `-N` | Pas de connexion USB (réseau) |
| `-h <host>` | Hôte Frida (si réseau) |
| `-p <port>` | Port Frida (si réseau) |
| `--serial <serial>` | Serial ADB de l'appareil |

### Exemples

```powershell
# Spawn avec bypass SSL
objection -g com.example.app explore --startup-command "android sslpinning disable"

# Appareil spécifique
objection --serial XXXXX -g com.example.app explore

# Connexion réseau (pas USB)
objection -N -h 192.168.1.50 -p 27042 -g com.example.app explore
```
