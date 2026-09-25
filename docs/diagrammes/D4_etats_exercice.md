# D4 — États-transitions : cycle de vie d'un exercice (bonus)

```mermaid
stateDiagram-v2
    [*] --> DEPOSE : dépôt du lien (EF6)

    DEPOSE --> EN_ATTENTE_RELECTURE : relecteur assigné (EF8, RG6)

    DEPOSE --> DEPOSE : lien remplacé\n(EF7, RG12 — tant qu'aucune relecture n'a débuté)

    EN_ATTENTE_RELECTURE --> RELU : relecture soumise (EF9)

    RELU --> RELU : note/commentaire corrigés\n(EF13, RG9 — tant que session non clôturée)

    RELU --> [*] : session clôturée (RG14, note définitive)
    EN_ATTENTE_RELECTURE --> [*] : session clôturée,\nrelecture jamais rendue (RG10)
