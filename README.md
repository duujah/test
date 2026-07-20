# CarGova — Documentation d'analyse & de conception

CarGova est une plateforme de mise en relation (modèle *inDrive*) entre **industries/expéditeurs** et
**transporteurs routiers**, avec une **garantie de fret retour**. Au-delà d'une simple marketplace,
c'est un écosystème combinant **marketplace + TMS** (Transport Management System) couvrant la
négociation, le KYC, le suivi temps réel, le paiement sous séquestre (escrow) et la gestion des litiges.

Ce dépôt contient l'analyse simplifiée du cahier des charges ainsi que les diagrammes de conception
(cas d'utilisation, classes, base de données) rédigés en **Mermaid** (rendus automatiquement par GitHub).

## Contenu

| Document | Description |
|----------|-------------|
| [docs/01-analyse.md](docs/01-analyse.md) | Analyse simplifiée du cahier des charges (acteurs, fonctionnalités, règles métier, MVP, stack) |
| [docs/02-diagramme-cas-utilisation.md](docs/02-diagramme-cas-utilisation.md) | Diagramme de cas d'utilisation (use case) |
| [docs/03-diagramme-classes.md](docs/03-diagramme-classes.md) | Diagramme de classes (modèle du domaine) |
| [docs/04-diagramme-base-de-donnees.md](docs/04-diagramme-base-de-donnees.md) | Diagramme entité-association (base de données PostgreSQL) |

## Résumé express

- **Problème** : les camions repartent souvent **à vide** faute de fret retour → surcoût financier + impact écologique.
- **Solution** : garantir un **fret retour** au chauffeur avant même le trajet aller, via un **déblocage prioritaire** de la ligne retour.
- **MVP** : lignes fixes uniquement — **Casablanca ↔ Tanger** et **Casablanca ↔ Fès**.
- **Stack** : Flutter (mobile), React.js (web), FastAPI/Python (backend), PostgreSQL + Redis, Google Maps / Mapbox.
- **Paiement** : escrow via **Cash Plus** ; commission CarGova prélevée au règlement final.

## Points restant à définir avec l'équipe CarGova

- Mode de calcul du prix de base + seuils (plancher industrie / plafond chauffeur) de la négociation encadrée.
- Taux exact de la commission CarGova.
- Détail des permissions et écrans du tableau de bord administrateur.
- Processus détaillé de résolution des litiges (médiation, remboursement partiel, arbitrage).
