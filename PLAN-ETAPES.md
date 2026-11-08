# Plan étape par étape — Réalisation Malaïka Freight

**Projet :** Malaïka Freight — plateforme fret Madrid ↔ Kinshasa  
**Objectif :** Guide d’exécution séquentiel — une étape à la fois, dans l’ordre.  
**Focus initial :** Lot A (lancement commercial)

> **Documents liés :** [Plan détaillé](./PLAN-DETAILLE.md) · [Documentation](./DOCUMENTATION-MALAIKA-FREIGHT.md) · [Architecture](./ARCHITECTURE.md) · [Structure](./STRUCTURE-ARBORESCENCE.md)

---

## Comment utiliser ce document

1. Suivre les étapes **dans l’ordre** (ne pas sauter).
2. Cocher chaque case `[ ]` une fois validée.
3. Faire un **commit Git** à la fin de chaque bloc.
4. Tester localement (Laragon / `php artisan serve`) après chaque bloc important.
5. Noter les blocages et décisions fondatrice dans le journal projet.

**Durée estimée Lot A :** 8 à 12 semaines · **~55 étapes**

---

## Vue rapide

| Bloc | Étapes | Contenu | Lot |
|------|--------|---------|-----|
| A | 1–5 | Préparation & bootstrap | A |
| B | 6–12 | Base de données & modèles | A |
| C | 13–18 | Auth, rôles, agences | A |
| D | 19–26 | Clients, réservations, colis, pièces | A |
| E | 27–33 | Tarifs, factures, étiquettes QR | A |
| F | 34–38 | Tracking & suivi public | A |
| G | 39–43 | Site public | A |
| H | 44–48 | Dashboards & UX rôles | A |
| I | 49–52 | Mails, notifications, audit | A |
| J | 53–55 | API, tests, recette / release | A |
| K | 56+ | Lot B et suivants | B/C/D |

---

# BLOC A — Préparation (Étapes 1 à 5)

## Étape 1 — Figurer le Lot A

**But :** Éviter le scope creep.

- [ ] Relire le CDC + documentation complète
- [ ] Lister explicitement ce qui est **in** / **out** du Lot A
- [ ] Noter les décisions ouvertes (prix, domaine, paiements…)
- [ ] Valider avec la fondatrice si possible

**Commit :** `docs: cadrage Lot A et décisions ouvertes`

---

## Étape 2 — Environnement Laragon

**But :** Stack locale opérationnelle.

- [ ] PHP 8.3+, Composer, Node, MySQL actifs
- [ ] Créer la base `malaika_freight`
- [ ] Vérifier `git` et accès au dépôt privé

**Vérification :** connexion MySQL OK ; dépôt `main` à jour.

---

## Étape 3 — Bootstrap Laravel

**But :** Application vide propre dans le repo.

```bash
composer create-project laravel/laravel .
# ou équivalent selon état du dépôt
cp .env.example .env
php artisan key:generate
```

- [ ] `.env` pointant vers `malaika_freight`
- [ ] `php artisan serve` (ou vhost Laragon) OK
- [ ] `.gitignore` couvre `.env`, `vendor`, `node_modules`

**Commit :** `chore: bootstrap Laravel Malaïka Freight`

---

## Étape 4 — Packages de base

```bash
composer require laravel/breeze --dev
php artisan breeze:install blade
composer require laravel/sanctum barryvdh/laravel-dompdf
# package QR / barcode selon choix
npm install && npm run build
```

- [ ] `/login` accessible
- [ ] Assets compilés

**Commit :** `chore: Breeze, Sanctum, DomPDF`

---

## Étape 5 — Design tokens & layouts squelette

**But :** Reprendre l’identité maquette (navy `#11263b`, gold `#c49b58`).

- [ ] Variables CSS / Tailwind theme
- [ ] Layouts : `public`, `app` (client), `admin`
- [ ] Page placeholder dashboard

**Commit :** `feat(ui): layouts et identité visuelle Malaïka`

---

# BLOC B — Base de données (Étapes 6 à 12)

## Étape 6 — Agences & utilisateurs étendus

- [ ] Migration `agencies`
- [ ] Champs `users` : `role`, `agency_id`, `is_active`, téléphone
- [ ] Seed 2 agences (Madrid, Kinshasa)

**Commit :** `feat(db): agences et champs utilisateurs`

---

## Étape 7 — Clients & adresses

