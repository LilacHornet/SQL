# Rapport d'enquete

J'ai d'abord examiné la structure da la database en interface graphique avec une extension VSCODE pour voir les tables et les colonnes.

J'ai affiché tous les rapports de scene de crime avec :

```sql
SELECT * FROM crime_scene_report WHERE type='murder'
```

- SELECT sert à sélectionner quelles colonnes afficher.
  - Ici, on affiche utilise la wildcard \* pour afficher toutes les colonnes.
- FROM indique dans quelle table on va chercher les informations.
- WHERE indique de n'afficher que les données respectant les conditions.
  - La condition est que le type doit être murder  

Ca affichait trop de donner pour que ca soit exploitable exploitable.

J'ai décidé d'afficher les meurtes ayant été commis à SQL City.

```sql
SELECT * FROM crime_scene_report WHERE type='murder' AND city='SQL City'
```

- SELECT sert à sélectionner quelles colonnes afficher.
  - Ici, on affiche utilise la wildcard \* pour afficher toutes les colonnes.
- FROM indique dans quelle table on va chercher les informations.
- WHERE indique de n'afficher que les données respectant les conditions.
  - Le type doit être murder.
  - La ville doit être SQL city.

Resultat :

|date|type|description|city|
|---|---|---|---|
|20180215|murder|REDACTED REDACTED REDACTED|SQL City|
|20180215|murder|Someone killed the guard! He took an arrow to the knee!|SQL City|
|20180115|murder|Security footage shows that there were 2 witnesses. The first witness lives at the last house on "Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave".|SQL City|

Le troisieme resultat semble être le plus pertinent.
J'ai donc décidé de chercher l'identité des deux témoins.
Pour le premier témoin, on a ces informations :

1. Il vit sur Northwestern Dr.
2. Il a le dernier numéro de la rue.

```sql
SELECT id, name, address_number, address_street_name FROM person WHERE address_street_name = 'Northwestern Dr' AND address_number = (SELECT MAX(address_number) FROM person WHERE address_street_name = 'Northwestern Dr');
```

- SELECT sert à sélectionner quelles colonnes afficher.
- FROM indique dans quelle table on va chercher les informations.
- WHERE indique de n'afficher que les données respectant les conditions.
  - La rue doit être Northwestern Dr.
  - Le numéro de rue doit être le  plus grand numéro de rue existant sur la Northwestern Dr.
    - MAX renvoie le plus grand élément des resultats.

Résultat :

|id|name|address_number|address_street_name|
|---|---|---|---|
|14887|Morty Schapiro|4919|Northwestern Dr|

Pour le deuxieme témoin :

1. Elle s'appelle Annabel
2. Elle vit sur Franklin Ave

```sql
SELECT id, name, address_number, address_street_name FROM person WHERE address_street_name = 'Franklin Ave' AND name LIKE 'Annabel %';
```

- SELECT sert à sélectionner quelles colonnes afficher.
- FROM indique dans quelle table on va chercher les informations.
- WHERE indique de n'afficher que les données respectant les conditions.
  - La rue doit être Northwestern Dr.
  - Le prénom de la personne doit être Annabel.
    - LIKE sert à vérifier qu'un texte soit sous une certaine forme, ici, Annabel puis un nombre indéterminé de charactères.

Resultat :

|id|name|address_number|address_street_name|
|---|---|---|---|
|16371|Annabel Miller|103|Franklin Ave|

Nous avons maintenant l'identité des deux témoins, on va aller chercher le résultat de leurs interrogatoires.

```sql
SELECT transcript FROM interview where person_id = 14887 OR person_id = 16371
```

- SELECT sert à sélectionner quelles colonnes afficher.
- FROM indique dans quelle table on va chercher les informations.
- WHERE indique de n'afficher que les données respectant les conditions.
  - L'id de la personne doit être soit 14887 soit 16371.

