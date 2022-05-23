# Aggregate 

Provarem les diferents comandes (pipelines)  a partir de la inserció d'aquests registres de prova: 

> db.articles.insertMany([
{ "author" : "dave", "score" : 80, "views" : 100 },
{ "author" : "dave", "score" : 85, "views" : 521 },
{ "author" : "ahn", "score" : 60, "views" : 1000 },
{ "author" : "li", "score" : 55, "views" : 5000 },
{ "author" : "annT", "score" : 60, "views" : 50 },
{ "author" : "li", "score" : 94, "views" : 999 },
{ "author" : "ty", "score" : 95, "views" : 1000}, { "author" : "dave", "score" : 70, "views" : 1100 },
{ "author" : "dave", "score" : 95, "views" : 1521 },
{ "author" : "ahn", "score" : 60, "views" :2000 },
{ "author" : "li", "score" : 35, "views" : 6000 },
{ "author" : "annT", "score" : 65, "views" : 1050 },
{ "author" : "li", "score" : 96, "views" : 1999 },
{ "author" : "ty", "score" : 89, "views" : 2000 }]);

La instrucció aggregate, serveix per concatenar diferents pipelines, que van modificant les dades a partir del pipeline anterior..

**Exemple **
![agreg1](agreg1.png "agreg1")

### Concatenació pipelines

![pipelines](pipelines.png "pipelines")


#### Diferents Pipelines 
El millor lloc i més actualitzat sempre serà la [pàgina oficial de mongodb](https://www.mongodb.com/docs/v4.2/reference/operator/aggregation-pipeline/#aggregation-pipeline-operator-reference "Link documentació")

##### MATCH

És l'encarregat de fer la la funció SQL **where** i funciona molt similar a la instrucció **find** de mongodb

>db.articles.aggregate(
    { $match : { author : "dave" } } ]
);

Aquesta instrucció selecciona tots els registres al qual el seu autor és David.

Tots els operadors vistos amb el find funcionen amb el match. 

*Exemple: *

> db.articles.aggregate( [
  { $match: { $or: [ { score: { $gt: 70, $lt: 90 } }, { views: { $gte: 1000 } } ] } }
] );

En aquest cas, seleccionem els articles que l'han puntuat(*score*) entre 70 i 90, o els articles que han rebut més de 1000 visites(*views*).

