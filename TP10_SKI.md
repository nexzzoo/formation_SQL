# 1 Liste des clients (toutes les informations) dont le nom commence par un D  
``` sql
SELECT noCli,nom,prenom,adresse,cpo,ville 
FROM clients 
WHERE nom LIKE 'D%';
```

# 2 Nom et prénom de tous les clients
```sql
SELECT nom,prenom
FROM clients;
```

# 3 Liste des fiches (n°, état) pour les clients (nom, prénom) qui habitent en Loire Atlantique (44)
``` sql
SELECT noFic,etat,nom,prenom    
FROM clients
inner JOIN fiches ON clients.noCli=fiches.noCli
where cpo like '44%';
```

# 4 Détail de la fiche n°1002 
``` sql
SELECT 
    fiches.noFic,clients.nom,clients.prenom,articles.refArt,articles.designation,lignesFic.depart,lignesFic.retour,tarifs.prixJour,(DATEDIFF(COALESCE(lignesFic.retour, NOW()), lignesFic.depart )+1)  * tarifs.prixJour  AS montant 
FROM fiches
inner JOIN clients ON fiches.noCli = clients.noCli
inner JOIN lignesFic ON fiches.noFic = lignesFic.noFic
inner JOIN articles ON lignesFic.refArt = articles.refArt
inner JOIN grilleTarifs ON articles.codeGam = grilleTarifs.codeGam AND articles.codeCate = grilleTarifs.codeCate
inner JOIN tarifs ON grilleTarifs.codeTarif = tarifs.codeTarif
WHERE fiches.noFic = 1002;
```

# 5 Prix journalier moyen de location par gamme
``` sql
Select  
CASE 
    WHEN Articles.codeGam = "EG" THEN "Entrée de gamme" 
    WHEN Articles.codeGam = "MG" THEN "Moyenne gamme"
    WHEN Articles.codeGam = "HG" THEN "Haute gamme"
    WHEN Articles.codeGam = "PR" THEN "Materiel professionnel"
    ELSE  'autre'
END as gamme,AVG(tarifs.prixJour) as tarif_journalier_moyen
FROM tarifs
JOIN grilleTarifs ON tarifs.codeTarif = grilleTarifs.codeTarif
JOIN articles ON grilleTarifs.codeGam = articles.codeGam
GROUP BY articles.codeGam;
```

# 6 Détail de la fiche n°1002 avec le total
``` sql
select 
    noFic,nom,prenom,refArt,designation,depart,retour,prixJour, montant,sum(montant) OVER() as Total
from (
SELECT 
    fiches.noFic,clients.nom,clients.prenom,articles.refArt,articles.designation,lignesFic.depart,lignesFic.retour,tarifs.prixJour,(DATEDIFF(COALESCE(lignesFic.retour, NOW()), lignesFic.depart )+1)  * tarifs.prixJour  AS montant 
FROM fiches
inner JOIN clients ON fiches.noCli = clients.noCli
inner JOIN lignesFic ON fiches.noFic = lignesFic.noFic
inner JOIN articles ON lignesFic.refArt = articles.refArt
inner JOIN grilleTarifs ON articles.codeGam = grilleTarifs.codeGam AND articles.codeCate = grilleTarifs.codeCate
inner JOIN tarifs ON grilleTarifs.codeTarif = tarifs.codeTarif
WHERE fiches.noFic = 1002
)as t
```

#7 Grille des tarifs
``` sql
select categories.libelle as libelle, 
CASE 
    WHEN gammes.codeGam = "EG" THEN "Entrée de gamme" 
    WHEN gammes.codeGam = "MG" THEN "Moyenne gamme"
    WHEN gammes.codeGam = "HG" THEN "Haute gamme"
    WHEN gammes.codeGam = "PR" THEN "Materiel professionnel"
    ELSE  'autre'
END as gamme, tarifs.libelle as tarif, prixjour
from gammes
inner join grilleTarifs on gammes.codegam = grilleTarifs.codegam
inner join tarifs on grilleTarifs.codetarif = tarifs.codetarif
inner join categories on grilleTarifs.codecate = categories.codecate
order by gammes.codegam;
```

# 8 liste des locations de la catégorie SURF
``` sql
SELECT articles.refArt, articles.designation, COUNT(articles.refArt) AS nbArticles
FROM lignesFic
JOIN fiches ON lignesFic.noFic = fiches.noFic
JOIN clients ON fiches.noCli = clients.noCli
JOIN articles ON lignesFic.refArt = articles.refArt
JOIN grilleTarifs ON articles.codeGam = grilleTarifs.codeGam AND articles.codeCate = grilleTarifs.codeCate
WHERE grilleTarifs.codeCate = 'SURF'
GROUP BY articles.refArt, articles.designation;
```

# 9 Calcul du nombre moyen d’articles loués par fiche de location
``` sql
SELECT AVG(total) as nb_lignes_moyen_par_fiche
FROM(SELECT COUNT(noFic)as total
FROM lignesFic
GROUP BY noFic) as total
```


# 10 Calcul du nombre de fiches de location établies pour les catégories de location Ski alpin, Surf et Patinette
``` sql
SELECT categories.libelle as categorie, COUNT(fiches.noFic) as nombre_de_location
FROM fiches
JOIN lignesFic ON fiches.noFic = lignesFic.noFic
JOIN articles ON lignesFic.refArt = articles.refArt
JOIN grilleTarifs ON articles.codeGam = grilleTarifs.codeGam AND articles.codeCate = grilleTarifs.codeCate
JOIN categories ON grilleTarifs.codeCate = categories.codeCate
WHERE categories.libelle = 'Ski alpin' OR categories.libelle = 'Surf' OR categories.libelle = 'Patinette'
GROUP BY categories.libelle ;
```

# 11 Calcul du montant moyen des fiches de location
```sql
SELECT AVG(total) as montant_moyen_une_fiche_de_location
FROM(SELECT (DATEDIFF(COALESCE(lignesFic.retour, NOW()), lignesFic.depart)+1) * tarifs.prixJour AS total
FROM lignesfic
INNER JOIN articles on articles.refart = lignesFic.refArt
INNER JOIN categories on categories.codeCate = articles.codeCate
INNER Join grilleTarifs on grilleTarifs.codeCate = categories.codeCate
INNER JOIN tarifs on tarifs.codeTarif = grilleTarifs.codeTarif
) as total
```
