# Structure & arborescence globale — Malaïka Freight

**Projet :** Malaïka Freight  
**Objectif :** Décrire l’organisation cible du dépôt et des modules applicatifs.

---

## 1. Vue logique du produit

```
Malaïka Freight
├── Site public          (acquisition + suivi public)
├── Espace client        (compte, documents, historique)
├── Back-office ops      (réception, scans, livraisons)
├── Back-office finance  (factures, paiements, caisses)
├── API                  (intégrations / futur mobile)
└── Jobs & notifications (emails, audit asynchrone)
```

---

## 2. Arborescence dépôt (cible Laravel)

```
malaika-freight/
├── app/
│   ├── Actions/                    # Cas d'usage métier (ConvertReservation, IssueInvoice…)
│   ├── Domain/
│   │   ├── Enums/                  # ParcelStatus, Currency, UserRole…
│   │   ├── States/                 # Machines à états
│   │   └── ValueObjects/           # Money, Weight, TrackingToken…
│   ├── Http/
│   │   ├── Controllers/
│   │   │   ├── Public/             # Home, Estimate, TrackingPublic…
│   │   │   ├── Client/
│   │   │   ├── Admin/
│   │   │   │   ├── DashboardController.php
│   │   │   │   ├── CustomerController.php
│   │   │   │   ├── ReservationController.php
│   │   │   │   ├── ParcelController.php
│   │   │   │   ├── InvoiceController.php
│   │   │   │   ├── PaymentController.php
│   │   │   │   ├── ShipmentController.php      # Lot B
│   │   │   │   ├── InventoryController.php     # Lot B
│   │   │   │   └── …
│   │   │   ├── Api/
│   │   │   └── Auth/
│   │   ├── Middleware/               # Role, AgencyScope, ActiveUser
│   │   ├── Requests/
│   │   └── Resources/                # API JSON Resources
│   ├── Models/
│   │   ├── Agency.php
│   │   ├── User.php
│   │   ├── Customer.php
│   │   ├── Address.php
│   │   ├── Reservation.php
│   │   ├── Parcel.php
│   │   ├── ParcelPiece.php
│   │   ├── TrackingEvent.php
│   │   ├── PriceRule.php
│   │   ├── Invoice.php
│   │   ├── InvoiceLine.php
│   │   ├── Payment.php
│   │   ├── Shipment.php              # Lot B
│   │   ├── InventoryMovement.php     # Lot B
│   │   ├── AuditLog.php
│   │   └── FeatureFlag.php
│   ├── Policies/
│   ├── Services/
│   │   ├── PricingService.php
│   │   ├── InvoiceService.php
│   │   ├── LabelService.php
│   │   ├── TrackingService.php
│   │   └── DashboardService.php
│   ├── Mail/
│   ├── Notifications/
│   ├── Jobs/
│   ├── Listeners/
│   └── Support/
├── bootstrap/
├── config/
│   ├── malaika.php                   # agences, formats étiquette, feature flags
│   └── …
├── database/
│   ├── factories/
│   ├── migrations/
│   └── seeders/
├── docs/
│   ├── DOCUMENTATION-MALAIKA-FREIGHT.md
│   ├── PLAN-DETAILLE.md
│   ├── PLAN-ETAPES.md
│   ├── STRUCTURE-ARBORESCENCE.md
│   ├── ARCHITECTURE.md
│   ├── diagrams/                     # *.mmd
│   └── _extracted/                   # pack client (CDC + maquette)
├── public/
│   ├── index.php
│   └── storage -> ../storage/app/public
├── resources/
│   ├── css/
│   ├── js/
│   ├── views/
│   │   ├── layouts/
│   │   │   ├── public.blade.php
│   │   │   ├── client.blade.php
│   │   │   └── admin.blade.php
│   │   ├── public/
│   │   ├── client/
│   │   ├── admin/
│   │   ├── pdf/
│   │   │   ├── invoice.blade.php
│   │   │   └── label.blade.php
│   │   ├── emails/
│   │   └── components/
│   │       ├── status-badge.blade.php
│   │       ├── tracking-timeline.blade.php
│   │       └── money.blade.php
│   └── lang/
│       ├── fr/
│       └── es/                       # phase bilingue
├── routes/
│   ├── web.php
│   ├── api.php
│   ├── admin.php                     # optionnel (groupé)
│   ├── client.php
│   └── console.php
├── storage/
├── tests/
│   ├── Feature/
│   │   ├── AgencyIsolationTest.php
│   │   ├── ReservationToParcelTest.php
│   │   ├── InvoicePaymentTest.php
│   │   ├── TrackingPrivacyTest.php
│   │   └── …
│   └── Unit/
│       ├── PricingServiceTest.php
│       └── ParcelStateMachineTest.php
├── .env.example
├── composer.json
├── package.json
├── README.md
└── vite.config.js
```

