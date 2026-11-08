# MALAÏKA FREIGHT — Dossier de transmission au développeur

**Référence fonctionnelle 1.0 — 2 novembre 2026**

## 0. Statut du livrable

Le fichier `index.html` est une **maquette interactive de référence**, utilisable hors serveur dans un navigateur, avec données fictives en stockage local. Ce n'est pas un système de production. Les interfaces, les règles et les parcours sont à reprendre dans une application sécurisée avec base de données. Le QR de démonstration dépend d'un service externe et contient un domaine fictif : **à remplacer impérativement**.

## 1. Objectif et périmètre

Plateforme web responsive et installable (PWA, si retenue), un seul produit avec trois surfaces : site public, espace client, administration privée. Société : Malaika Saida Holding SARLU ; marque commerciale : Malaïka Freight. Départ pilote Madrid → Kinshasa ; architecture prête pour Kinshasa → Madrid. Modules futurs : envoi de documents physiques et transfert d'argent sur les deux corridors, **désactivés** jusqu'aux conditions techniques, opérationnelles et réglementaires nécessaires. Les autres branches de la holding ne sont pas dans le périmètre de livraison initial.

## 2. Parcours prioritaire — du prospect à la remise

1. Prospect consulte site, formule (au kilo / carton 5, 10, 20 kg), conditions et pré-réserve.
2. Système crée réservation avec identifiant, poids estimé, origine, destination, expéditeur, destinataire, consentements et statut « Pré-réservé ».
3. Agent autorisé réceptionne, vérifie la nature admissible du contenu, confirme poids réel, nombre de pièces, dimensions si nécessaires, conditionnement et état. Génération d'une référence **unique par pièce** si plusieurs cartons.
4. Tarif est calculé selon itinéraire, service, période, catégorie et poids facturable ; agent valide les frais supplémentaires et devise. Toute modification de poids/tarif doit être journalisée.
5. Facture PDF et étiquette PDF imprimable avec QR + code-barres sont générées depuis la même fiche, mais restent deux documents séparés. Le client reçoit la facture selon son canal choisi.
6. Agent associe chaque pièce à un manifeste d'expédition et enregistre les scans d'étape : réception, préparation, départ, arrivée, dédouanement, mise à disposition, remise.
7. Le client consulte une page de suivi sécurisée (lien à jeton ou référence + vérification). Aucun nom, téléphone, prix ou détail privé n'est révélé par un scan public non authentifié.
8. À la remise, preuve de livraison (agent, date, mode, signature ou code de remise si retenu). Incident et réclamation tracés.

## 3. Écrans du site public

Accueil (proposition de valeur et deux CTA « Pré-réserver » / « Suivre un colis »), services, itinéraires, offres/cartons, estimateur indicatif, pré-réservation, suivi, FAQ, contact/WhatsApp, conditions et confidentialité. Le tarif doit être affiché comme estimatif tant que le poids réel et les frais ne sont pas confirmés. Contenu bilingue FR/ES envisageable plus tard.

## 4. Modules de l'administration

- **Tableau de bord** : volumes par agence, poids reçus, réservés, en transit, livrés, incidents, encaissements séparés EUR/USD, dépenses, marge par manifeste, alertes ; filtres agence, date, itinéraire.
- **Clients** : contacts, consentements, historique, expéditeur/destinataire, documents accessibles par rôle.
- **Réservations et colis** : identifiants, poids estimé/réel/facturable, photos autorisées, contenu déclaré, pièces, scans, anomalies.
- **Expéditions** : calendrier, manifeste, prestataires, AWB/numéros transport, poids consolidé, coût réel, état douanier, documents associés.
- **Catalogue/stock** : prix au kilo et cartons 5/10/20 kg ; stocks cartons, rubans, films, étiquettes ; mouvements et seuils.
- **Facturation** : devis, facture numérotée, PDF, reçus, acomptes, paiements partiels, avoirs, solde, frais supplémentaires, taxes/mentions légales configurables après validation.
- **Finances** : caisses par agence et devise, encaissements, dépenses, rapprochement, marges ; puis journal/grand livre/balance/états OHADA avec validation comptable. Ne jamais additionner EUR et USD sans taux daté et source enregistrée.
- **Personnel** : rôles, agence, pointage QR avec contrôles antifraude adaptés, audit des actions ; un scan QR seul ne prouve pas la présence physique.
- **Livraisons** : planning, affectation, preuve de remise, échecs et incidents.
- **Avis** : sollicitation post-livraison, score, modération et traitement des plaintes.
- **Paramètres** : agences, itinéraires, devises, services, prix datés, numérotation, permissions, modèles PDF, sauvegardes.
- **Documents (futur)** : envoi physique sécurisé, preuve de remise, conservation limitée.
- **Transfert (futur)** : corridor Madrid ↔ Kinshasa, simulation seulement tant que non autorisé. Opérations réelles uniquement après cadre légal, conformité KYC/AML et partenariat/agréments appropriés en Espagne et RDC. Séparer les fonds des clients des recettes de fret.

## 5. Champs minimum — fiche colis

`id`, `reservation_id`, `sender_id`, `recipient_id`, `origin_agency_id`, `destination_agency_id`, `service_id`, `pieces[]`, `declared_contents`, `estimated_weight_kg`, `actual_weight_kg`, `billable_weight_kg`, `carton_type`, `price_rule_id`, `unit_price`, `currency`, `extra_fees[]`, `taxes[]`, `total`, `amount_paid`, `balance_due`, `status`, `shipment_id`, `received_at`, `created_by`, `updated_at`, `tracking_token`, `events[]`.

