# Plan détaillé de développement — Malaïka Freight

**Projet :** Plateforme fret Madrid ↔ Kinshasa  
**Durée estimée Lot A :** 8 à 12 semaines (rythme régulier)  
**Durée indicative Lot B :** +6 à 10 semaines  
**Stack proposée :** Laravel · PHP 8.3+ · MySQL 8 · Blade · Tailwind · Alpine · Sanctum · DomPDF · Queues

---

## Documents de référence

- [Documentation complète](./DOCUMENTATION-MALAIKA-FREIGHT.md)
- [Plan étape par étape](./PLAN-ETAPES.md)
- [Structure & arborescence](./STRUCTURE-ARBORESCENCE.md)
- [Architecture](./ARCHITECTURE.md)
- [Cahier des charges](./_extracted/Malaika_Freight/CAHIER_DES_CHARGES.md)
- [Diagrammes](./diagrams/)

---

## Principes de pilotage

1. **Livrer par lots** (A → B → C → D) ; pas de big bang.
2. **Sécurité et multi-agences dès le Lot A** (pas un “ajout plus tard”).
3. **Réutiliser les patterns ColisTrack** (rôles, tracking, PDF, audit) sans copier le domaine examen.
4. Chaque phase a : objectif, tâches, livrables, critères de done, commit suggéré.
5. Les modules Documents / Transfert restent derrière **feature flags**.

---

## Vue d’ensemble des phases

| Phase | Contenu | Lot | Priorité |
|-------|---------|-----|----------|
| **P0** | Cadrage, repo, environnement, design tokens | A | Critique |
| **P1** | Schéma BDD Lot A + modèles Eloquent | A | Critique |
| **P2** | Auth, rôles, permissions agence | A | Critique |
| **P3** | Clients, réservations, colis, pièces | A | Critique |
| **P4** | Tarification, factures PDF, étiquettes QR | A | Critique |
| **P5** | Tracking events + suivi public sécurisé | A | Critique |
| **P6** | Site public + estimateur + pré-réservation | A | Critique |
| **P7** | Dashboard admin + espaces rôles | A | Haute |
| **P8** | Emails, notifications, audit | A | Haute |
| **P9** | API REST minimale + tests automatisés | A | Haute |
| **P10** | Déploiement, sauvegardes, recette Lot A | A | Critique |
| **P11** | Expéditions / manifestes | B | Haute |
| **P12** | Stocks, caisses, dépenses, livraisons | B | Haute |
| **P13** | Reporting avancé & préparation OHADA | C | Moyenne |
| **P14** | Documents physiques / corridor inverse | C | Moyenne |
| **P15** | Transfert d’argent (si conformité) | D | Bloqué réglementaire |

---

## Phase 0 — Cadrage & environnement

**Objectif :** Base projet propre, décisions figées pour démarrer.

### Tâches

- [ ] Valider Lot A avec la fondatrice (périmètre écrit)
- [ ] Figurer stack, hébergement cible, domaine provisoire
- [ ] Initialiser Laravel dans le dépôt `malaika-freight`
- [ ] Configurer Laragon (vhost, BDD `malaika_freight`)
- [ ] Installer Breeze (Blade), Sanctum, DomPDF, QR
- [ ] Poser design tokens (navy / gold issus de la maquette)
- [ ] Copier / référencer CDC + maquette dans `docs/`
- [ ] Définir conventions Git (branches, commits conventionnels)

### Livrables

- App Laravel bootable
- `.env.example` complet sans secrets
- README d’installation
- Backlog Lot A priorisé

### Commit suggéré

```
chore: bootstrap Laravel et documentation de démarrage
```

---

## Phase 1 — Base de données Lot A

**Objectif :** Socle relationnel pour réservations, colis, pièces, clients, facturation légère.

### Tables prioritaires

`agencies`, `users` (+ rôle, agency_id), `customers`, `addresses`, `reservations`, `parcels`, `parcel_pieces`, `price_rules`, `invoices`, `invoice_lines`, `payments`, `tracking_events`, `audit_logs`, `feature_flags`

### Tâches

- [ ] Migrations + index (références, statuts, agency_id, tracking_token)
- [ ] Modèles Eloquent + relations
- [ ] Enums PHP (statuts, devises, rôles)
- [ ] Seeders : 2 agences, rôles, utilisateurs démo, règles de prix fictives
- [ ] Factories pour tests

### Critères de done

- `php artisan migrate --seed` OK
- Contraintes FK et unicité (facture, tracking_token, piece reference)

### Commit suggéré

```
feat(db): schéma Lot A réservations colis facturation
```

---

## Phase 2 — Authentification & autorisations

**Objectif :** Accès sécurisé multi-rôles et multi-agences.

### Tâches

- [ ] Breeze : login, reset password, profil
- [ ] Middleware `role`, `agency`, `active`
- [ ] Policies / Gates sur parcels, invoices, customers
- [ ] Redirections post-login selon rôle
- [ ] Journalisation connexions (audit)

### Critères de done

- Un agent Madrid ne peut pas ouvrir un colis Kinshasa hors droit
- Comptes désactivés bloqués

### Commit suggéré

```
feat(auth): rôles, agences et policies Lot A
```

---

## Phase 3 — Clients, réservations, colis, pièces

**Objectif :** Cœur opérationnel réception.

### Tâches

- [ ] CRUD customers (expéditeur / destinataire)
- [ ] Création réservation (admin + formulaire public plus tard)
- [ ] Conversion réservation → parcel
- [ ] Gestion multi-pièces (sequence `1/n`)
- [ ] Photos autorisées (upload sécurisé)
- [ ] Recherche / filtres / pagination
- [ ] Machine à états `canTransitionTo` / `transitionTo`

