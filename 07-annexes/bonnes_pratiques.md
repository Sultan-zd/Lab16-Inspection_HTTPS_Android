# 📋 Bonnes pratiques — Lab 16

## Sécurité et éthique

| Pratique | Raison |
|----------|--------|
| N'inspectez que vos propres apps / apps autorisées | Cadre légal obligatoire |
| Ne capturez pas de données sensibles réelles | Protection de la vie privée |
| Retirez la CA du proxy après les tests | Éviter les attaques MITM non intentionnelles |
| Arrêtez frida-server après la session | Réduire la surface d'attaque |
| Désactivez le débogage USB après les tests | Sécurité de l'appareil |

---

## Workflow optimal

| Pratique | Raison |
|----------|--------|
| Commencez par le **spawn** | Patcher avant le premier check SSL |
| Si le spawn crash, basculez en **attach** | Approche progressive et stable |
| Ne surchargez pas `--startup-command` | Une seule commande pour isoler les problèmes |
| Naviguez dans l'app après le bypass | Déclencher les requêtes réseau |
| Relancez `android sslpinning disable` si nécessaire | Certaines classes sont chargées tardivement |

---

## Gestion des versions

| Pratique | Raison |
|----------|--------|
| Versions Frida alignées (client = server) | Éviter les erreurs de protocole incompatible |
| Mettre à jour Objection et Frida ensemble | Éviter les incompatibilités |
| Tester `frida-ps -Uai` avant Objection | S'assurer que la connexion fonctionne |
| Noter les versions dans `analyse_info.txt` | Traçabilité et reproductibilité |

---

## Proxy

| Pratique | Raison |
|----------|--------|
| Désactivez l'intercept (Burp) pendant les tests | Laisser le flux passer sans interruption |
| Utilisez `adb reverse` pour la connexion USB | Plus fiable que le proxy Wi-Fi |
| Vérifiez avec le navigateur d'abord | Isoler les problèmes proxy vs SSL pinning |
| Filtrez par domaine dans le proxy | Réduire le bruit |

---

## Logs et documentation

| Pratique | Raison |
|----------|--------|
| Minimisez les logs en session réelle | Certaines apps détectent un IO verbeux |
| Capturez les preuves avant le nettoyage | Livrables pour le rapport |
| Journalisez les commandes dans `commands.log` | Reproductibilité du lab |
| Ne stockez pas de données sensibles interceptées | Éthique et conformité |
