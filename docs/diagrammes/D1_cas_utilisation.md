# D1 — Cas d'utilisation

Formalisme : Mermaid `flowchart`

```mermaid
flowchart LR
    Formateur(["👤 Formateur"])
    Etudiant(["👤 Étudiant"])
    Relecteur(["👤 Étudiant — rôle relecteur"])
    Systeme(["⚙️ Système"])

    UC1(["Ouvrir une session
    et obtenir un code"])
    UC2(["Marquer sa présence
    avec un code"])
    UC3(["Déposer / remplacer
    le lien d'un exercice"])
    UC4(["Noter et commenter
    un exercice assigné"])
    UC5(["Consulter le tableau
    présence + moyennes"])
    UC6(["Ajouter une présence
    manuellement"])
    UC7(["Clôturer la session"])
    UC8(["Consulter sa note
    et le commentaire reçu"])
    UC9(["Assigner un relecteur
    au hasard (RG6)"])

    Formateur --> UC1
    Formateur --> UC5
    Formateur --> UC6
    Formateur --> UC7

    Etudiant --> UC2
    Etudiant --> UC3
    Etudiant --> UC8

    Relecteur --> UC4

    Systeme --> UC9
    UC3 -.déclenche.-> UC9
    UC9 -.assigne à.-> Relecteur

    UC2 -.contraint par RG1 RG2 RG3.-> UC1
    UC4 -.contraint par RG4 RG5 RG7 RG8 RG9.-> UC9
```

**Notes de lecture**
- Le rôle « Relecteur » n'est pas un compte distinct : c'est un étudiant, désigné automatiquement par
  le système (UC9) parmi les étudiants présents à la session (RG6).
- UC9 est un cas d'usage système, sans interaction humaine directe : il est déclenché par UC3 (dépôt
  d'exercice), voir hypothèse Z5 du cahier des charges.
- UC7 (clôturer la session) est l'action introduite pour combler le trou fonctionnel Z1 — elle
  conditionne RG9, RG11, RG12, RG14.
