# Cahier des charges — Plateforme de présence et relecture par les pairs (KFOKAM48)

Auteur : `204` · Version 1 (Étape 1 — analyse) · Frontend : **React**, parce que son écosystème de composants et son intégration avec une couche d'appel API dédiée conviennent bien à des écrans simples et indépendants (formateur / étudiant / relecteur).

---

## 1. Contexte et objectif

La direction de la formation KFOKAM48 souhaite outiller trois moments récurrents de la vie d'une
session de cours :

1. la prise de présence, aujourd'hui non tracée numériquement ;
2. le dépôt d'exercices par les étudiants ;
3. la relecture par les pairs (un étudiant note et commente l'exercice d'un autre), qui remplace ou
   complète la correction par le formateur.

L'objectif de l'application est de donner au formateur une vue unique, par étudiant, de sa présence,
de ses dépôts et de ses notes, sans qu'il ait à recouper des fichiers séparés — et de donner à
l'étudiant un parcours simple : entrer un code, déposer un lien, relire un pair.

## 2. Acteurs et rôles

| Acteur | Ce qu'il peut faire |
|---|---|
| **Formateur** | Ouvre une session et obtient un code de présence ; ajoute une présence manuellement ; clôture une session ; consulte le tableau de bord (présences, moyennes, relectures en attente) |
| **Étudiant** | Se choisit dans une liste (pas de mot de passe, Q1) ; saisit un code pour marquer sa présence ; dépose (et remplace) le lien de son exercice ; consulte sa propre note et le commentaire reçu |
| **Relecteur** | N'est pas un compte séparé : c'est un étudiant à qui le système assigne, au hasard, l'exercice d'un pair présent à la même session (Q7). Il note (0–20) et commente. |
| **Système** | Génère les codes de présence, gère leur expiration, assigne les relecteurs, calcule les moyennes exposées par l'API (le frontend ne recalcule rien) |

## 3. Périmètre

**Inclus**
- Ouverture de session et génération de code de présence, avec expiration.
- Marquage de présence par code, y compris ajout manuel par le formateur.
- Dépôt et remplacement du lien d'exercice.
- Assignation automatique et aléatoire d'un relecteur, notation et commentaire.
- Clôture de session par le formateur (nécessaire au fonctionnement des règles RG9, RG11, RG12, RG14 — voir §7, point Z1).
- Tableau de bord formateur par étudiant : présences, exercices déposés, moyenne, relectures en attente.
- Consultation par l'étudiant de sa propre note et du commentaire reçu, sans identité du relecteur.

**Exclu explicitement**
- Authentification par mot de passe ou tout mécanisme de compte sécurisé (Q1).
- Création/gestion des promotions et des listes d'étudiants (supposées pré-existantes, voir §7 point Z3).
- Notifications (email, push, SMS).
- Contenu pédagogique, planning des cours, paiement, gestion administrative.
- Interface d'administration générique (back-office autre que le tableau formateur).

## 4. Exigences fonctionnelles

