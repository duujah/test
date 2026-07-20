# 02 — Diagramme de cas d'utilisation (Use Case)

Vue des interactions entre les acteurs et les grandes fonctionnalités de CarGova.

```mermaid
flowchart LR
    %% Acteurs
    IND(("👤 Industrie / Expéditeur"))
    SOL(("👤 Chauffeur indépendant"))
    SOC(("👤 Société de transport"))
    RAT(("👤 Chauffeur rattaché"))
    ADM(("👤 Administrateur"))
    CP["💳 Cash Plus"]
    MAP["🗺️ Google Maps / Mapbox"]

    subgraph CarGova["Plateforme CarGova"]
        UC1(["S'inscrire / soumettre KYC"])
        UC2(["Publier une offre de transport"])
        UC3(["Rechercher des offres"])
        UC4(["Négocier le prix (enchère encadrée)"])
        UC5(["Accepter une offre / mission"])
        UC6(["Débloquer la ligne retour prioritaire"])
        UC7(["Assigner une mission à un chauffeur"])
        UC8(["Exécuter la mission"])
        UC9(["Suivre la mission (GPS + confirmation double)"])
        UC10(["Payer sous séquestre (escrow)"])
        UC11(["Libérer / recevoir le paiement"])
        UC12(["Ouvrir une réclamation / gérer un litige"])
        UC13(["Évaluer l'autre partie"])
        UC14(["Gérer la liste de favoris"])
        UC15(["Recevoir des notifications"])
        UC16(["Administrer la plateforme (back-office)"])
    end

    %% Industrie
    IND --- UC1
    IND --- UC2
    IND --- UC4
    IND --- UC10
    IND --- UC12
    IND --- UC13
    IND --- UC14
    IND --- UC15

    %% Chauffeur indépendant
    SOL --- UC1
    SOL --- UC3
    SOL --- UC4
    SOL --- UC5
    SOL --- UC8
    SOL --- UC11
    SOL --- UC13
    SOL --- UC15

    %% Société de transport
    SOC --- UC1
    SOC --- UC3
    SOC --- UC4
    SOC --- UC5
    SOC --- UC7
    SOC --- UC11
    SOC --- UC13
    SOC --- UC15

    %% Chauffeur rattaché
    RAT --- UC8
    RAT --- UC9
    RAT --- UC15

    %% Administrateur
    ADM --- UC16

    %% Relations include / extend
    UC5 -. include .-> UC6
    UC5 -. include .-> UC10
    UC8 -. include .-> UC9
    UC11 -. extend .-> UC12

    %% Systèmes externes
    UC10 --- CP
    UC11 --- CP
    UC9 --- MAP
```

## Description des cas d'utilisation

| Cas | Acteur(s) | Résumé |
|-----|-----------|--------|
| S'inscrire / KYC | Tous | Création de compte + soumission des documents ; activation après validation |
| Publier une offre | Industrie | Type de camion, poids/volume, chargement, prise en charge & livraison |
| Rechercher des offres | Chauffeur solo, Société | Consulter les offres disponibles sur les lignes couvertes |
| Négocier le prix | Industrie, Chauffeur, Société | Enchère encadrée (plancher / plafond) |
| Accepter une offre | Chauffeur solo, Société | Déclenche le déblocage prioritaire du retour + le paiement escrow |
| Déblocage ligne retour | Système (déclenché par acceptation) | Accès prioritaire à la ligne retour |
| Assigner une mission | Société | Attribuer la mission à un chauffeur rattaché |
| Exécuter la mission | Chauffeurs | Réaliser le transport |
| Suivre la mission | Chauffeur + Industrie | Étapes GPS horodatées + confirmation croisée |
| Payer (escrow) | Industrie | Paiement séquestré via Cash Plus |
| Libérer / recevoir paiement | Système, Transporteur | Reversement du net après confirmation |
| Réclamation / litige | Industrie, Admin | Suspend le paiement jusqu'à résolution |
| Évaluer | Industrie, Chauffeur, Société | Notation à double sens |
| Gérer favoris | Industrie | Liste de chauffeurs de confiance |
| Notifications | Tous | Push sur événements critiques |
| Administrer | Administrateur | Gestion globale via back-office |