- [ ] Tables `customers`, `addresses`
- [ ] Relations Eloquent
- [ ] Seeder clients fictifs (aucune donnée réelle)

---

## Étape 8 — Réservations

- [ ] Table `reservations` + statuts
- [ ] Référence unique
- [ ] Consentements stockés

---

## Étape 9 — Colis & pièces

- [ ] Tables `parcels`, `parcel_pieces`
- [ ] Poids estimé / réel / facturable
- [ ] `tracking_token` unique opaque
- [ ] Référence pièce unique

---

## Étape 10 — Tarifs, factures, paiements

- [ ] `price_rules`
- [ ] `invoices`, `invoice_lines`
- [ ] `payments` (partiels autorisés)
- [ ] Devises EUR / USD

---

## Étape 11 — Tracking & audit

- [ ] `tracking_events`
- [ ] `audit_logs`
- [ ] `feature_flags`

---

## Étape 12 — Seeders globaux & reset propre

- [ ] `DatabaseSeeder` complet
- [ ] Comptes démo par rôle
- [ ] `migrate:fresh --seed` documenté

**Commit :** `feat(db): schéma Lot A complet et seeders`

**Vérification :** relations Eloquent testées en tinker.

---

# BLOC C — Auth & permissions (Étapes 13 à 18)

## Étape 13 — Enum rôles

- [ ] Enum `UserRole` (superadmin, responsable, reception, livraison, comptable, client)
- [ ] Helper `hasRole` / `canAccessAgency`

---

## Étape 14 — Middleware

- [ ] `EnsureUserIsActive`
- [ ] `EnsureRole`
- [ ] `EnsureAgencyScope`

---

## Étape 15 — Policies

- [ ] `ParcelPolicy`, `InvoicePolicy`, `CustomerPolicy`
- [ ] Tests Feature : refus cross-agence

---

## Étape 16 — Redirections post-login

- [ ] Mapping rôle → dashboard
- [ ] Menu latéral selon permissions

---

## Étape 17 — Sécurité sessions

- [ ] Timeout raisonnable
- [ ] Protection CSRF
- [ ] Rate limit login

---

## Étape 18 — Audit auth

- [ ] Log connexion / échec / logout

**Commit :** `feat(auth): middlewares, policies et audit connexions`

---

# BLOC D — Opérations colis (Étapes 19 à 26)

## Étape 19 — CRUD clients admin

- [ ] Resource controller + FormRequests
- [ ] Recherche / pagination

---

## Étape 20 — Création réservation (back-office)

- [ ] Formulaire agent
- [ ] Statut `pre_reserve`

---

## Étape 21 — Conversion réservation → colis

- [ ] Action métier transactionnelle
- [ ] Copie des parties / poids estimé

---

## Étape 22 — Réception & pesée

- [ ] Saisie poids réel, contenu déclaré, carton
- [ ] Journalisation écarts tarifaires

---

## Étape 23 — Multi-pièces

- [ ] Création N pièces
- [ ] Affichage `i/n`

---

## Étape 24 — Machine à états

- [ ] Transitions autorisées
- [ ] Blocage transitions illégales

---

## Étape 25 — Upload photo colis

- [ ] Validation mime/size
- [ ] Stockage `storage/app/public`

---

## Étape 26 — Liste / filtres colis

- [ ] Filtres agence, statut, dates, recherche référence

**Commit :** `feat(ops): cycle réservation → réception multi-pièces`

---

# BLOC E — Documents (Étapes 27 à 33)

## Étape 27 — Moteur de tarification

- [ ] Calcul selon `price_rules`
- [ ] Frais extra
- [ ] Total + solde

---

## Étape 28 — Émission facture

- [ ] Numérotation unique
- [ ] Immutabilité post-émission
- [ ] Lignes détaillées

---

## Étape 29 — Paiements partiels

- [ ] Encaissement
- [ ] Mise à jour `amount_paid` / `balance_due`

---

## Étape 30 — PDF facture

- [ ] Template Blade PDF
- [ ] Mentions légales configurables

---

## Étape 31 — Génération jeton & QR

- [ ] Token opaque
- [ ] URL suivi
- [ ] Image QR

---

## Étape 32 — PDF étiquette 100×150

- [ ] Layout thermique
- [ ] Code 128
- [ ] Option A4

---

## Étape 33 — Réimpression versionnée