Pour chaque pièce : `piece_id`, `parcel_id`, `sequence_number`, `total_pieces`, `weight_kg`, `dimensions`, `label_version`, `label_printed_at`. La référence publique ne doit pas être un secret d'accès aux données personnelles.

## 6. Facture — contenu et règles

Identité juridique, contacts professionnels, numéro/date, référence(s) colis, expéditeur, destinataire, itinéraire, prestation, poids facturable, prix/kg, emballage, frais, taxes selon applicabilité, devise, total, montant encaissé, reste à payer, moyen de paiement, mentions légales et conditions applicables. L'adresse et les identifiants fiscaux seront ajoutés après confirmation. Numérotation unique, immuable après émission ; corrections par avoir ou facture rectificative selon règles applicables.

## 7. Étiquette — format et scan

PDF thermique 100×150 mm (4×6 pouces) et option A4 ; logo, référence pièce, destination, nom destinataire et contact strictement nécessaire, poids, numéro de pièce `1/3`, QR, code-barres Code 128, consignes de manutention. Le QR contient une URL de suivi avec jeton opaque, ou un identifiant non sensible, **jamais les données client en clair**. Les scans de travail nécessitent authentification agent et autorisation. Réimpression avec version et historique ; éviter doublons actifs non contrôlés.

## 8. Rôles et sécurité

Superadministratrice ; responsable Madrid ; responsable Kinshasa ; agent réception ; agent livraison ; comptable ; client. Permissions fines par agence et action, authentification forte des comptes sensibles, journal d'audit côté serveur, TLS, chiffrement des sauvegardes, rétention et suppression définies, limitation des accès, contrôle d'intégrité des fichiers, séparation démonstration/production. Les étudiants peuvent développer sur données fictives, sans accès aux vrais dossiers clients ou secrets de production.

## 9. Proposition de modèle relationnel

`organizations`, `agencies`, `users`, `roles`, `permissions`, `customers`, `addresses`, `reservations`, `parcels`, `parcel_pieces`, `tracking_events`, `shipments`, `shipment_parcels`, `price_rules`, `packaging_skus`, `inventory_movements`, `invoices`, `invoice_lines`, `payments`, `expenses`, `cash_accounts`, `exchange_rates`, `deliveries`, `proofs_of_delivery`, `documents`, `audit_logs`, `feature_flags`. Ajouter clés étrangères, contraintes d'unicité, horodatage et index sur références/états/agences. Transactions atomiques pour paiements et changements de stock.

## 10. API indicative

`POST /reservations`, `POST /parcels`, `GET /parcels/:id`, `POST /parcels/:id/events`, `POST /shipments`, `POST /shipments/:id/parcels`, `POST /invoices`, `GET /invoices/:id/pdf`, `GET /parcels/:id/label`, `POST /payments`, `GET /dashboard`, `GET /tracking/:token`, `POST /deliveries/:id/confirm`. Chaque route privée : autorisation par rôle et agence ; validation des données, journalisation, pagination et protection contre abus.

## 11. Livraisons par lots

**Lot A — lancement commercial** : site public, pré-réservation, fiches clients/colis, facture, étiquette QR, suivi public sécurisé, tableau de bord de base, connexion et rôles.
**Lot B — opérations avancées** : manifestes, scans, stocks, dépenses, encaissements, livraisons, audit et rapports.
**Lot C — extensions** : Kinshasa → Madrid, Documents, automatisations et reporting comptable approfondi.
**Lot D — transfert d'argent** : uniquement après faisabilité réglementaire et partenaire/agréments.

## 12. Critères de recette obligatoires

- Créer une réservation, la convertir en colis, générer une facture correcte et une étiquette imprimable 100×150 mm.
- Plusieurs pièces ont chacune un identifiant et une étiquette ; les scans se rattachent au bon dossier.
- Un agent Madrid ne peut ni voir ni modifier des données Kinshasa hors de ses permissions.
- Les montants EUR et USD restent séparés ; les paiements partiels actualisent le solde et l'historique.
- Un lien public de suivi ne révèle aucune donnée personnelle non autorisée.
- Un statut modifié est visible dans le tableau de bord et le suivi, avec audit agent/heure.
- Sauvegarde restaurée sur environnement de test ; aucune donnée perdue dans un scénario de reprise.
- Application testée sur Android, ordinateur et navigateurs récents ; documents imprimés lisibles.

## 13. Décisions encore ouvertes (à valider par la fondatrice)

Logo final ; domaine ; identité graphique définitive ; prix/kg et prix carton ; règles de poids volumétrique ; frais de dédouanement et responsabilités ; moyens de paiement ; adresses/agences ; délais de départ ; conditions générales ; consentements ; budget hébergement ; politique de conservation ; choix technologique du développeur.

## 14. Remise technique attendue de l'équipe

Code source dans un dépôt appartenant à Malaika Saida Holding SARLU, documentation d'installation, schéma DB et migrations, fichiers de configuration d'exemple sans secrets, scripts de sauvegarde, tests automatisés prioritaires, procédure de déploiement et réversibilité, liste des dépendances et coûts récurrents. Domaine, serveur et comptes d'administration détenus par la société ; accès de l'équipe révocables.
