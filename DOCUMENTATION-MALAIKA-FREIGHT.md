# Documentation complète — Malaïka Freight

**Projet :** Plateforme numérique de fret Madrid ↔ Kinshasa  
**Société :** Malaika Saida Holding SARLU  
**Marque commerciale :** Malaïka Freight  
**Référence :** Documentation produit & technique 1.0 — novembre 2026  
**Dépôt :** [github.com/Xenoblade-lab/malaika-freight](https://github.com/Xenoblade-lab/malaika-freight)

---

## Documents liés

| Document | Rôle |
|----------|------|
| [Cahier des charges (source client)](./_extracted/Malaika_Freight/CAHIER_DES_CHARGES.md) | Exigences fonctionnelles transmises |
| [Plan détaillé](./PLAN-DETAILLE.md) | Phases, livrables, estimations |
| [Plan étape par étape](./PLAN-ETAPES.md) | Exécution séquentielle |
| [Structure & arborescence](./STRUCTURE-ARBORESCENCE.md) | Organisation du dépôt |
| [Architecture complète](./ARCHITECTURE.md) | Conception technique |
| [Diagrammes Mermaid](./diagrams/) | Représentations `.mmd` |
| Maquette interactive | `docs/_extracted/Malaika_Freight/index.html` |

---

## Table des matières

1. [Présentation](#1-présentation)
2. [Problématique et objectifs](#2-problématique-et-objectifs)
3. [Périmètre et lots](#3-périmètre-et-lots)
4. [Acteurs et rôles](#4-acteurs-et-rôles)
5. [Surfaces applicatives](#5-surfaces-applicatives)
6. [Fonctionnalités](#6-fonctionnalités)
7. [Flux métier principaux](#7-flux-métier-principaux)
8. [Règles métier critiques](#8-règles-métier-critiques)
9. [Modèle de données (vue métier)](#9-modèle-de-données-vue-métier)
10. [Statuts et transitions](#10-statuts-et-transitions)
11. [Facturation, étiquettes et QR](#11-facturation-étiquettes-et-qr)
12. [Sécurité et confidentialité](#12-sécurité-et-confidentialité)
13. [Stack technique retenue](#13-stack-technique-retenue)
14. [API indicative](#14-api-indicative)
15. [Recette et critères d’acceptation](#15-recette-et-critères-dacceptation)
16. [Décisions ouvertes](#16-décisions-ouvertes)
17. [Hors périmètre initial](#17-hors-périmètre-initial)
18. [Glossaire](#18-glossaire)

---

## 1. Présentation

**Malaïka Freight** est un service de fret entre **Madrid** et **Kinshasa**. La plateforme numérique vise à digitaliser le cycle complet : acquisition du client, pré-réservation, réception, facturation, étiquetage QR, consolidation en expédition, suivi sécurisé, livraison et pilotage à distance.

Le produit est **un seul système** avec trois surfaces :

1. **Site public** — acquisition, estimateur, pré-réservation, suivi, contact
2. **Espace client** — historique, documents, soldes, suivi authentifié
3. **Administration privée** — opérations multi-agences (Madrid / Kinshasa)

> La maquette HTML fournie est une **référence UX** à données fictives (localStorage). Elle n’est pas un système de production.

---

## 2. Problématique et objectifs

### Problématique

Sans plateforme unifiée, le fret Madrid–Kinshasa repose sur des échanges manuels (WhatsApp, Excel, papier). Conséquences typiques : doublons de références, erreurs de poids/tarif, suivi opaque pour le client, difficulté de piloter les caisses EUR/USD, et absence d’audit fiable entre agences.

### Objectifs

| Objectif | Description |
|----------|-------------|
| **Traçabilité** | Chaque pièce a une référence unique, des scans horodatés et un historique auditables |
| **Transparence client** | Suivi sécurisé sans exposer de données personnelles en clair |
| **Opérations multi-agences** | Permissions par agence ; Madrid et Kinshasa collaborent sans fuite de données |
| **Fiabilité financière** | Factures, acomptes, soldes ; EUR et USD jamais mélangés sans taux daté |
| **Pilotage** | Tableau de bord volumes, transit, incidents, encaissements |
| **Évolutivité** | Architecture prête pour corridor inverse, documents physiques, transferts (sous conformité) |

---

## 3. Périmètre et lots

### Lot A — Lancement commercial (priorité)

- Site public (accueil, services, estimateur, pré-réservation, suivi, FAQ, contact)
- Clients, réservations, colis, pièces
- Facture PDF + étiquette PDF (QR + code-barres)
- Suivi public sécurisé (jeton opaque)
- Auth + rôles de base
- Dashboard opérationnel minimal

### Lot B — Opérations avancées

- Manifestes / expéditions
- Scans d’étape renforcés
- Stocks (cartons, emballages)
- Dépenses, caisses, rapprochements simples
- Livraisons + preuve de remise
- Audit et rapports

### Lot C — Extensions

- Kinshasa → Madrid en production
- Module documents physiques
- Automatisations et reporting comptable approfondi (vers OHADA)

### Lot D — Transfert d’argent

- **Uniquement** après cadre légal, KYC/AML, agréments ES/RDC et partenaire agréé
- Simulation seule tant que non autorisé
- Séparation stricte des fonds clients et des recettes fret

---

## 4. Acteurs et rôles

| Rôle | Responsabilités |
|------|-----------------|
| **Superadministratrice** | Configuration globale, permissions, paramètres, accès complet |
| **Responsable Madrid** | Pilotage agence Madrid, validation opérationnelle locale |
| **Responsable Kinshasa** | Pilotage agence Kinshasa, validation opérationnelle locale |
| **Agent réception** | Réception, pesée, création pièces, documents, association manifeste |
| **Agent livraison** | Planning, remise, preuve de livraison, incidents |
| **Comptable** | Factures, paiements, caisses, dépenses, états financiers |
| **Client** | Pré-réservation, suivi, consultation documents / soldes |

### Matrice de permissions (Lot A — indicative)

| Action | Superadmin | Resp. agence | Réception | Livraison | Comptable | Client |
|--------|:----------:|:------------:|:---------:|:---------:|:---------:|:------:|
| Gérer paramètres | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Voir dashboard global | ✅ | ⚠️ agence | ⚠️ limité | ⚠️ limité | ⚠️ finance | ❌ |
| Créer / convertir colis | ✅ | ✅ | ✅ | ❌ | ❌ | Pré-résa |
| Générer facture / étiquette | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Mettre à jour scan / statut | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ |
| Confirmer livraison | ✅ | ✅ | ❌ | ✅ | ❌ | ❌ |
| Enregistrer paiement | ✅ | ✅ | ⚠️ | ❌ | ✅ | ❌ |
| Suivi public (jeton) | — | — | — | — | — | ✅ |

---

## 5. Surfaces applicatives

### 5.1 Site public

- Accueil (proposition de valeur + CTA Pré-réserver / Suivre)
- Services, itinéraires, offres / cartons (5, 10, 20 kg + au kilo)
- Estimateur **indicatif**
- Formulaire de pré-réservation
- Page de suivi sécurisée
- FAQ, contact / WhatsApp
- Conditions générales & confidentialité
- Contenu bilingue FR/ES (phase ultérieure envisageable)

### 5.2 Espace client

- Authentification
- Liste des réservations / colis
- Documents (facture, suivi)
- Solde et historique de paiements
- Profil et consentements

### 5.3 Administration

Modules cibles (maquette + CDC) :

- Tableau de bord
- Clients
- Réservations & colis
- Expéditions
- Catalogue / stock
- Facturation
- Finances
- Personnel
- Livraisons
- Avis
- Paramètres
- Documents (futur) / Transfert (futur)

---

## 6. Fonctionnalités

### Lot A

| Domaine | Fonctionnalités |
|---------|-----------------|
| Public | Pages marketing, estimateur, pré-réservation, suivi jeton |
| Auth | Login, rôles, sessions sécurisées, audit connexions |
| Clients | Fiches contacts, expéditeur / destinataire, consentements |
| Colis | Conversion réservation → colis, poids estimé/réel/facturable, pièces |
| Documents | Facture PDF, étiquette 100×150 mm + option A4, QR opaque |
| Suivi | Événements horodatés, page publique limitée |
| Dashboard | Volumes, transit, livrés, incidents, encaissements EUR/USD séparés |

### Lot B+

Manifestes, stocks, caisses/dépenses, preuves de livraison, rapports, OHADA progressif.

---

## 7. Flux métier principaux

### Flux 1 — Du prospect à la pré-réservation

```
Prospect consulte site
  → estime un tarif (indicatif)
  → pré-réserve (expéditeur, destinataire, poids estimé, consentements)
  → système crée réservation (statut Pré-réservé) + identifiant
  → confirmation (email / WhatsApp selon canal choisi)
```

### Flux 2 — Réception et documents

```
Agent réceptionne
  → vérifie admissibilité du contenu
  → confirme poids réel, pièces, dimensions si besoin
  → génère une référence unique par pièce
  → calcule tarif (itinéraire, service, période, catégorie, poids facturable)
  → journalise toute modification poids/tarif
  → émet facture PDF + étiquette(s) PDF
  → client reçoit la facture via son canal
```

### Flux 3 — Expédition et suivi

```
Agent associe pièce(s) au manifeste
  → scans : réception → préparation → départ → arrivée → dédouanement → mise à disposition → remise
  → client suit via lien à jeton (données minimales)
  → remise : preuve (agent, date, mode, signature ou code)
  → incident / réclamation tracés
```

---

## 8. Règles métier critiques

1. **Tarif estimatif** tant que poids réel et frais ne sont pas confirmés.
2. **Une référence unique par pièce** si plusieurs cartons.
3. Facture et étiquette sont **deux documents séparés**, issus de la même fiche.
4. Le QR / suivi public **ne contient jamais** nom, téléphone, prix ou détails privés en clair.
5. Les scans opérationnels exigent **authentification agent** + autorisation.
6. **EUR et USD ne s’additionnent jamais** sans taux daté et source enregistrée.
7. Numérotation de facture **unique et immuable** après émission ; correction par avoir / facture rectificative.
8. Un agent Madrid **ne voit ni ne modifie** les données Kinshasa hors permissions.
9. Réimpression d’étiquette versionnée ; éviter les doublons actifs non contrôlés.
10. Modules Documents et Transfert **désactivés** jusqu’aux conditions opérationnelles / réglementaires.

---

## 9. Modèle de données (vue métier)

### Entités principales

`organizations`, `agencies`, `users`, `roles`, `permissions`, `customers`, `addresses`, `reservations`, `parcels`, `parcel_pieces`, `tracking_events`, `shipments`, `shipment_parcels`, `price_rules`, `packaging_skus`, `inventory_movements`, `invoices`, `invoice_lines`, `payments`, `expenses`, `cash_accounts`, `exchange_rates`, `deliveries`, `proofs_of_delivery`, `documents`, `audit_logs`, `feature_flags`.

### Fiche colis — champs minimum

`id`, `reservation_id`, `sender_id`, `recipient_id`, `origin_agency_id`, `destination_agency_id`, `service_id`, `pieces[]`, `declared_contents`, `estimated_weight_kg`, `actual_weight_kg`, `billable_weight_kg`, `carton_type`, `price_rule_id`, `unit_price`, `currency`, `extra_fees[]`, `taxes[]`, `total`, `amount_paid`, `balance_due`, `status`, `shipment_id`, `received_at`, `created_by`, `updated_at`, `tracking_token`, `events[]`.

### Pièce

`piece_id`, `parcel_id`, `sequence_number`, `total_pieces`, `weight_kg`, `dimensions`, `label_version`, `label_printed_at`.

> La référence publique **n’est pas** un secret d’accès aux données personnelles : le suivi public repose sur un **jeton opaque**.

---

## 10. Statuts et transitions

Statuts de référence (alignés maquette / CDC) :

| Statut | Signification |
|--------|---------------|
| Pré-réservé | Demande client non encore réceptionnée |
| Reçu à Madrid / Reçu à Kinshasa | Réception physique confirmée |
| En préparation | Conditionnement / consolidation |
| En transit | Départ effectué |
| Arrivé à destination | Arrivée agence destination |
| En dédouanement | Traitement douanier |
| Disponible pour retrait | Prêt pour client / livraison |
| Livré | Remise confirmée |
| Incident | Anomalie à traiter |
| Annulé | Clôture sans livraison |

Les transitions sont **contrôlées côté serveur** (machine à états) et chaque changement produit un `tracking_event` + entrée d’audit.

---

## 11. Facturation, étiquettes et QR

### Facture

Identité juridique, contacts, numéro/date, références colis, expéditeur, destinataire, itinéraire, prestation, poids facturable, prix/kg, emballage, frais, taxes, devise, total, encaissé, reste à payer, moyen de paiement, mentions légales.

### Étiquette

- Format thermique **100×150 mm** (4×6) + option A4
- Logo, référence pièce, destination, destinataire (données strictement nécessaires)
- Poids, numéro de pièce `1/3`, QR, Code 128, consignes
- QR = URL de suivi avec jeton opaque **ou** identifiant non sensible

---

## 12. Sécurité et confidentialité

- Authentification forte pour comptes sensibles
- Autorisation fine par **rôle + agence + action**
- TLS, sauvegardes chiffrées, rétention / suppression définies
- Journal d’audit serveur
- Séparation démonstration / production
- Données fictives uniquement en développement étudiant / maquette
- Domaine, serveur et comptes d’administration **détenus par la société**

---

## 13. Stack technique retenue

Choix proposé (inspiré de l’expérience ColisTrack, adapté au produit commercial) :

| Couche | Technologie |
|--------|-------------|
| Backend | PHP 8.3+ · Laravel 11/12+ |
| Frontend admin / client | Blade · Tailwind CSS · Alpine.js · Vite |
| Site public | Blade (composants) — PWA optionnelle plus tard |
| BDD | MySQL 8 |
| Auth | Laravel Breeze (web) · Sanctum (API mobile future) |
| PDF | DomPDF ou Snappy (étiquettes thermiques à valider) |
| QR / barcodes | Package dédié (ex. simplesoftwareio/simple-qrcode + code128) |
| Queues / mails | Database/Redis queue · SMTP / provider transactionnel |
| Hébergement | VPS / Laragon (dev) · déploiement documenté |

> Le choix final reste une **décision développeur** validée avec la fondatrice (CDC §13).

---

## 14. API indicative

Routes privées : autorisation rôle + agence, validation, journalisation, pagination, anti-abus.

| Méthode | Endpoint | Usage |
|---------|----------|-------|
| POST | `/reservations` | Pré-réservation |
| POST | `/parcels` | Création / conversion colis |
| GET | `/parcels/:id` | Détail |
| POST | `/parcels/:id/events` | Scan / événement |
| POST | `/shipments` | Créer manifeste |
| POST | `/shipments/:id/parcels` | Associer colis |
| POST | `/invoices` | Émettre facture |
| GET | `/invoices/:id/pdf` | PDF facture |
| GET | `/parcels/:id/label` | Étiquette |
| POST | `/payments` | Encaissement |
| GET | `/dashboard` | KPIs |
| GET | `/tracking/:token` | Suivi public |
| POST | `/deliveries/:id/confirm` | Preuve de remise |

---

## 15. Recette et critères d’acceptation

- [ ] Créer une réservation → convertir en colis → facture correcte + étiquette 100×150 mm
- [ ] Plusieurs pièces : chacune a un ID et une étiquette ; scans rattachés au bon dossier
- [ ] Isolation Madrid / Kinshasa hors permissions
- [ ] EUR et USD séparés ; paiements partiels mettent à jour solde + historique
- [ ] Lien public de suivi sans données personnelles non autorisées
- [ ] Statut modifié visible dashboard + suivi + audit agent/heure
- [ ] Sauvegarde restaurée sur environnement de test
- [ ] Test Android, desktop, navigateurs récents ; impressions lisibles

---

## 16. Décisions ouvertes

À valider par la fondatrice avant production :

Logo final · domaine · identité graphique · prix/kg et cartons · poids volumétrique · frais de dédouanement · moyens de paiement · adresses/agences · délais de départ · CGU · consentements · budget hébergement · conservation des données · stack finale.

---

## 17. Hors périmètre initial

- Autres branches de la holding
- Transfert d’argent réel
- Envoi de documents physiques (Lot C)
- Comptabilité OHADA complète (approche progressive Lot B/C)
- Application mobile native (API d’abord)

---

## 18. Glossaire

| Terme | Définition |
|-------|------------|
| **Réservation** | Demande client avant réception physique |
| **Colis / Parcel** | Dossier métier regroupant une ou plusieurs pièces |
| **Pièce** | Unité physique étiquetée (carton) |
| **Manifeste** | Regroupement de colis pour un départ |
| **Poids facturable** | Poids retenu pour le tarif (réel et/ou volumétrique) |
| **Jeton de suivi** | Secret opaque donnant accès limité au suivi public |
| **Agence** | Site opérationnel (Madrid, Kinshasa) |

---

*Document de travail interne — Malaïka Freight · Transmission développeur · Novembre 2026*