---

## 3. Organisation des routes (logique)

| Préfixe | Fichier / groupe | Audience |
|---------|------------------|----------|
| `/` | `web.php` public | Visiteurs |
| `/suivi/{token}` | public | Suivi anonyme limité |
| `/app` ou `/client` | client authentifié | Clients |
| `/admin` | admin | Staff |
| `/api/v1` | `api.php` | API token Sanctum |

---

## 4. Modules fonctionnels ↔ dossiers

| Module | Controllers | Models | Services |
|--------|-------------|--------|----------|
| Public / acquisition | `Public/*` | `Reservation` | Pricing (estimatif) |
| Clients | `Admin/Customer*` | `Customer`, `Address` | — |
| Opérations colis | `Admin/Parcel*`, `Reservation*` | `Parcel`, `ParcelPiece` | Tracking, Pricing |
| Facturation | `Invoice*`, `Payment*` | `Invoice*`, `Payment` | Invoice, Label |
| Pilotage | `Dashboard*` | — | Dashboard |
| Expéditions (B) | `Shipment*` | `Shipment` | Shipment |
| Stocks (B) | `Inventory*` | `InventoryMovement` | Inventory |
| Auth / sécurité | `Auth*` | `User`, `AuditLog` | — |

---

## 5. Séparation des préoccupations

1. **Controllers** : HTTP uniquement (thin).
2. **Actions / Services** : règles métier et transactions.
3. **Models** : persistance + relations + petits helpers.
4. **Policies** : autorisations.
5. **Enums / States** : transitions explicites.
6. **Views** : présentation ; calculs métier interdits.

---

## 6. Artefacts non-code

```
docs/
├── DOCUMENTATION-MALAIKA-FREIGHT.md
├── PLAN-DETAILLE.md
├── PLAN-ETAPES.md
├── STRUCTURE-ARBORESCENCE.md
├── ARCHITECTURE.md
├── diagrams/
│   ├── architecture-globale.mmd
│   ├── architecture-applicative.mmd
│   ├── architecture-donnees.mmd
│   ├── architecture-deploiement.mmd
│   ├── flux-metier-principal.mmd
│   └── roles-permissions.mmd
└── _extracted/Malaika_Freight/
    ├── CAHIER_DES_CHARGES.md
    ├── index.html
    └── LISEZ_MOI.txt
```

---

## 7. Branches Git recommandées

| Branche | Usage |
|---------|--------|
| `main` | Stable / déployable |
| `develop` | Intégration (optionnel) |
| `feature/...` | Travaux unitaires |
| `release/lot-a` | Freeze recette Lot A |

---

## 8. État actuel du dépôt (constat)

Au moment de la rédaction :

```
Projet Malaika Freight/
├── .git/
├── .gitignore
└── docs/
    ├── Malaika_Freight_Pack_Developpeur_2026.zip
    ├── _extracted/...
    └── (documentation en cours d’ajout)
```

L’application Laravel sera initialisée dans une phase ultérieure (étape 3 du plan).

---

*Structure & arborescence · Malaïka Freight · v1.0*
