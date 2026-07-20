# 03 — Diagramme de classes (modèle du domaine)

Modèle objet simplifié du domaine CarGova (backend FastAPI).

```mermaid
classDiagram
    class User {
        +UUID id
        +string email
        +string phone
        +string passwordHash
        +Role role
        +KycStatus kycStatus
        +Language language
        +datetime createdAt
    }

    class Industry {
        +string companyName
        +string tradeRegister
        +string address
    }

    class TransportCompany {
        +string companyName
        +string tradeRegister
    }

    class Driver {
        +string firstName
        +string lastName
        +string licenseNumber
        +DriverType type
        +float rating
    }

    class KycDocument {
        +UUID id
        +DocType type
        +string fileUrl
        +KycStatus status
        +datetime reviewedAt
    }

    class Truck {
        +UUID id
        +TruckType type
        +float capacityWeight
        +float capacityVolume
        +string plateNumber
        +bool available
    }

    class Line {
        +UUID id
        +string origin
        +string destination
        +bool active
    }

    class Offer {
        +UUID id
        +TruckType requiredTruckType
        +float weight
        +float volume
        +LoadingResponsibility loadingBy
        +string productDetails
        +string pickupLocation
        +string dropoffLocation
        +int estimatedLoadingTime
        +float basePrice
        +float minPrice
        +float maxPrice
        +OfferStatus status
        +bool isReturnFreight
        +datetime createdAt
    }

    class Bid {
        +UUID id
        +float amount
        +BidSide side
        +BidStatus status
        +datetime createdAt
    }

    class Mission {
        +UUID id
        +MissionStatus status
        +float agreedPrice
        +datetime startedAt
        +datetime completedAt
        +bool returnUnlocked
    }

    class MissionStep {
        +UUID id
        +StepType type
        +GeoPoint gps
        +datetime timestamp
        +bool confirmedByDriver
        +bool confirmedByIndustry
    }

    class Payment {
        +UUID id
        +float amount
        +float cashPlusFee
        +float commission
        +float netToCarrier
        +PaymentStatus status
        +datetime paidAt
        +datetime releasedAt
    }

    class Claim {
        +UUID id
        +string reason
        +ClaimStatus status
        +datetime openedAt
        +datetime resolvedAt
    }

    class Review {
        +UUID id
        +int rating
        +string comment
        +ReviewDirection direction
        +datetime createdAt
    }

    class Favorite {
        +UUID id
        +datetime createdAt
    }

    class Notification {
        +UUID id
        +NotifType type
        +string message
        +bool read
        +datetime createdAt
    }

    class WaitingPenalty {
        +UUID id
        +int extraMinutes
        +float amount
    }

    %% Héritage (profils rattachés au compte User)
    User <|-- Industry
    User <|-- TransportCompany
    User <|-- Driver

    %% Associations
    User "1" --> "*" KycDocument : soumet
    TransportCompany "1" --> "*" Driver : emploie
    TransportCompany "1" --> "*" Truck : possède
    Driver "1" --> "0..1" Truck : conduit
    Industry "1" --> "*" Truck : possède
    Industry "1" --> "*" Offer : publie
    Offer "*" --> "1" Line : sur
    Offer "1" --> "*" Bid : reçoit
    User "1" --> "*" Bid : soumet
    Offer "1" --> "0..1" Mission : donne lieu à
    Mission "1" --> "*" MissionStep : comporte
    Mission "1" --> "1" Driver : exécutée par
    Mission "1" --> "1" Payment : réglée par
    Payment "1" --> "0..*" Claim : peut être bloqué par
    Mission "1" --> "0..*" Review : évaluée via
    Mission "1" --> "0..1" WaitingPenalty : génère
    Industry "1" --> "*" Favorite : constitue
    Favorite "*" --> "1" Driver : cible
    User "1" --> "*" Notification : reçoit
    Line "1" --> "0..1" Line : ligne retour
```

## Énumérations principales

| Enum | Valeurs |
|------|---------|
| `Role` | INDUSTRY, TRANSPORT_COMPANY, DRIVER, ADMIN |
| `DriverType` | SOLO, ATTACHED |
| `KycStatus` | PENDING, APPROVED, REJECTED |
| `DocType` | DRIVING_LICENSE, INSURANCE, TRUCK_STATE, TRADE_REGISTER |
| `TruckType` | SEMI_TRAILER, REFRIGERATED, FLATBED, SMALL_VAN, ... |
| `LoadingResponsibility` | INDUSTRY, DRIVER |
| `OfferStatus` | DRAFT, PUBLISHED, NEGOTIATING, ACCEPTED, CANCELLED |
| `BidSide` | INDUSTRY_DOWN, DRIVER_UP |
| `BidStatus` | PENDING, ACCEPTED, REJECTED |
| `MissionStatus` | CREATED, STARTED, AT_PICKUP, LOADED, IN_TRANSIT, AT_DROPOFF, DELIVERED, CLOSED |
| `StepType` | START, PICKUP_ARRIVAL, LOADING_CONFIRMED, IN_TRANSIT, DESTINATION_ARRIVAL, DROPOFF_DECLARED, DROPOFF_CONFIRMED |
| `PaymentStatus` | ESCROWED, RELEASED, REFUNDED, ON_HOLD |
| `ClaimStatus` | OPEN, RESOLVED, REJECTED |
| `ReviewDirection` | INDUSTRY_TO_DRIVER, DRIVER_TO_INDUSTRY |
| `NotifType` | RETURN_OPPORTUNITY, TRUCK_ARRIVAL, LOADING_DELAY, PAYMENT, CLAIM, ... |
| `Language` | AR, FR, EN |