##### GROUP
[link documentació](https://www.mongodb.com/docs/v4.2/reference/operator/aggregation/group/#pipe._S_group "link")
Seria la funció similar a la de SQL (*GROUP BY*)

> db.articles.aggregate( [
  { $match: { $or: [ { score: { $gt: 70, $lt: 90 } }, { views: { $gte: 1000 } } ] } },
  { $group: { _id: null, count: { $sum: 1 } } }
] );

Els camps que es fiquen després de _id:, són els camps pel qual agruparà els registres, en aquest cas al ficar **null**, agafa tots els registres de la collection i realitzarà la funció count (contarà quants registres existeixen).

>db.articles2.aggregate([
   { $match: { score: {$gt:35} } },
   { $group: { _id: "$author", total: { $sum: "$views" } } }
])

En aquest cas després d'eliminar tots els registres que tenen puntuació menor de 35, agruparem per autor i contarem quantes visites han rebut cadascun dels diferents autors, que es mostraran en un camp nou anomenat **total**.

###### Funcións aplicables a group.
Totes les funcions que podem utilitzar al pipeline group, es pot consultar [aqui](https://www.mongodb.com/docs/v4.2/reference/operator/aggregation/group/#accumulators-group "aqui"), moltes son similars a funcions existents en sql.

Una de les més interessants i que no existeix en SQL, seria el $addToSet, que agafa tots els valors diferents  d'un camp per una  agrupació determinada i els fica tots en un array.

>db.articles.aggregate(
   [
     {
       $group:
         { _id: "$author",
           puntuacions: { $addToSet: "$score" }
         }
     }
   ]
)

Amb aquesta instrucció agrupem tots els articles per author i en un camp nou anomenat **puntuacions**, crearem un array amb les distintes puntuacions creades. 

![addtoset](addtoset.png "addtoset")

##### PROJECT
Seria l'equivalent al select camp1,camp2,camp3, de sql, amb la diferència, que tots els camps que no estiguin indicats amb 1, desapareixen per les següents etapes.
[documentació aquí](https://www.mongodb.com/docs/v4.2/reference/operator/aggregation/project/#pipe._S_project "documentació aquí")

>db.articles2.aggregate([
   { $match: { score: {$gt:35} } },
   { $project: { _id: 0, "author":1, score:1 } }
])

Aquesta instrucció mostraria els camps author i el score, els altres no els mostraria. Per defecte tots els camps estan marcats a 0 (no es projecten) excepte el _id, que per defecte es troba a 1 i si no volen que es marqui, s'ha de ficar a 0.

##### SORT
Ordena la collection pels camps indicats en aquest pipeline. Els camps marcats amb *1*  s'ordenaran de forma ascendent, i amb *-1* descendent
[documentació aqui](https://www.mongodb.com/docs/v4.2/reference/operator/aggregation/sort/#pipe._S_sort "documentació aqui")


>db.articles.aggregate(
   [
     { $sort : { score : -1, views: 1 } }
   ]
)

Aquest cas ordenaríem, pels articles més valorats, i en cas d'empat ordena per les que tenen més visites primers.


##### SKIP

Salta els 5 primers registres que troba

> db.article.aggregate([
    { $skip : 5 }
]);

##### LIMIT

Mostra els registres indicats al pipeline limit 

> db.article.aggregate([
   { $limit : 5 }
]);


##### UNWIND

Descomposa, si en un registre existeix un camp amb valors arrays , els pots descompondre.

Suposem que tenim aquest registre a articles

>  db.articles.insert(
{ "author" : "jairo", "score" : 10, "views" : 30 , "version":["web","phone","paper"]})

aquest registre mostraria un article, publicat amb 3 versions, via web, via mòbil i versió escrita.

>db.articles.aggregate( [ { $unwind : "$version" } ] )

![imatge](jairo.png "image")

Hi ha més opcions interessants que es poden fer amb el pipeline unwind, que es poden trobar a la [documentació oficial](https://www.mongodb.com/docs/v4.2/reference/operator/aggregation/unwind/#pipe._S_unwind "documentació oficial")

##### OUT
Comanda per agafar el resultat final e insertarlo a una nova collection, indicada en aquest pipeline.
[documentació aquí](https://www.mongodb.com/docs/v4.2/reference/operator/aggregation/out/#pipe._S_out)

Amb SQL, seria com si fessis un SELECT, i el resultat que et dones aquell select, realitzessis un INSERT INTO

> db.articles..aggregate({out:"backup"})

##### MERGE

És una evolució del out, crea nous registres en una nova collection o si existeix els inserta, o fins i tot pots indicar que els actualitzi, en cas que coincideixi cert camp.

>db.articles..aggregate({ $merge: { into: "articles2", on: "_id", whenMatched: "replace", whenNotMatched: "insert" } })

En aquest cas inseriria a una nova collection, si aquesta collection ja existeix, i ja hi és amb aquella id, realitzaria un update, sinó faria un insert.
**Sol funciona a partir de la versió 4.2 de mongodb**


##### SET

Afegeixes nous camps a un registre o a tots els registres especificats.

>db.articles..aggregate( {
     $set: {
        ViewmoreTotal: { $add: [ "$score", "$views" ] }
     }
   })

En aquest cas a tots els registres, els hi afegiria un nou camp, **ViewmoreTotal** , amb la suma dels valors dels camps *views* i *score*
**Sol funciona a partir de la versió 4.2 de mongodb**
