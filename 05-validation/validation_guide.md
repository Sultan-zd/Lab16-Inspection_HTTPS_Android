# ✅ Étape 5 — Validation de la capture du trafic HTTPS

## Objectif

Confirmer que le bypass SSL pinning fonctionne en vérifiant que le trafic HTTPS de l'application apparaît en clair dans le proxy (Burp Suite ou mitmproxy).

---

## 5.1 — Générer du trafic dans l'app

Avec Objection actif et le SSL pinning désactivé, naviguez dans l'app cible :

1. **Écran de login** — entrez des identifiants de test
2. **Écran principal** — parcourez les menus et fonctionnalités
3. **Chargement de données** — ouvrez des listes, profils, paramètres
4. **Recherche** — utilisez la fonction de recherche si disponible
5. **Actions API** — toute action qui déclenche une requête réseau

> L'objectif est de provoquer le maximum de requêtes HTTPS.

---

## 5.2 — Vérifier dans le proxy

### Burp Suite

1. Onglet **Proxy** > **HTTP history**
2. Filtrer par le domaine de l'app (ex: `api.example.com`)
3. Vérifier :

| Élément | Ce que vous devez voir |
|---------|----------------------|
| **Méthode** | GET, POST, PUT, etc. |
| **URL** | Chemin complet de l'API (ex: `/api/v1/users`) |
| **Host** | Domaine de l'API de l'app |
| **Headers** | Authorization, Content-Type, User-Agent, etc. |
| **Request Body** | JSON/form data si POST/PUT |
| **Response** | Code 200, JSON/HTML de réponse |

### mitmproxy

1. Les requêtes apparaissent en temps réel dans le terminal
2. Naviguer avec les flèches ↑↓ et Entrée pour voir les détails
3. En mode `mitmweb`, ouvrir http://127.0.0.1:8081

---

## 5.3 — Points de validation

### ✅ Succès — le bypass fonctionne

| Critère | Attendu | Status |
|---------|---------|--------|
| Requêtes HTTPS visibles | URL + headers lisibles | ✅ |
| Réponses visibles | Corps de réponse en clair | ✅ |
| Pas d'erreur SSL côté app | L'app fonctionne normalement | ✅ |
| Logs Objection | `TrustManager bypass applied` | ✅ |
| Logs Objection | `CertificatePinner bypass applied` | ✅ |

### ❌ Échec — diagnostic nécessaire

| Symptôme | Cause probable | Solution |
|----------|---------------|---------|
| Pas de requêtes dans le proxy | Proxy mal configuré | Vérifier IP/port du proxy Wi-Fi |
| Erreur SSL dans l'app | CA non installée | Installer le certificat (Étape 3) |
| Erreur de connexion | Proxy non joignable | Même réseau + port ouvert |
| Requêtes HTTP mais pas HTTPS | Bypass non appliqué | Relancer `android sslpinning disable` |
| L'app crash | Protection anti-Frida | Essayer attach au lieu de spawn |

---

## 5.4 — Surveiller la console Objection

Pendant que vous naviguez dans l'app, la console Objection affiche les hooks déclenchés :

```
(agent) [ssl-pinning-bypass] TrustManager bypass applied
(agent) [ssl-pinning-bypass] OkHTTP CertificatePinner bypass applied
```

> Si de nouveaux messages apparaissent, cela confirme que les hooks sont actifs et que l'app tente de vérifier les certificats.

### Commandes de diagnostic dans le shell Objection

```
# Vérifier les jobs actifs (hooks en cours)
jobs list

# Chercher les classes réseau chargées
android hooking search classes okhttp
android hooking search classes conscrypt
android hooking search classes trust

# Si aucun résultat, naviguer dans l'app puis relancer :
android sslpinning disable
```

---

## 5.5 — Livrables de validation

Si c'est un exercice / TP, capturez les preuves suivantes :

| # | Livrable | Commande / Action |
|---|---------|-------------------|
| 1 | Version Objection | `objection --version` |
| 2 | Version Frida | `frida --version` |
| 3 | Liste apps | `frida-ps -Uai` |
| 4 | Commande de lancement | Copier la commande spawn ou attach utilisée |
| 5 | Logs Objection | Capture d'écran de la console avec les messages de bypass |
| 6 | Capture proxy | Screenshot du proxy montrant une requête HTTPS de l'app |
| 7 | Réponse API | Screenshot du body de réponse en clair |

---

## 5.6 — Nettoyage post-validation

```powershell
# 1. Quitter Objection
exit

# 2. Arrêter frida-server
adb shell "kill $(pidof frida-server)"

# 3. Retirer le certificat CA
#    Paramètres > Sécurité > Identifiants de confiance > Utilisateur > Supprimer

# 4. Réinitialiser le proxy Wi-Fi
#    Paramètres > Wi-Fi > Modifier > Proxy > Aucun

# 5. Arrêter le proxy (Burp/mitmproxy)

# 6. (Optionnel) Désactiver le débogage USB
```

> ⚠️ **IMPORTANT** : ne laissez jamais une CA de proxy installée sur un appareil utilisé au quotidien. Cela expose l'appareil à des attaques MITM.

---

## Résumé

```
┌─────────────────────────────────────────────────────────────┐
│                    VALIDATION RÉUSSIE                        │
│                                                             │
│  ✅ Objection actif avec SSL pinning désactivé              │
│  ✅ Requêtes HTTPS de l'app visibles dans le proxy          │
│  ✅ Réponses API lisibles (JSON, headers, etc.)             │
│  ✅ L'app fonctionne sans erreur de connexion               │
│  ✅ Console Objection affiche les hooks déclenchés          │
│  ✅ Captures d'écran enregistrées pour les livrables        │
│  ✅ Nettoyage effectué (CA retirée, frida-server arrêté)   │
└─────────────────────────────────────────────────────────────┘
```
