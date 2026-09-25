# D2 — Modèle de données (diagramme de classes)

Formalisme : Mermaid `classDiagram`. Suffisamment précis pour être traduit directement en migrations
(types, contraintes d'unicité en commentaire, cardinalités explicites).

```mermaid
classDiagram
    class Promotion {
        +UUID id
        +String nom
    }

    class Etudiant {
        +UUID id
        +String nom
        +UUID promotionId
    }

    class Session {
        +UUID id
        +String titre
        +UUID promotionId
        +String code
        +DateTime ouvertureAt
        +DateTime expirationAt
        +StatutSession statut
    }

    class StatutSession {
        <<enumeration>>
        OUVERTE
        CLOTUREE
    }

    class Presence {
        +UUID id
        +UUID sessionId
        +UUID etudiantId
        +SourcePresence source
        +DateTime marqueeAt
    }

    class SourcePresence {
        <<enumeration>>
        ETUDIANT
        FORMATEUR
    }

    class Exercice {
        +UUID id
        +UUID sessionId
        +UUID etudiantId
        +String lien
        +StatutExercice statut
        +DateTime deposeAt
    }

    class StatutExercice {
        <<enumeration>>
        DEPOSE
        EN_ATTENTE_RELECTURE
        RELU
    }

    class Relecture {
        +UUID id
        +UUID exerciceId
        +UUID relecteurId
        +Integer note
        +String commentaire
        +DateTime soumiseAt
        +DateTime modifieeAt
    }

    class TentativeCode {
        +UUID id
        +UUID sessionId
        +UUID etudiantId
        +Integer echecs
        +DateTime bloqueJusqua
    }

    Promotion "1" --> "*" Etudiant : regroupe
    Promotion "1" --> "*" Session : concerne
    Session "1" --> "*" Presence : enregistre
    Session "1" --> "*" Exercice : recoit
    Etudiant "1" --> "*" Presence : marque
    Etudiant "1" --> "*" Exercice : depose
    Etudiant "1" --> "*" Relecture : relit (rôle relecteur)
    Exercice "1" --> "0..1" Relecture : est relu par
    Session "1" --> "*" TentativeCode : compte les echecs de

    Session --> StatutSession
    Presence --> SourcePresence
    Exercice --> StatutExercice
```

**Contraintes portées par ce modèle (à traduire en migrations, étape 2)**
- `Presence` : unicité (`sessionId`, `etudiantId`) → applique RG contre la double présence (EF4).
- `Exercice` : unicité (`sessionId`, `etudiantId`) → un seul exercice actif par étudiant et par session.
- `Relecture` : unicité (`exerciceId`) → un seul relecteur par exercice (RG5) ; `relecteurId` ≠
  `etudiant_id` de l'exercice associé (RG4, à vérifier en base et en service).
- `Relecture.note` : entier, contrainte `CHECK (note BETWEEN 0 AND 20)` (RG8).
- `Session.statut` conditionne, au niveau service, l'écriture sur `Presence`, `Exercice` et `Relecture`
  (RG2, RG9, RG11, RG12, RG14).
- `TentativeCode` porte le compteur d'échecs et la fenêtre de blocage de 2 minutes (RG3, EF5).
