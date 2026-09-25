# D3 — Séquence : « marquer sa présence »

Cas couverts : nominal, code expiré, étudiant déjà présent. Codes HTTP alignés sur `api/contrat.yaml`.

```mermaid
sequenceDiagram
    participant E as Étudiant
    participant F as Frontend
    participant API as PresenceController
    participant S as PresenceService
    participant DB as Base de données

    E->>F: saisit le code
    F->>API: POST /api/presences { code, etudiantId }
    API->>S: enregistrer(code, etudiantId)
    S->>DB: rechercher session par code

    alt code inconnu
        DB-->>S: aucune session
        S-->>API: CodeInconnuException
        API-->>F: 400 { "code": "CODE_INCONNU", "message": "..." }
    else code expiré (RG1)
        DB-->>S: session trouvée, expirationAt dépassé
        S-->>API: CodeExpireException
        API-->>F: 410 { "code": "CODE_EXPIRE", "message": "Le code de présence a expiré." }
    else déjà présent (RG2, EF4)
        DB-->>S: session valide, présence existante pour cet étudiant
        S-->>API: DejaPresentException
        API-->>F: 409 { "code": "DEJA_PRESENT", "message": "..." }
    else cas nominal
        DB-->>S: session valide, aucune présence existante
        S->>DB: créer Presence(source = ETUDIANT)
        DB-->>S: Presence créée
        S-->>API: Presence
        API-->>F: 201 { id, sessionId, etudiantId, source }
    end

    F-->>E: confirmation ou message d'erreur
