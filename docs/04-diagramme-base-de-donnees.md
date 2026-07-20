# 04 — Diagramme de base de données (Entité-Association)

Modèle relationnel proposé pour **PostgreSQL** (données transactionnelles & utilisateurs).
Le suivi temps réel des positions chauffeurs s'appuie sur **Redis** (hors schéma relationnel ci-dessous).

```mermaid
erDiagram
    USERS ||--o{ KYC_DOCUMENTS : "soumet"
    USERS ||--o| INDUSTRIES : "profil"
    USERS ||--o| TRANSPORT_COMPANIES : "profil"
    USERS ||--o| DRIVERS : "profil"
    USERS ||--o{ NOTIFICATIONS : "reçoit"
    USERS ||--o{ BIDS : "soumet"

    TRANSPORT_COMPANIES ||--o{ DRIVERS : "emploie"
    TRANSPORT_COMPANIES ||--o{ TRUCKS : "possède"
    INDUSTRIES ||--o{ TRUCKS : "possède"
    DRIVERS ||--o| TRUCKS : "conduit"

    INDUSTRIES ||--o{ OFFERS : "publie"
    LINES ||--o{ OFFERS : "concerne"
    LINES ||--o| LINES : "ligne_retour"

    OFFERS ||--o{ BIDS : "reçoit"
    OFFERS ||--o| MISSIONS : "génère"

    MISSIONS ||--o{ MISSION_STEPS : "comporte"
    MISSIONS ||--|| PAYMENTS : "réglée_par"
    MISSIONS ||--o| WAITING_PENALTIES : "génère"
    MISSIONS ||--o{ REVIEWS : "évaluée_via"
    DRIVERS ||--o{ MISSIONS : "exécute"

    PAYMENTS ||--o{ CLAIMS : "bloqué_par"

    INDUSTRIES ||--o{ FAVORITES : "constitue"
    DRIVERS ||--o{ FAVORITES : "ciblé_par"

    USERS {
        uuid id PK
        string email UK
        string phone UK
        string password_hash
        enum role
        enum kyc_status
        enum language
        timestamp created_at
    }

    INDUSTRIES {
        uuid user_id PK,FK
        string company_name
        string trade_register
        string address
    }

    TRANSPORT_COMPANIES {
        uuid user_id PK,FK
        string company_name
        string trade_register
    }

    DRIVERS {
        uuid user_id PK,FK
        uuid company_id FK "nullable (solo si null)"
        string first_name
        string last_name
        string license_number
        enum driver_type
        float rating
    }

    KYC_DOCUMENTS {
        uuid id PK
        uuid user_id FK
        enum doc_type
        string file_url
        enum status
        timestamp reviewed_at
    }

    TRUCKS {
        uuid id PK
        uuid owner_id FK "société ou industrie"
        uuid driver_id FK "nullable"
        enum truck_type
        float capacity_weight
        float capacity_volume
        string plate_number
        bool available
    }

    LINES {
        uuid id PK
        string origin
        string destination
        uuid return_line_id FK "nullable"
        bool active
    }

    OFFERS {
        uuid id PK
        uuid industry_id FK
        uuid line_id FK
        enum required_truck_type
        float weight
        float volume
        enum loading_by
        string product_details
        string pickup_location
        string dropoff_location
        int estimated_loading_time
        float base_price
        float min_price
        float max_price
        enum status
        bool is_return_freight
        timestamp created_at
    }

    BIDS {
        uuid id PK
        uuid offer_id FK
        uuid user_id FK
        float amount
        enum side
        enum status
        timestamp created_at
    }

    MISSIONS {
        uuid id PK
        uuid offer_id FK
        uuid driver_id FK
        enum status
        float agreed_price
        bool return_unlocked
        timestamp started_at
        timestamp completed_at
    }

    MISSION_STEPS {
        uuid id PK
        uuid mission_id FK
        enum step_type
        float gps_lat
        float gps_lng
        timestamp step_timestamp
        bool confirmed_by_driver
        bool confirmed_by_industry
    }

    PAYMENTS {
        uuid id PK
        uuid mission_id FK
        float amount
        float cash_plus_fee
        float commission
        float net_to_carrier
        enum status
        timestamp paid_at
        timestamp released_at
    }

    CLAIMS {
        uuid id PK
        uuid payment_id FK
        uuid opened_by FK
        string reason
        enum status
        timestamp opened_at
        timestamp resolved_at
    }

    WAITING_PENALTIES {
        uuid id PK
        uuid mission_id FK
        int extra_minutes
        float amount
    }

    REVIEWS {
        uuid id PK
        uuid mission_id FK
        uuid author_id FK
        uuid target_id FK
        int rating
        string comment
        enum direction
        timestamp created_at
    }

    FAVORITES {
        uuid id PK
        uuid industry_id FK
        uuid driver_id FK
        timestamp created_at
    }

    NOTIFICATIONS {
        uuid id PK
        uuid user_id FK
        enum notif_type
        string message
        bool is_read
        timestamp created_at
    }
```

## Notes de conception

- **Héritage `USERS` → profils** : un compte `USERS` central porte l'authentification, le rôle, le
  statut KYC et la langue ; les tables `INDUSTRIES`, `TRANSPORT_COMPANIES`, `DRIVERS` en héritent via
  une clé primaire partagée (`user_id`).
- **Ligne retour** : `LINES.return_line_id` référence la ligne inverse ; c'est ce lien qui permet le
  **déblocage prioritaire** du fret retour à l'acceptation d'un aller.
- **Escrow** : `PAYMENTS.status` passe par `ESCROWED → RELEASED` (ou `ON_HOLD` si une `CLAIMS` est
  ouverte, puis `REFUNDED` le cas échéant).
- **Suivi temps réel** : les positions GPS instantanées des chauffeurs vivent dans **Redis** ; seules
  les étapes horodatées et confirmées sont persistées dans `MISSION_STEPS`.
- **Notation double sens** : `REVIEWS.direction` distingue industrie→chauffeur et chauffeur→industrie.
