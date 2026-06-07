# 📋 Périmètre d'analyse — Lab 16

> **Lab** : Inspection HTTPS Android — SSL Pinning Bypass avec Objection + Proxy  
> **Date** : 2026-06-08  
> **Analyste** : Étudiant Sécurité

---

## Cadre d'intervention

| Élément | Détail |
|---------|--------|
| **Type** | Audit sécurité réseau — Inspection TLS/HTTPS |
| **Méthode** | Instrumentation dynamique (Objection / Frida) |
| **Cible** | Application Android avec SSL Pinning |
| **Appareil** | Android 8+ (Débogage USB activé) — appareil personnel |
| **Proxy** | Burp Suite Community/Pro ou mitmproxy |
| **Référentiel** | OWASP MASVS v2.0 — MASVS-NETWORK |

---

## Autorisations

| Condition | Statut |
|-----------|--------|
| Appareil personnel ou prêté avec autorisation | ✅ |
| Application de test (propriétaire ou lab) | ✅ |
| Réseau isolé / contrôlé | ✅ |
| Aucune donnée de production interceptée | ✅ |
| Cadre pédagogique vérifié | ✅ |

---

## Périmètre technique

### ✅ Inclus

- Installation et configuration d'Objection (surcouche Frida)
- Déploiement de frida-server sur l'appareil Android
- Configuration d'un proxy TLS (Burp Suite ou mitmproxy)
- Installation du certificat CA sur l'appareil
- Désactivation du SSL pinning via `android sslpinning disable`
- Capture et validation du trafic HTTPS en clair
- Diagnostic et dépannage des cas courants

### ❌ Exclus

- Rétro-ingénierie complète de l'APK
- Bypass de pinning natif (BoringSSL/OpenSSL) — couvert en Lab 15
- Écriture de scripts Frida JavaScript manuels — couvert en Lab 15
- Interception de trafic d'applications de production
- Extraction ou stockage de données sensibles réelles

---

## Différence avec le Lab 15

| Aspect | Lab 15 | Lab 16 |
|--------|--------|--------|
| **Focus** | Multi-couches (scripts Frida JS) | Objection CLI uniquement |
| **Scripts** | 5 scripts JS (universel, OkHttp, WebView, natif, enum) | Aucun script manuel |
| **Bypass natif** | ✅ (BoringSSL hooks) | ❌ (Java seulement) |
| **Complexité** | Intermédiaire-Avancé | Intermédiaire |
| **Durée** | 3h30 | 2h30 |
| **Objectif** | Comprendre chaque couche SSL | Bypass rapide et efficace |

---

## Engagements éthiques

- ⚠️ N'appliquer ces techniques que dans un cadre légal
- 🔒 Aucune donnée réelle capturée ou stockée
- 🧹 Nettoyage complet après tests (CA, frida-server, proxy)
- 📝 Documentation à des fins pédagogiques uniquement

---

**Signature** : _Étudiant Sécurité — Périmètre validé le 2026-06-08_
