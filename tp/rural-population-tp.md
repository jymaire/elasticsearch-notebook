TP à partir du jeu de données "urban-and-rural-population.csv" disponible dans le répertoire "data"



### Ajout des données

- Aller sur la page d'accueil de Kibana
- Aller dans la partie "Upload a file"
- Télécharger le fichier CSV puis laisser les options par défaut. Nommer l'index "rural-population"
- Réaliser l'import

Résultat attendu : un index "rural-population" a été créé. Il contient 16 000 documents.

`GET rural-population/_count`



### Recherche de données

Rechercher le document correspondant à la population Afghane de 1960 :

`GET rural-population/_search
{
  "fields": [
    "Urban population"
  ], 
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "Code": "AFG"
          }},{
          "match": {
            "Year": "1960"
          }
        }
      ]
    }
  }
}`



#### Ajout d'un runtime field

On va rajouter un champ qui soit la somme de la population urbaine et de la population rurale.

Deux manières de faire : soit dans la requête, soit dans le mapping.

##### à la recherche

`GET rural-population/_search`
`{`
  `"size": 0,` 
  `"runtime_mappings": {`
    `"total-pop": {`
      `"type": "long",`
      `"script": {`
        `"source": "emit(doc['Urban population'].value + doc['Rural population'].value)"`
      `}`
    `}`
  `},`
  `"aggs": {`
    `"total-pop": {`
      `"terms": {`
        `"field": "total-pop",`
        `"size": 10`
      `}`
    `}`
  `}`
`}`

##### dans le mapping

On commence par récupérer le mapping existant afin de pouvoir comparer 

`GET rural-population/_mapping`

On met à jour le mapping

`PUT rural-population/_mapping`
`{`
  `"properties": {`
    `"total-pop": {`
      `"type": "long",`
      `"script": {`
        `"source": "emit(doc['Urban population'].value + doc['Rural population'].value)"`
      `}`
    `}`
  `}`
`}`

On peut vérifier qu'on voit bien les données

`GET rural-population/_search`
`{`
  `"fields": [ "total-pop" ]`
`}`