# Backlog — issues à créer sur le dépôt

**Définition** : une issue = un titre orienté résultat + des critères d'acceptation vérifiables + une
priorité Must/Should/Could + une référence à une EFx ou une RGx du cahier des charges.

**Exemple d'issue complète** (celle utilisée ci-dessous pour la présence) :

> **Titre** : L'étudiant marque sa présence avec un code
> **Labels** : `Must`, `EF2`, `EF3`, `EF4`, `RG1`, `RG2`
> **Description** :
> En tant qu'étudiant, je veux saisir le code donné par le formateur pour que ma présence à la
> session soit enregistrée.
> **Critères d'acceptation** :
> - Un code valide, saisi avant expiration et avant clôture, crée ma présence et je le vois confirmé.
> - Un code expiré est refusé avec le message « code de présence expiré » (410).
> - Une deuxième tentative avec un code déjà utilisé par moi est refusée (409) sans créer de doublon.

**Découpage Git prévu** : une branche par issue (`feature/issue-N-slug`), une PR par branche, et un
commit ou la description de la PR contenant `Closes #N` — c'est cette mention qui ferme
automatiquement l'issue quand la PR est fusionnée sur `main`. Aucune fermeture manuelle d'issue.

---

### Issue 1 — Le formateur ouvre une session et obtient un code de présence
**Priorité** : Must · **Réf** : EF1, RG1
- Étant formateur, quand je crée une session avec un titre et une promotion, j'obtiens un code, une
  date d'ouverture et une date d'expiration (+15 min).
- Deux sessions ouvertes simultanément ont des codes différents.

### Issue 2 — L'étudiant marque sa présence avec un code
**Priorité** : Must · **Réf** : EF2, EF3, EF4, RG1, RG2
- Un code valide, saisi avant expiration et avant clôture, crée ma présence et je le vois confirmé.
- Un code expiré est refusé avec le message « code de présence expiré ».
- Une deuxième tentative avec un code déjà utilisé par moi est refusée sans créer de doublon.

### Issue 3 — Le système protège les codes contre les essais répétés
**Priorité** : Should · **Réf** : EF5, RG3
- Après 5 codes erronés consécutifs sur une session, mes tentatives suivantes sont refusées
  pendant 2 minutes, avec un message explicite.

### Issue 4 — L'étudiant dépose et remplace le lien de son exercice
**Priorité** : Must · **Réf** : EF6, EF7, RG11, RG12
- Je peux déposer le lien de mon exercice jusqu'à la clôture de la session.
- Je peux remplacer ce lien tant qu'aucun relecteur n'a commencé la relecture.
- Une fois la relecture débutée, la tentative de remplacement est refusée avec un message clair.

### Issue 5 — Un relecteur est assigné automatiquement à chaque exercice
**Priorité** : Must · **Réf** : EF8, RG4, RG5, RG6
- Après le dépôt d'un exercice, un seul relecteur est désigné, au hasard, parmi les étudiants
  présents à la session, en excluant l'auteur.
- Si aucun candidat n'est éligible, l'exercice reste visible comme « en attente de relecture ».

### Issue 6 — Le relecteur note et commente l'exercice qui lui est assigné
**Priorité** : Must · **Réf** : EF9, EF10, EF11, RG7, RG8
- Je peux soumettre une note entière (0–20) et un commentaire pour l'exercice qui m'est assigné.
- Une note hors bornes ou non entière est refusée avec un message clair.
- Je ne peux pas relire mon propre exercice.
- Je ne peux pas soumettre deux fois une relecture sur le même exercice.

### Issue 7 — Le relecteur peut corriger sa note tant que la session est ouverte
**Priorité** : Should · **Réf** : EF13, RG9
- Tant que le formateur n'a pas clôturé la session, je peux modifier ma note et mon commentaire.
- Après clôture, toute tentative de modification est refusée.

### Issue 8 — Le formateur ajoute une présence manuellement
**Priorité** : Should · **Réf** : EF14, RG13
- Je peux ajouter la présence d'un étudiant qui n'a pas pu saisir de code.
- Cette présence est visiblement distinguée d'une présence saisie par l'étudiant lui-même.

### Issue 9 — Le formateur consulte le tableau de bord par étudiant
**Priorité** : Must · **Réf** : EF15, RG15
- Pour une promotion donnée, je vois par étudiant : ses présences, le nombre d'exercices déposés,
  sa moyenne des notes reçues, et le nombre de relectures qu'il lui reste à faire.
- Les exercices en attente de relecture sont clairement identifiables (RG10).

### Issue 10 — Le formateur clôture une session
**Priorité** : Must · **Réf** : EF16, RG14
- Je peux clôturer une session ouverte.
- Après clôture, plus aucune présence, dépôt, remplacement ou modification de relecture n'est
  accepté sur cette session.

### Issue 11 — L'étudiant consulte sa note sans connaître le relecteur
**Priorité** : Must · **Réf** : EF12, RG7
- Je peux consulter la note et le commentaire reçus sur mon exercice.
- Je ne vois à aucun moment l'identité de mon relecteur.

### Issue 12 — Des données de démonstration sont chargées au démarrage
**Priorité** : Could · **Réf** : contrainte technique « démarrage » du sujet
- Au premier lancement, l'application contient une promotion, des étudiants, une session et
  quelques exercices/relectures d'exemple, pour que le correcteur puisse l'explorer immédiatement.

---

**Rappel de découpage Git** : chaque issue ⇒ une branche (`feature/issue-N-slug`) ⇒ une PR qui
référence l'issue (`Closes #N`) ⇒ commits atomiques sur la branche. Pas de « ticket » distinct de
l'« issue » : c'est le même objet, sous un seul nom désormais.
