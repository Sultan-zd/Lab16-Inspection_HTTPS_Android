# 📋 Checklist de clôture — Lab 16

> **Lab** : Inspection HTTPS Android — SSL Pinning Bypass avec Objection + Proxy  
> **Date** : 2026-06-08  
> **Analyste** : Étudiant Sécurité

---

## ✅ Environnement

- [x] Python 3.8+ installé et fonctionnel
- [x] pip à jour
- [x] ADB installé et appareil reconnu (`device`)
- [x] Frida installé (`frida --version` = 16.5.9)
- [x] Objection installé (`objection --version` = 1.11.0+)
- [x] frida-server déployé et lancé sur l'appareil
- [x] `frida-ps -Uai` retourne la liste des apps

## ✅ Proxy TLS

- [x] Proxy (Burp / mitmproxy) installé et configuré sur le PC
- [x] Listener actif sur le bon port (ex: `0.0.0.0:8080`)
- [x] Certificat CA exporté et installé sur l'appareil
- [x] Proxy Wi-Fi configuré sur le téléphone (ou `adb reverse`)
- [x] Navigation HTTP/HTTPS basique visible dans le proxy (test navigateur)

## ✅ Objection — SSL Pinning Bypass

- [x] Package cible identifié (`frida-ps -Uai`)
- [x] Mode **Spawn** testé :
  - `objection -g <pkg> explore --startup-command "android sslpinning disable"`
- [x] Mode **Attach** testé (fallback) :
  - `objection -g <pkg> explore` → `android sslpinning disable`
- [x] Logs de bypass observés :
  - `TrustManager bypass applied`
  - `OkHTTP CertificatePinner bypass applied`

## ✅ Validation

- [x] Requêtes HTTPS de l'app visibles en clair dans le proxy
- [x] URL, headers et body lisibles dans le proxy
- [x] Réponses API visibles sans erreur SSL côté app
- [x] L'app fonctionne normalement après le bypass
- [x] Console Objection affiche les hooks déclenchés

## ✅ Diagnostic (le cas échéant)

- [x] Commandes de recherche de classes exécutées (`search classes ssl/trust/pin`)
- [x] Cas du pinning natif identifié (renvoi vers Lab 15 si nécessaire)
- [x] Cas de l'obfuscation traité (`search classes` + noms alternatifs)

## ✅ Livrables

- [x] Capture `objection --version` et `frida --version`
- [x] Capture `frida-ps -Uai`
- [x] Commande exacte de lancement (spawn ou attach) documentée
- [x] Capture du proxy montrant des requêtes HTTPS de l'app
- [x] Tous les guides rédigés (Markdown)
- [x] `commands.log` renseigné
- [x] `analyse_info.txt` renseigné
- [x] `README.md` rédigé

## ✅ Nettoyage

- [x] Session Objection fermée (`exit`)
- [x] frida-server arrêté sur l'appareil
- [x] Certificat CA retiré de l'appareil
- [x] Proxy Wi-Fi désactivé sur le téléphone
- [x] Proxy arrêté sur le PC
- [x] Débogage USB désactivé (si nécessaire)

---

**Signature** : _Étudiant Sécurité — Lab 16 complété le 2026-06-08_

> ⚠️ Ce lab a été réalisé dans un cadre **strictement pédagogique et défensif**.
