# Architecture complète — Malaïka Freight

**Projet :** Plateforme fret Madrid ↔ Kinshasa  
**Version :** 1.0  
**Style :** Monolithe Laravel modulaire (évolutif vers API-first)

---

## Documents liés

- [Documentation produit](./DOCUMENTATION-MALAIKA-FREIGHT.md)
- [Structure](./STRUCTURE-ARBORESCENCE.md)
- [Diagrammes Mermaid](./diagrams/)

---

## 1. Principes d’architecture

1. **Un produit, trois surfaces** : public, client, admin — même codebase.
2. **Monolithe modulaire** : rapidité de livraison Lot A ; frontières claires (Actions/Services).
3. **Multi-agence dès le départ** : chaque donnée opérationnelle porte un contexte d’agence.
4. **Sécurité par défaut** : policies + middleware ; suivi public minimal.
5. **Devises isolées** : EUR/USD comme dimensions financières distinctes.
6. **Feature flags** : Documents / Transfert / corridor inverse désactivables.
7. **Auditabilité** : actions sensibles journalisées côté serveur.
8. **Évolutif** : API Sanctum pour mobile / partenaires sans réécriture totale.

---

## 2. Architecture logique (couches)

```
┌─────────────────────────────────────────────────────────┐
│ Présentation (Blade / Tailwind / Alpine)                │
│  Public · Client · Admin · PDF · Emails                 │
├─────────────────────────────────────────────────────────┤
│ Interface HTTP (Controllers · FormRequests · Resources) │
├─────────────────────────────────────────────────────────┤
│ Application (Actions · Services · Jobs · Notifications) │
├─────────────────────────────────────────────────────────┤
│ Domaine (Enums · State machines · Value objects · Rules)│
├─────────────────────────────────────────────────────────┤
│ Infrastructure (Eloquent · Storage · Queue · Mail · QR) │
└─────────────────────────────────────────────────────────┘
```

Voir aussi : [`diagrams/architecture-applicative.mmd`](./diagrams/architecture-applicative.mmd)

---

## 3. Architecture globale du système

Acteurs externes et composants internes :

- Visiteurs / clients (navigateur)
- Agents Madrid & Kinshasa
- Comptable / direction
- SMTP / provider mail
- (Futur) prestataire transport / paiement / mobile money
- MySQL, storage fichiers, queues

Voir : [`diagrams/architecture-globale.mmd`](./diagrams/architecture-globale.mmd)

---

## 4. Découpage applicatif

### 4.1 Bounded contexts (logiques)

| Contexte | Responsabilité |
|----------|----------------|
| **Identity & Access** | Users, rôles, agences, policies |
| **CRM léger** | Customers, adresses, consentements |
| **Booking** | Réservations publiques / internes |
| **Operations** | Parcels, pièces, scans, statuts |
| **Pricing & Billing** | Price rules, invoices, payments |
| **Tracking** | Events, tokens, page publique |
| **Shipping** (Lot B) | Manifestes, consolidation |
| **Inventory** (Lot B) | Stocks emballages |
| **Treasury** (Lot B) | Caisses, dépenses, taux de change |
| **Compliance flags** | Feature flags Documents / Transfer |

### 4.2 Communication interne

- Synchrone : appels Services/Actions dans une requête HTTP
- Asynchrone : Jobs (mails, génération PDF lourde si besoin)
- Événements domaine : `ParcelStatusChanged`, `InvoiceIssued`, `PaymentRecorded`

---

## 5. Architecture des données

### 5.1 Règles

- Clés étrangères strictes
- Unicité : références colis/pièces, numéros facture, `tracking_token`
- Index : `status`, `agency_id`, dates, références
- Transactions atomiques : conversion réservation, paiement + solde, stock (Lot B)

### 5.2 Schéma conceptuel

Voir : [`diagrams/architecture-donnees.mmd`](./diagrams/architecture-donnees.mmd)

### 5.3 Isolation multi-agence

- Colonnes `origin_agency_id` / `destination_agency_id` / `agency_id` selon entité
- Scope Eloquent global ou explicite dans requêtes + Policy
- Dashboard agrégé réservé aux rôles autorisés

---

## 6. Flux métier architectural

