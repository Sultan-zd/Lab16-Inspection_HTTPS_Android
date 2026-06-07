# 🔄 Spawn vs Attach — Guide de décision

## Contexte

Objection (et Frida) offrent deux modes d'injection dans un processus Android :

- **Spawn** : Objection relance l'app et injecte les hooks avant l'exécution du code principal
- **Attach** : Objection se connecte à un processus déjà en cours d'exécution

---

## Comparaison détaillée

| Critère | Spawn | Attach |
|---------|-------|--------|
| **Timing** | Avant l'init de l'app | Après le lancement de l'app |
| **Commande** | `objection -g <pkg> explore --startup-command "..."` | `objection -g <pkg> explore` |
| **Couverture** | Tous les checks SSL (y compris init) | Checks post-lancement uniquement |
| **Stabilité** | Peut crasher certaines apps protégées | Plus stable en général |
| **Anti-tampering** | Détectable par certaines protections | Moins détectable |
| **Recommandé** | ✅ En premier choix | 🔄 En fallback |

---

## Arbre de décision

```
                    ┌─────────────────────┐
                    │  Début : Identifier  │
                    │  le package cible    │
                    └──────────┬──────────┘
                               │
                    ┌──────────▼──────────┐
                    │  Essayer le SPAWN   │
                    │  --startup-command  │
                    └──────────┬──────────┘
                               │
                    ┌──────────▼──────────┐
                    │  L'app démarre      │───── OUI ──► ✅ Utiliser le SPAWN
                    │  normalement ?      │
                    └──────────┬──────────┘
                               │ NON
                    ┌──────────▼──────────┐
                    │  Essayer l'ATTACH   │
                    │  (ouvrir l'app      │
                    │   puis attacher)    │
                    └──────────┬──────────┘
                               │
                    ┌──────────▼──────────┐
                    │  Le bypass          │───── OUI ──► ✅ Utiliser l'ATTACH
                    │  fonctionne ?       │
                    └──────────┬──────────┘
                               │ NON
                    ┌──────────▼──────────┐
                    │  Diagnostiquer :    │
                    │  - Anti-Frida ?     │
                    │  - Pinning natif ?  │
                    │  - Obfuscation ?    │
                    └─────────────────────┘
```

---

## Exemples pratiques

### Spawn — App classique (OkHttp standard)

```powershell
objection -g com.example.app explore --startup-command "android sslpinning disable"

# ✅ L'app démarre, les hooks sont appliqués, le trafic est visible dans le proxy
```

### Attach — App avec anti-tampering au démarrage

```powershell
# 1. Ouvrir l'app normalement sur le téléphone
# 2. Attendre que l'app soit entièrement chargée
# 3. Attacher Objection :

objection -g com.example.app explore

# 4. Dans le shell :
android sslpinning disable

# 5. Naviguer dans l'app pour générer du trafic
```

### Cas avancé — Relancer le bypass après chargement tardif

Certaines apps chargent leurs classes réseau **après** l'écran principal (lazy loading). Dans ce cas :

```
com.example.app on (samsung: 13.0) [usb] # android sslpinning disable
# → Pas de hook détecté

# Naviguer dans l'app (ouvrir un écran qui fait des requêtes réseau)

com.example.app on (samsung: 13.0) [usb] # android sslpinning disable
# → (agent) [ssl-pinning-bypass] TrustManager bypass applied ✅
```

---

## Conseils

1. **Commencez toujours par le spawn** — il couvre la majorité des cas
2. **Ne surchargez pas `--startup-command`** — une seule commande pour isoler les problèmes
3. **Si l'app crash**, c'est souvent lié à une protection anti-Frida, pas au SSL bypass lui-même
4. **Avec l'attach**, naviguez d'abord dans l'app, puis lancez le bypass
5. **Relancez la commande** si les classes réseau n'étaient pas encore chargées
