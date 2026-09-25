# Analyse des 16 questions client

| Qx | Information | Impact métier | Décision / RG associé |
|---|---|---|---|
| Q1 | Pas de mot de passe, choix du nom dans une liste | Simplifie l'authentification, mais suppose une liste d'étudiants pré-existante (voir Z3) | Aucune RG dédiée — traité comme contrainte de conception (pas de compte sécurisé), et opération complémentaire `GET /api/etudiants` |
| Q2 | Code expire 15 min après ouverture | Fenêtre de présence limitée dans le temps | RG1 |
| Q3 | Pas de présence après la fin de session | Confirme une borne temporelle stricte, distincte de la clôture (voir Z4) | RG2 |
| Q4 | Blocage 2 min après 5 échecs | Anti-devinette de code entre étudiants | RG3, EF5 |
| Q5 | Interdiction de l'auto-relecture | Principe fondateur de la relecture par les pairs | RG4, EF10 |
| Q6 | Un seul relecteur par exercice | Simplifie l'agrégation des notes (une seule note par exercice) | RG5 |
| Q7 | Relecteur choisi au hasard parmi les présents | Nécessite de connaître la liste des présents au moment de l'assignation | RG6, EF8 |
| Q8 | L'étudiant voit note + commentaire, jamais le relecteur | Anonymat du relecteur à préserver côté API et frontend | RG7, EF12 |
| Q9 | Note entière sur 20 | Contrainte de validation stricte | RG8, EF9 |
| Q10 | Correction de note possible avant clôture | **Contradictoire avec Q15**, voir tranché en C1 | RG9, EF13 |
| Q11 | Exercice « en attente » si relecture non rendue | État intermédiaire explicite à modéliser | RG10 ; statut `EN_ATTENTE_RELECTURE` (D2) |
| Q12 | Dépôt possible jusqu'à clôture de session | Distingue « fin de session » (Q3) de « clôture » (voir Z4) | RG11 |
| Q13 | Remplacement du lien tant qu'aucune relecture n'a débuté | Contrainte d'intégrité entre `Exercice` et `Relecture` | RG12, EF7 |
| Q14 | Ajout de présence manuel par le formateur, marqué comme tel | Nécessite un champ `source` sur `Presence` | RG13, EF14 |
| Q15 | Note définitive dès validation | **Contradictoire avec Q10**, voir C1 | RG9 (réinterprété : « validation » = clôture de session) |
| Q16 | Contenu du tableau formateur : présence par session, exercices déposés, moyenne, relectures à faire | Définit exactement les colonnes de `GET /api/tableau` | RG15, EF15 |

**Contradiction identifiée** : Q10 / Q15 — tranchée en C1 dans `CAHIER_DES_CHARGES.md` §7. Décision
retenue : la note reste modifiable jusqu'à la clôture de la session (RG9), la « validation » de Q15
étant réinterprétée comme cette clôture plutôt que comme l'instant de soumission de la note.

**Trou fonctionnel principal identifié** : aucune des 16 questions ne décrit l'action de « clôturer une
session », alors qu'elle est présupposée par Q10, Q12, Q13 et Q15. Traité en Z1 (cahier des charges §7)
par l'introduction d'EF16 et de l'opération complémentaire `PATCH /api/sessions/{id}/cloturer`.

**Question sans impact direct sur la conception** : Q1 ne définit pas de règle de gestion à proprement
parler (« ne perdez pas de temps là-dessus », dit le client) — elle cadre simplement l'absence
d'authentification.