Le flux principal traverse Booking → Operations → Billing → Tracking → Delivery.

Voir : [`diagrams/flux-metier-principal.mmd`](./diagrams/flux-metier-principal.mmd)

### Points de contrôle techniques

| Étape métier | Contrôle architecture |
|--------------|----------------------|
| Pré-réservation | Throttle, validation, pas de tarif ferme |
| Réception | Transaction DB, audit poids/tarif |
| Facture | Numéro unique, immutabilité |
| Étiquette | Token opaque, version label |
| Scan | Auth agent + policy agence |
| Suivi public | Données minimales + rate limit |
| Paiement | Update solde atomique |

---

## 7. Sécurité

### 7.1 Authentification

- Sessions web (Breeze)
- Tokens API (Sanctum) pour clients machine
- 2FA optionnelle pour comptes sensibles (recommandée Lot A+/B)

### 7.2 Autorisation

Matrice rôle × agence × action — voir [`diagrams/roles-permissions.mmd`](./diagrams/roles-permissions.mmd)

### 7.3 Données personnelles

- Minimisation sur suivi public
- Chiffrement TLS
- Sauvegardes chiffrées
- Rétention configurable
- Secrets hors Git (`.env`)

### 7.4 Abus

- Rate limiting login / pré-réservation / tracking
- CSRF sur formulaires web
- Validation stricte uploads

---

## 8. Documents PDF & impression

```
Parcel (+ pieces)
   ├─ InvoiceService → PDF facture (A4)
   └─ LabelService  → PDF étiquette (100×150 / A4)
         ├─ QR (URL + token)
         └─ Code 128 (référence pièce)
```

Les PDF sont générés côté serveur à partir de templates Blade versionnés. L’impression thermique doit être prototypée tôt (risque UX matériel).

---

## 9. API

- Versionnage `/api/v1`
- Auth Bearer Sanctum
- Resources JSON homogènes (`data`, `message`, `meta`)
- Idempotence progressive sur paiements / scans (clé idempotency Lot B)

Endpoints cibles : voir documentation §14.

---

## 10. Architecture de déploiement

### Développement

Laragon (Windows) · MySQL local · Mailtrap · `queue:work`

### Production (cible)

```
Internet
   → Reverse proxy (Nginx) + TLS
      → PHP-FPM (Laravel)
         → MySQL
         → Redis (cache/queue) optionnel
         → Object storage / disque pour preuves & PDF
```

Voir : [`diagrams/architecture-deploiement.mmd`](./diagrams/architecture-deploiement.mmd)

### Exigences exploitation

- Backups DB + fichiers planifiés
- Procédure restore testée
- Monitoring basique (uptime, disque, queue)
- Accès admin révocables ; propriété des comptes chez la société

---

## 11. Observabilité

| Signal | Outil / mécanisme |
|--------|-------------------|
| Audit métier | `audit_logs` |
| Erreurs app | Laravel log / provider externe |
| Jobs échoués | `failed_jobs` |
| Métriques ops | Dashboard applicatif |

---

## 12. Évolutions prévues

| Évolution | Impact architecture |
|-----------|---------------------|
| PWA | Service worker + cache lecture suivi |
| App mobile agents | API + auth Sanctum renforcée |
| OHADA | Module Treasury + exports ; pas dans le cœur colis |
| Transfert d’argent | Contexte Compliance isolé ; jamais mélangé au ledger fret |
| Multi-tenant holding | `organizations` déjà prévu dans le modèle |

---

## 13. Non-objectifs d’architecture (Lot A)

- Microservices
- Event sourcing complet
- Comptabilité légale complète
- Paiement mobile production multi-pays sans cadrage
- Exposition publique d’API non authentifiée sur données personnelles

---

## 14. Décision stack (ADR résumé)

**Contexte :** besoin d’un back-office riche, PDF, auth, délais serrés, compétences Laravel existantes (ColisTrack).  
**Décision :** monolithe Laravel modulaire.  
**Conséquences :** livraison rapide Lot A ; API ajoutée progressivement ; découpage futur possible par contexte si charge / équipe grandit.

---

*Architecture Malaïka Freight · v1.0 · Novembre 2026*