### Critères de done

- Scénario CDC : pré-réservé → reçu → documents générables
- Une pièce = une référence unique

### Commit suggéré

```
feat(ops): réservations, colis et pièces multi-étiquettes
```

---

## Phase 4 — Tarification, factures, étiquettes

**Objectif :** Documents imprimables conformes CDC.

### Tâches

- [ ] Moteur de prix (itinéraire, service, carton, poids facturable)
- [ ] Frais supplémentaires journalisés
- [ ] Émission facture numérotée + lignes
- [ ] Paiements partiels / solde
- [ ] PDF facture
- [ ] PDF étiquette 100×150 + A4
- [ ] QR jeton + Code 128
- [ ] Versioning réimpression étiquette

### Critères de done

- Impression lisible ; QR pointe vers URL suivi opaque
- EUR/USD jamais additionnés dans les totaux globaux

### Commit suggéré

```
feat(docs): factures PDF et étiquettes QR versionnées
```

---

## Phase 5 — Tracking & suivi public

**Objectif :** Traçabilité interne + transparence externe contrôlée.

### Tâches

- [ ] Enregistrement events (agent, horodatage, localisation, commentaire)
- [ ] Timeline UI admin / client
- [ ] Page publique `/suivi/{token}` (données minimales)
- [ ] Rate limiting + tokens non devinables
- [ ] Sync statut parcel ↔ dernier event

### Critères de done

- Scan public ne révèle pas téléphone / prix / adresse complète

### Commit suggéré

```
feat(tracking): événements et suivi public par jeton
```

---

## Phase 6 — Site public

**Objectif :** Acquisition commerciale.

### Tâches

- [ ] Layout public (branding maquette)
- [ ] Pages : accueil, services, offres, FAQ, contact, légal
- [ ] Estimateur indicatif
- [ ] Formulaire pré-réservation + anti-spam
- [ ] CTA WhatsApp
- [ ] Responsive mobile

### Critères de done

- Pré-réservation crée une `reservation` en base
- Tarif clairement marqué “estimatif”

### Commit suggéré

```
feat(public): site vitrine, estimateur et pré-réservation
```

---

## Phase 7 — Dashboards & espaces rôles

**Objectif :** Pilotage à distance.

### Tâches

- [ ] KPIs : volumes, poids, transit, livrés, incidents
- [ ] Encaissements EUR / USD séparés
- [ ] Filtres agence / dates / corridor
- [ ] Vues dédiées réception / livraison / comptable
- [ ] Alertes (incidents, soldes ouverts)

### Commit suggéré

```
feat(dashboard): pilotage multi-agences Lot A
```

---

## Phase 8 — Emails, notifications, audit

### Tâches

- [ ] Mailables : confirmation pré-résa, facture, statut, livraison
- [ ] Notifications database (cloche admin)
- [ ] Queues
- [ ] `audit_logs` sur actions sensibles (tarif, paiement, statut, PDF)

### Commit suggéré

```
feat(comms): mails, notifications et audit trail
```

---

## Phase 9 — API & tests

### Tâches

- [ ] Endpoints Lot A (reservations, parcels, events, invoices, tracking, payments)
- [ ] Sanctum tokens (préparation mobile / partenaires)
- [ ] Tests Feature prioritaires (auth agence, facture, tracking privacy)
- [ ] Collection Postman

### Commit suggéré

```
feat(api): endpoints Lot A et tests de recette
```

---

## Phase 10 — Déploiement & recette Lot A

### Tâches

- [ ] Procédure déploiement + rollback
- [ ] Scripts sauvegarde / restauration testés
- [ ] Checklist recette CDC §12
- [ ] Remise accès à la société (domaine, admin, secrets)
- [ ] Documentation d’exploitation

### Commit suggéré

```
chore(release): préparation mise en ligne Lot A
```

---

## Phases 11–15 — Lots B / C / D (synthèse)

| Phase | Contenu clé |
|-------|-------------|
| P11 | `shipments`, association colis, AWB, coûts, état douanier |
| P12 | Stocks emballages, mouvements, caisses, dépenses, POD livraison |
| P13 | Journal / états financiers progressifs, exports comptable |
| P14 | Documents physiques, corridor inverse activé |
| P15 | Transfert : partenaire, KYC/AML, wallets séparés — go/no-go juridique |

---

## Estimation indicative (Lot A)

| Phase | Effort relatif |
|-------|----------------|
| P0–P2 | 1.5 à 2 semaines |
| P3–P5 | 3 à 4 semaines |
| P6–P8 | 2 à 3 semaines |
| P9–P10 | 1.5 à 2 semaines |
| **Total** | **≈ 8–12 semaines** |

Les estimations dépendent de la disponibilité des décisions ouvertes (prix, identité fiscale, paiements, design final).

---

## Risques & mitigations

| Risque | Impact | Mitigation |
|--------|--------|------------|
| Scope creep (OHADA, transfert…) | Délai | Lots stricts + feature flags |
| Impression thermique non conforme | Opération | Prototyper étiquette dès P4 |
| Fuite de données via suivi public | Confiance / RGPD | Jeton opaque + reviews sécurité |
| Confusion EUR/USD | Finance | Totaux séparés + tests |
| Décisions fondatrice en attente | Blocage | Valeurs de config datées + placeholders |

---

## Définition de “Lot A terminé”

1. Parcours prospect → pré-résa → réception → facture + étiquette → suivi → livraison simulée OK
2. Isolation agences vérifiée
3. Recette CDC §12 cochée
4. Code dans un dépôt contrôlé par la société
5. Procédure de sauvegarde testée

---

*Plan détaillé Malaïka Freight · v1.0 · Novembre 2026*
