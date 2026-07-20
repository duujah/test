# 01 — Analyse simplifiée du cahier des charges

> Synthèse du document *CarGova — Cahier des Charges v1.0*. Objectif : rendre le besoin exploitable
> pour la conception et le développement.

## 1. Vision

CarGova met en relation des **industries** qui doivent livrer des marchandises et des **transporteurs**
(chauffeurs indépendants ou sociétés de transport). Sa particularité : **garantir un fret retour** afin
d'éviter les trajets à vide.

## 2. Problème → Solution

| Problème | Conséquence | Solution CarGova |
|----------|-------------|------------------|
| Le camion attend / repart à vide après une livraison | Surcoût financier | Garantie de fret retour |
| Trajet retour sans chargement | Impact environnemental (carburant, émissions) | Optimisation du remplissage des trajets |
| Pas de visibilité sur la demande retour | Incertitude pour le chauffeur | Lignes prédéfinies où la demande est établie dans les 2 sens |

## 3. Périmètre du MVP

Le lancement se limite à des **lignes fixes** pour garantir la promesse de fret retour :

- **Casablanca ↔ Tanger**
- **Casablanca ↔ Fès**

## 4. Acteurs

| Acteur | Accès | Rôle principal |
|--------|-------|----------------|
| **Chauffeur indépendant (solo)** | Mobile | Recherche, négocie, accepte et exécute des missions seul |
| **Société de transport** | Web + Mobile | Recherche, négocie, puis assigne les missions à ses chauffeurs |
| **Chauffeur rattaché** | Mobile | Exécute uniquement les missions assignées par sa société |
| **Industrie / Expéditeur** | Web + Mobile | Publie des offres, suit ses livraisons, peut proposer ses propres camions pour du fret retour |
| **Administrateur (CarGova)** | Back-office web | Gestion globale (utilisateurs, offres, lignes, transactions, litiges) |

## 5. Fonctionnalités clés

### 5.1 Création d'une offre (industrie)
L'offre précise : type de camion requis, poids & volume, responsable du chargement/déchargement
(→ estimation du temps de référence), détails du produit, conditions de prise en charge et de livraison.

### 5.2 Garantie de fret retour — « déblocage prioritaire »
1. Un chauffeur accepte un trajet **aller** (ex. Casablanca → Tanger).
2. La plateforme lui **débloque automatiquement** l'accès à la ligne **retour** (Tanger → Casablanca).
3. Sur cette ligne retour, il est **prioritaire** sur toutes les offres disponibles.
4. Il peut donc obtenir un retour **avant même de démarrer** l'aller.

### 5.3 Tarification — enchère encadrée
- Prix de base **calculé automatiquement** par le système.
- **Industrie** : peut baisser le prix, dans la limite d'un **seuil plancher**.
- **Chauffeur** : peut faire une contre-offre à la hausse, dans la limite d'un **plafond**.
- *À définir* : formule du prix de base + valeurs des seuils.

### 5.4 KYC — vérification & confiance
Aucun compte activé sans validation des documents :
- **Chauffeur / Société** : permis, assurance, documents d'état du camion.
- **Industrie** : registre de commerce.

### 5.5 Suivi de mission — GPS + confirmation double
Étapes horodatées, validées par confirmation croisée chauffeur ↔ industrie :
`Démarrage → Arrivée prise en charge (GPS) → Chargement confirmé (industrie) → En route →
Arrivée destination (GPS + notif) → Dépôt déclaré (chauffeur) → Dépôt confirmé (industrie)`.
L'horodatage permet d'imputer un éventuel retard au chauffeur (route) ou à l'industrie (chargement/déchargement).

### 5.6 Temps d'attente & pénalités
Le temps de chargement/déchargement de référence sert de base : si l'industrie le dépasse, des
**frais d'attente** lui sont facturés pour compenser le temps perdu par le chauffeur.

### 5.7 Paiement — escrow via Cash Plus
1. Accord sur le prix → l'industrie **paie**.
2. CarGova **séquestre** (escrow) le montant jusqu'à la livraison.
3. Livraison confirmée → CarGova **reverse** le montant (net) au transporteur et prélève sa **commission**.

L'industrie voit le détail des frais (Cash Plus, commission) ; le transporteur ne voit que son **montant net**.

### 5.8 Litiges & réclamations
- Une **réclamation** peut être ouverte par l'industrie avant déblocage du paiement.
- Tant qu'une réclamation est ouverte, le paiement reste **suspendu**.
- Le montant n'est débloqué qu'après confirmation de bonne réception ou résolution du litige.

### 5.9 Évaluations & favoris
- **Notation à double sens** : chauffeur ↔ industrie.
- L'industrie peut constituer une liste de **chauffeurs favoris / de confiance**.

### 5.10 Notifications
Notifications **push** pour les événements critiques (opportunité de retour prioritaire, arrivée du camion,
retard de chargement, etc.).

### 5.11 Back-office administrateur
Super Admin Dashboard : gestion globale des utilisateurs, offres, lignes, transactions et litiges.

## 6. Langues supportées

Arabe · Français · Anglais.

## 7. Architecture technique

| Composant | Technologie |
|-----------|-------------|
| Applications mobiles (chauffeur, société, industrie) | **Flutter** (Android & iOS) |
| Applications web (admin, société, industrie) | **React.js** |
| Backend | **FastAPI** (Python) |
| Base de données | **PostgreSQL** (transactionnel) + **Redis** (suivi temps réel) |
| Cartographie / géolocalisation | **Google Maps API** ou **Mapbox** |

## 8. Règles métier importantes (extraites)

- RG1 — Un compte ne peut être **actif** qu'après validation KYC.
- RG2 — Accepter un trajet aller **débloque** et **priorise** la ligne retour associée.
- RG3 — La négociation est **encadrée** : plancher côté industrie, plafond côté chauffeur.
- RG4 — Le paiement est **séquestré** et n'est libéré qu'après confirmation de livraison / résolution du litige.
- RG5 — Une **réclamation ouverte suspend** le paiement.
- RG6 — Le **dépassement du temps de référence** côté industrie génère des frais d'attente.
- RG7 — Chaque étape de mission est **horodatée** et **confirmée des deux côtés**.
- RG8 — Un chauffeur rattaché n'accède **qu'aux missions assignées** par sa société.

## 9. Points en suspens (à clarifier)

1. Formule du prix de base + seuils de négociation.
2. Taux de commission CarGova.
3. Permissions & écrans détaillés du back-office admin.
4. Processus détaillé de résolution des litiges (médiation, remboursement partiel, arbitrage).