Resultat :
>I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag. The membership number on the bag started with "48Z". Only gold members have those bags. The man got into a car with a plate that included "H42W".
>
>I saw the murder happen, and I recognized the killer from my gym when I was working out last week on January the 9th.

On a beaucoup d'information sur le meurtier :

1. Il est inscrit à la salle de sport Get Fit Now Gym.
2. Son identifiant de membre commence par 48Z.
3. Il est membre or.
4. Sa plaque d'immatriculation contient H42W
5. Il était à sa salle de sport le 9 janvier.

```sql
SELECT get_fit_now_member.id, get_fit_now_member.person_id, get_fit_now_member.name 
FROM get_fit_now_member 
INNER JOIN get_fit_now_check_in  ON get_fit_now_member.id=get_fit_now_check_in.membership_id 
WHERE get_fit_now_member.membership_status = 'gold' 
    AND get_fit_now_member.id LIKE '48Z%' 
    AND get_fit_now_check_in.check_in_date = 20180109;
```

- SELECT sert à sélectionner quelles colonnes afficher.
  - On indique les colonnes sous la forme *nom_de_la_table*.*nom_de_la_colonne* car il pourrait y avoir des conflits à cause de la jointure.
- FROM indique dans quelle table on va chercher les informations.
- JOIN sert à créer une jointure, faire correspondre les lignes de deux tables en se basant sur une colonne commune.
  - INNER permet de n'obtenir en résultat que les lignes ayant des correspondances entre les deux tables.
- WHERE indique de n'afficher que les données respectant les conditions.
  - Le statut de l'abonnement est or.
  - L'id de membre doit commencer par 48Z
    - LIKE sert à vérifier qu'un texte soit sous une certaine forme, ici, 48Z puis un nombre indéterminé de charactères.
  - La date de check-in doit être le 09/01/2018.

Resultat :

|id|person_id|name|
|---|---|---|
|48Z7A|28819|Joe Germuska|
|48Z55|67318|Jeremy Bowers|

On trouve donc deux personnes correspondant au profil du tueur, mais on a encore un indice pour trouver lequel des deux est le coupable.

```sql
SELECT name , license_id FROM person WHERE name = 'Joe Germuska' OR name = 'Jeremy Bowers';
```

- SELECT sert à sélectionner quelles colonnes afficher.
- FROM indique dans quelle table on va chercher les informations.
- WHERE indique de n'afficher que les données respectant les conditions.
  - Le nom de la personne doit être soit Joe Germuska soit Jeremy Bowers.

Resultat :

|name|license_id|
|---|---|
|Joe Germuska|173289|
|Jeremy Bowers|423327|

On a l'id de leurs permit de leurs permis de conduire, on peut l'utiliser pour trouver la plaque d'immatriculation de leurs voitures.

```sql
SELECT id, plate_number FROM drivers_license WHERE (id = 173289 OR id = 423327) AND plate_number LIKE '%H42W%';
```

- SELECT sert à sélectionner quelles colonnes afficher.
- FROM indique dans quelle table on va chercher les informations.
- WHERE indique de n'afficher que les données respectant les conditions.
  - La plaque contiens H42W.
    - LIKE sert à vérifier qu'un texte soit sous une certaine forme, ici, H42W entre n'importe quelles charactères.

Resultat :

|id|plate_number|
|---|---|
|423327|0H42W2|

On peut réduire ces requètes en une :

```sql
SELECT 
    person.name,
    get_fit_now_member.id, 
    get_fit_now_member.membership_status,
    drivers_license.plate_number,
    get_fit_now_check_in.check_in_date
FROM get_fit_now_member 
INNER JOIN get_fit_now_check_in 
    ON get_fit_now_member.id = get_fit_now_check_in.membership_id 
INNER JOIN person 
    ON get_fit_now_member.person_id = person.id
INNER JOIN drivers_license 
    ON person.license_id = drivers_license.id
WHERE get_fit_now_member.membership_status = 'gold' 
    AND get_fit_now_member.id LIKE '48Z%' 
    AND get_fit_now_check_in.check_in_date = 20180109
    AND drivers_license.plate_number LIKE '%H42W%';
```