| Réf | Exigence | Critère d'acceptation | Priorité |
|---|---|---|---|
| EF1 | Le formateur ouvre une session et obtient un code de présence | En créant une session (titre + promotionId), le système retourne un code, une date d'ouverture et une date d'expiration (ouverture + 15 min) | Must |
| EF2 | L'étudiant marque sa présence avec un code valide | Un code valide, saisi avant expiration et avant clôture de la session, crée une présence (201) visible immédiatement dans le tableau du formateur | Must |
| EF3 | Le système refuse un code expiré | Passé le délai de 15 min (RG1), toute tentative renvoie 410 `CODE_EXPIRE` | Must |
| EF4 | Le système empêche une double présence | Une deuxième tentative du même étudiant sur la même session renvoie 409 `DEJA_PRESENT` | Must |
| EF5 | Le système limite les essais de code erronés | Après 5 échecs consécutifs sur une session, les tentatives suivantes sont bloquées 2 minutes | Should |
| EF6 | L'étudiant dépose le lien de son exercice | Un dépôt (sessionId, etudiantId, lien) avant clôture de la session crée l'exercice au statut « déposé » (201) | Must |
| EF7 | L'étudiant remplace le lien de son exercice | Le remplacement n'est accepté que si aucune relecture n'a débuté sur cet exercice (Q13) | Should |
| EF8 | Un relecteur est assigné automatiquement à chaque exercice déposé | Le relecteur est choisi au hasard parmi les étudiants présents à la session, en excluant l'auteur de l'exercice | Must |
| EF9 | Le relecteur note et commente l'exercice assigné | Une note entière 0–20 et un commentaire sont acceptés (200) ; une note hors bornes ou non entière renvoie 400 | Must |
| EF10 | Le système empêche l'auto-relecture | Une tentative de relecture de son propre exercice renvoie 403 `AUTO_RELECTURE` | Must |
| EF11 | Le système empêche une double relecture | Une deuxième soumission de relecture sur le même exercice renvoie 409 `RELECTURE_DEJA_RENDUE` | Must |
| EF12 | L'étudiant relu consulte sa note et le commentaire, sans connaître le relecteur | La réponse exposée à l'étudiant ne contient jamais l'identifiant ni le nom du relecteur | Must |
| EF13 | Le relecteur corrige sa note tant que la session n'est pas clôturée | Toute modification est acceptée si le statut de la session ≠ clôturée, refusée sinon (résout la contradiction Q10/Q15, voir §7) | Should |
| EF14 | Le formateur ajoute une présence manuellement | La présence créée porte `source = FORMATEUR` et est visiblement distinguée d'une présence par code (`source = ETUDIANT`) | Should |
| EF15 | Le formateur consulte le tableau de bord par étudiant | Pour `GET /api/tableau?promotionId=...`, chaque ligne expose présences, exercices déposés, moyenne des notes, relectures en attente | Must |
| EF16 | Le formateur clôture une session | Une session clôturée n'accepte plus de nouvelles présences, dépôts, remplacements ou modifications de relecture | Must |

## 5. Exigences non fonctionnelles

| Réf | Exigence | Comment on la vérifie |
|---|---|---|
| ENF1 | Utilisation en mobile (étudiants en amphithéâtre, sur téléphone) | Les trois écrans (formateur, étudiant, relecteur) restent utilisables sur un écran de 375px de large |
| ENF2 | Temps de réponse perçu court sur la saisie de code | Réponse API < 1s en conditions normales (hors charge exceptionnelle, non testée en examen) |
| ENF3 | Cohérence des données affichées | La moyenne, les compteurs de présence et le statut des relectures affichés proviennent toujours de l'API — aucun recalcul côté frontend (contrainte F3 du sujet) |
| ENF4 | Traçabilité des erreurs | Toute erreur API renvoie le format imposé `{ "code": ..., "message": ... }`, jamais une trace technique brute |
| ENF5 | Démarrage reproductible | L'application démarre depuis un clone vierge avec des données de démonstration (contrainte technique du sujet) |

## 6. Règles de gestion

| Réf | Règle | Source |
|---|---|---|
| RG1 | Le code de présence expire 15 minutes après l'ouverture de la session | Q2 |
| RG2 | Impossible de marquer une présence après expiration du code ou après clôture de la session | Q2, Q3, Z1 |
| RG3 | Après 5 tentatives de code erronées sur une session, blocage de 2 minutes | Q4 |
| RG4 | Un étudiant ne peut jamais relire son propre exercice | Q5 |
| RG5 | Un exercice n'a qu'un seul relecteur | Q6 |
| RG6 | Le relecteur est choisi au hasard parmi les étudiants présents à la session concernée | Q7 |
| RG7 | L'étudiant relu voit sa note et le commentaire, jamais l'identité du relecteur | Q8 |
| RG8 | La note est un entier compris entre 0 et 20 | Q9 |
| RG9 | Le relecteur peut modifier sa note/commentaire tant que le formateur n'a pas clôturé la session ; passé la clôture, la note devient définitive et non modifiable | Q10 et Q15 (contradiction tranchée, voir §7 point C1) |
| RG10 | Un exercice sans relecture rendue reste au statut « en attente de relecture », visible distinctement dans le tableau formateur | Q11 |
| RG11 | Le dépôt d'un exercice est possible jusqu'à la clôture de la session | Q12 |
| RG12 | Le lien d'un exercice peut être remplacé tant qu'aucune relecture n'a débuté | Q13 |
| RG13 | Une présence ajoutée manuellement par le formateur porte la mention `source = FORMATEUR` | Q14 |
| RG14 | Une session clôturée ne peut plus recevoir de présence, de dépôt, ni de modification de relecture | Q10, Q12, Q15, Z1 (hypothèse — voir §7) |
| RG15 | Le tableau formateur agrège, par étudiant : présence à chaque session, nombre d'exercices déposés, moyenne des notes reçues, relectures encore à faire | Q16 |