- [ ] Incrément `label_version`
- [ ] Historique impression

**Commit :** `feat(docs): facturation et étiquettes QR`

---

# BLOC F — Tracking (Étapes 34 à 38)

## Étape 34 — Enregistrer un event

- [ ] Endpoint / action agent
- [ ] Preuve optionnelle (photo)

---

## Étape 35 — Timeline UI

- [ ] Composant timeline admin + client

---

## Étape 36 — Page suivi public

- [ ] Route `/suivi/{token}`
- [ ] Données minimales uniquement

---

## Étape 37 — Rate limiting suivi

- [ ] Protection énumération

---

## Étape 38 — Tests privacy

- [ ] Assert absence téléphone / prix dans HTML public

**Commit :** `feat(tracking): suivi public sécurisé`

---

# BLOC G — Site public (Étapes 39 à 43)

## Étape 39 — Pages marketing

- [ ] Accueil, services, itinéraires, offres, FAQ, contact, légal

---

## Étape 40 — Estimateur

- [ ] Calcul indicatif
- [ ] Disclaimer “estimatif”

---

## Étape 41 — Formulaire pré-réservation

- [ ] Validation + honeypot / throttle
- [ ] Création `reservation`

---

## Étape 42 — Confirmation

- [ ] Écran succès + email (si canal)

---

## Étape 43 — Responsive QA

- [ ] Mobile / desktop

**Commit :** `feat(public): acquisition et pré-réservation`

---

# BLOC H — Dashboards (Étapes 44 à 48)

## Étape 44 — KPIs globaux

- [ ] Volumes, poids, transit, livrés, incidents

---

## Étape 45 — Finance séparée EUR/USD

- [ ] Cartes encaissements distinctes

---

## Étape 46 — Filtres

- [ ] Agence, période, corridor

---

## Étape 47 — Vue réception

- [ ] File du jour, pré-réservations

---

## Étape 48 — Vue livraison

- [ ] Disponibles / en cours / incidents

**Commit :** `feat(dashboard): pilotage opérationnel Lot A`

---

# BLOC I — Communications (Étapes 49 à 52)

## Étape 49 — Mailables essentiels

- [ ] Pré-réservation, facture, changement statut, livraison

---

## Étape 50 — Queues

- [ ] `queue:work` documenté

---

## Étape 51 — Notifications in-app

- [ ] Cloche admin

---

## Étape 52 — Audit métier

- [ ] Paiements, changements tarif, impressions

**Commit :** `feat(comms): emails notifications audit`

---

# BLOC J — API, tests, release (Étapes 53 à 55)

## Étape 53 — API Sanctum Lot A

- [ ] Auth token
- [ ] Parcels, events, tracking, invoices (selon besoin)
- [ ] Collection Postman

---

## Étape 54 — Batterie de tests Feature

- [ ] Isolation agence
- [ ] Facture + solde
- [ ] Privacy tracking
- [ ] Transitions statut

---

## Étape 55 — Recette & déploiement

- [ ] Checklist CDC §12
- [ ] Backup / restore testés
- [ ] Remise accès société
- [ ] Tag release `v0.1.0-lot-a`

**Commit :** `chore(release): Lot A prêt pour recette`

---

# BLOC K — Suite (aperçu)

| Étape | Contenu | Lot |
|-------|---------|-----|
| 56 | Manifestes / shipments | B |
| 57 | Association colis ↔ départ | B |
| 58 | Stocks emballages | B |
| 59 | Caisses & dépenses | B |
| 60 | Preuves de livraison avancées | B |
| 61 | Reporting comptable | C |
| 62 | Documents physiques | C |
| 63 | Corridor inverse prod | C |
| 64 | Go/no-go juridique transfert | D |

---

## Comptes de démonstration (à seeder)

| Rôle | Email indicatif | Mot de passe démo |
|------|-----------------|-------------------|
| Superadmin | admin@malaika.test | (généré / .env local) |
| Resp. Madrid | madrid@malaika.test | idem |
| Resp. Kinshasa | kinshasa@malaika.test | idem |
| Réception | reception@malaika.test | idem |
| Livraison | livraison@malaika.test | idem |
| Comptable | compta@malaika.test | idem |
| Client | client@malaika.test | idem |

> **Jamais** de vrais clients ni secrets de production dans les seeders.

---

*Plan étapes Malaïka Freight · v1.0 · Novembre 2026*