- SELECT sert à sélectionner quelles colonnes afficher.
  - On indique les colonnes sous la forme *nom_de_la_table*.*nom_de_la_colonne* car il pourrait y avoir des conflits à cause de la jointure.
- FROM indique dans quelle table on va chercher les informations.
- JOIN sert à créer une jointure, faire correspondre les lignes de deux tables en se basant sur une colonne commune.
  - INNER permet de n'obtenir en résultat que les lignes ayant des correspondances entre les deux tables.
- WHERE indique de n'afficher que les données respectant les conditions.
  - Le statut de l'abonnement est or.
  - L'id de membre doit commencer par 48Z
    - LIKE sert à vérifier qu'un texte soit sous une certaine forme, ici, 48Z puis un nombre indéterminé de charactères.
  - La date de check-in doit être le 09/01/2018.
  - La plaque contiens H42W.
    - LIKE sert à vérifier qu'un texte soit sous une certaine forme, ici, H42W entre n'importe quelles charactères.

Résultat :

|name|id|membership_status|plate_number|check_in_date|
|---|---|---|---|---|
|Jeremy Bowers|48Z55|gold|0H42W2|20180109|

On trouve donc que le meurtrier est **Jeremy Bowers**.
On peut aller consulter son interrogatoire.

```sql
SELECT transcript FROM interview WHERE person_id = 67318;
```

- SELECT sert à sélectionner quelles colonnes afficher.
- FROM indique dans quelle table on va chercher les informations.
- WHERE indique de n'afficher que les données respectant les conditions.
  - L'id de la personne doit être 67318.

Resultat :
>I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5" (65") or 5'7" (67"). She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017.

On va trouver la personne ayant commanditer le meurtre en se basant sur ces informations.

```sql
SELECT person.name
FROM person
INNER JOIN drivers_license ON person.license_id = drivers_license.id
WHERE drivers_license.gender = 'female'
  AND drivers_license.height BETWEEN 65 AND 67
  AND drivers_license.hair_color = 'red'
  AND drivers_license.car_make = 'Tesla'
  AND drivers_license.car_model = 'Model S'
  AND person.id IN (
    SELECT person_id
    FROM facebook_event_checkin
    WHERE event_name = 'SQL Symphony Concert'
      AND date BETWEEN 20171201 AND 20171231
    GROUP BY person_id
    HAVING COUNT(*) = 3
  );
```

- SELECT sert à sélectionner quelles colonnes afficher.
  - On indique les colonnes sous la forme *nom_de_la_table*.*nom_de_la_colonne* car il pourrait y avoir des conflits à cause de la jointure.
- FROM indique dans quelle table on va chercher les informations.
- JOIN sert à créer une jointure, faire correspondre les lignes de deux tables en se basant sur une colonne commune.
  - INNER permet de n'obtenir en résultat que les lignes ayant des correspondances entre les deux tables.
- WHERE indique de n'afficher que les données respectant les conditions.
  - Le genre de la personne est female
  - La taille de la personne est comprise entre 65 et 67 pouces.
    - BETWEEN sert à vérifier qu'un élément est compris entre deux valeurs.
  - La couleur de cheveux de la personne est red.
  - La marque de la voiture est Tesla.
  - Le modele de la voiture est Model S.
  - L'id de la personne apparait trois fois dans des lignes de la table facebook_event_checkin où le nom de l'évenement est SQL Symphony Concert et la date est comprise entre le 01/12/2017 et 31/12/2017.
    - GROUP BY permet de grouper les résultat ayant la même valeur dans une colonne.
    - HAVING sert à filtrer les groupes de GROUP BY, équivalent à un WHERE
    - COUNT permet de compter le nombre de ligne.

La commanditaire est **Miranda Priestly**.