## 7. Zones d'ombre, hypothèses et contradictions

| Réf | Qx concernées | Problème | Décision retenue | Pourquoi |
|---|---|---|---|---|
| **C1** | Q10, Q15 | Q10 dit que le relecteur peut corriger sa note tant que la session n'est pas clôturée. Q15 dit que la note est définitive dès qu'elle est validée. Ces deux réponses sont contradictoires si on lit « validation » (Q15) comme « soumission » (Q10). | On retient Q10 comme règle opérationnelle (RG9) : la note est modifiable jusqu'à la **clôture de la session** par le formateur. On réinterprète le mot « validation » de Q15 comme désignant cette clôture, et non l'instant de soumission initiale de la note. | Q11 décrit un usage réel qui suppose des relectures en attente sur une session encore ouverte — donc un état intermédiaire nécessairement modifiable. Sans clôture comme point de bascule, Q12 (dépôt possible jusqu'à clôture) et Q10 deviendraient incohérents entre eux. La clôture de session est le seul événement métier explicite qui peut jouer le rôle de « validation finale » évoqué en Q15. |
| **Z1** | Q10, Q12, Q13, Q15 | Le client mentionne à quatre reprises la « clôture de la session » comme événement déclencheur, mais **aucune des 16 questions ne décrit l'action de clôturer une session**, ni qui la déclenche, ni ce qu'elle bloque exactement. C'est le trou fonctionnel principal du sujet. | On introduit une action explicite « clôturer une session », déclenchée par le formateur (EF16, RG14), qui verrouille : nouvelles présences, nouveaux dépôts, remplacements de lien, et modifications de relecture. | Sans cette action, RG9, RG11, RG12 et la contradiction C1 n'ont pas de point d'ancrage temporel vérifiable — le cahier des charges serait incohérent avec lui-même. |
| **Z2** | Q7 | Le relecteur est choisi « au hasard parmi les étudiants présents à cette session ». Le client ne dit pas ce qui se passe si **aucun autre étudiant n'est présent** (session à un seul participant, ou dépôt fait après la session par un étudiant absent — cf. Z4). | Hypothèse : l'exercice reste au statut « en attente de relecture » (comme en Q11) jusqu'à ce qu'un candidat relecteur devienne éligible, ou jusqu'à intervention manuelle du formateur (hors périmètre de l'automatisation, à discuter en §7 pour l'étape 3). | Le client a explicitement prévu l'état « en attente » (Q11) pour l'absence de relecture rendue ; on réutilise cet état plutôt que d'inventer un comportement d'échec non demandé. |
| **Z3** | Q1 | « L'étudiant choisit son nom dans une liste » suppose une liste d'étudiants déjà existante, mais aucune question ne couvre la création ou l'import des étudiants et des promotions. | Hypothèse : la liste des étudiants et des promotions est pré-chargée en données de démonstration (contrainte « données de démo au démarrage » du sujet), sans écran de gestion dédié. | Ce n'est pas un besoin exprimé par le client (items 1 à 5) ; en faire un écran complet consommerait du temps hors périmètre noté. |
| **Z4** | Q3, Q12 | Q3 dit qu'on ne peut pas marquer sa présence après la fin de la session, mais Q12 autorise le dépôt d'exercice jusqu'à la clôture — deux instants potentiellement différents (« fin de session » vs « clôture »). Le client ne précise pas si ces deux instants sont identiques. | Hypothèse : « fin de session » (Q3) = expiration du code (RG1, 15 min après ouverture) ; « clôture » (Q12) = action explicite et distincte du formateur (Z1), qui peut survenir bien plus tard. Un étudiant peut donc déposer un exercice sans avoir pu marquer sa présence si son code a expiré entre-temps. | C'est la seule lecture qui rend Q12 utile : le client justifie lui-même ce délai (« certains n'ont pas de connexion le soir même »), ce qui n'aurait aucun sens si « fin de session » et « clôture » étaient le même instant. |
| **Z5** | Q6, Q7 | Rien n'indique **quand** l'assignation du relecteur a lieu : immédiatement au dépôt de l'exercice, ou à un moment choisi par le formateur/le système en différé. | Hypothèse : assignation automatique et immédiate à la validation du dépôt (EF8), pour rester cohérent avec une API sans opération d'assignation manuelle explicitement demandée. | Simplicité et cohérence avec les 5 opérations imposées par le contrat, qui ne prévoient pas d'endpoint d'assignation séparé. |

## 8. Contraintes techniques

**Backend**
- B1 : Java 21, Maven, wrapper `mvnw` commité.
- B2 : `api/contrat.yaml` respecté à la lettre (chemins, verbes, codes HTTP, format d'erreur).
- B3 : Séparation contrôleur / service / repository ; DTO obligatoires, aucune entité JPA exposée en JSON.
- B4 : Validation des entrées, gestion centralisée des erreurs (`@RestControllerAdvice`), jamais de stack trace exposée.
- B5 : Migrations versionnées (Flyway ou Liquibase), `ddl-auto=update` interdit hors tests.
- B6 : Un test unitaire sur une règle métier réelle + un test d'intégration sur un endpoint, exécutables sur poste vierge.

**Frontend**
- F1 : Framework déclaré et justifié en une ligne dans le README, build fonctionnel.
- F2 : Trois écrans — formateur (session, tableau), étudiant (présence, dépôt), relecteur (relecture).
- F3 : Couche d'appel API dédiée, états de chargement/erreur gérés, aucune règle métier dupliquée côté frontend (moyenne = celle de l'API).

**Démarrage** : `docker compose up` ou trois commandes documentées, testées depuis un clone vierge, avec données de démonstration.

## 9. Livrables

- `docs/CAHIER_DES_CHARGES.md`.
- `docs/diagrammes/D1_cas_utilisation.md`, `D2_modele_donnees.md`, `D3_sequence_presence.md` (Mermaid, versionnés).
- `docs/diagrammes/D4_etats_exercice.md` (bonus, cycle de vie de l'exercice).
- Backlog : issues créées sur le dépôt GitHub (liste proposée dans `docs/BACKLOG_ISSUES.md`).
- `api/contrat.yaml` complété (5 opérations imposées + opérations complémentaires justifiées).
- `docs/ANALYSE_16_QUESTIONS.md` (table Qx / information / impact / décision).
- `docs/COHERENCE_CHECKLIST.md` (vérification croisée avant jalon `[JALON] analyse`).
- Commit `[JALON] analyse`, vide de code, précédant tout commit applicatif.

## 10. Démarche prévue

1. **Étape 1 (en cours)** — Valider ensemble ce cahier des charges, les diagrammes, le backlog et le contrat d'API ; trancher le point ouvert du §7 (frontend) ; poser `.gitignore` Java+JS ; committer `[JALON] analyse`.
2. **Étape 2** — Ne développer que les stories `Must` du backlog. Une branche par issue, une PR par branche, fermeture des issues par les commits/PR (`Closes #N`), migrations Flyway/Liquibase dès le premier schéma. Commit `[JALON] v0.1`.
3. **Étape 3** — Une fois `[JALON] v0.1` poussé, demander l'enveloppe au surveillant. Traiter bug et évolution comme deux changements séparés (issue, migration, contrat, frontend). Mettre à jour ce cahier des charges et les diagrammes dans un commit explicite si le besoin change.
4. **Étape 4** — Compléter les stories `Should`/`Could` restantes selon le temps disponible, `CHANGELOG.md`, README testé depuis un clone vierge, commit `[JALON] v1.0`.
5. **Étape 5** — `SOUMISSION.md` avant 18h00.

**Definition of Done** (une issue est terminée quand) : le code respecte B1–B6/F1–F3 pertinents pour
l'issue, ses critères d'acceptation sont vérifiés manuellement ou par test, la PR est fusionnée sur
`main`, l'issue est fermée automatiquement par la PR (`Closes #N`), et `JOURNAL.md` est mis à jour.