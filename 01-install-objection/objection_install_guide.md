# 🔧 Étape 1 — Installer Objection et Frida côté PC

## Objectif

Installer **Objection** (surcouche CLI de Frida) et **Frida** sur le PC pour pouvoir instrumenter des applications Android à distance.

---

## Qu'est-ce qu'Objection ?

| Aspect | Détail |
|--------|--------|
| **Type** | CLI Python basée sur Frida |
| **Rôle** | Bypass de sécurité mobile (SSL pinning, root detection, etc.) |
| **Avantage** | Pas besoin d'écrire de scripts Frida — commandes prêtes à l'emploi |
| **Auteur** | SensePost (Orange Cyberdefense) |
| **Dépendances** | Python 3.8+, Frida, frida-tools |

---

## Option A : Installation isolée avec pipx (recommandée)

`pipx` installe Objection dans un environnement virtuel isolé, évitant les conflits de dépendances.

```powershell
# 1. Installer pipx
pip install --user pipx

# 2. Ajouter pipx au PATH
pipx ensurepath

# 3. Installer Objection
pipx install objection
```

> **Note** : redémarrez le terminal après `pipx ensurepath` pour que le PATH soit mis à jour.

---

## Option B : Installation via pip classique

```powershell
pip install --upgrade objection frida frida-tools
```

> ⚠️ Cette méthode installe tout dans l'environnement Python global. Risque de conflits avec d'autres packages.

---

## Vérification

```powershell
# Objection
objection --version                                    # → 1.11.0+

# Frida CLI
frida --version                                        # → 16.5.9

# Frida module Python
python -c "import frida; print(frida.__version__)"     # → 16.5.9
```

---

## Résolution de problèmes

### `objection` n'est pas reconnu (Windows)

Le binaire `objection` est installé dans le dossier `Scripts` de Python. Ajoutez-le au PATH :

```powershell
# Trouver le chemin :
python -c "import site; print(site.getusersitepackages())"

# Ajouter au PATH (exemple) :
# %USERPROFILE%\AppData\Roaming\Python\Python311\Scripts
```

Ou ajoutez-le via :
```powershell
$env:PATH += ";$env:USERPROFILE\AppData\Roaming\Python\Python311\Scripts"
```

### `frida` version mismatch

```powershell
# Mettre à jour Frida :
pip install --upgrade frida frida-tools

# Vérifier la cohérence :
frida --version
python -c "import frida; print(frida.__version__)"
# Les deux doivent afficher la même version
```

### Erreur SSL lors de l'installation pip

```powershell
pip install --upgrade objection --trusted-host pypi.org --trusted-host files.pythonhosted.org
```

---

## Résumé

| Composant | Commande d'installation | Vérification |
|-----------|------------------------|-------------|
| pipx | `pip install --user pipx` | `pipx --version` |
| Objection | `pipx install objection` | `objection --version` |
| Frida | installé avec Objection | `frida --version` |
| frida-tools | installé avec Objection | `frida --version` |

> ✅ Une fois Objection installé, Frida et frida-tools sont automatiquement installés comme dépendances.
