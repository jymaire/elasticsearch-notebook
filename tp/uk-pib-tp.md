Ajouter les données dans ES (index uk-pnb)

### Création d'une gestion de cycle de vie d'index (index lifecycle management)

#### creation policy

aller dans Stack management / index  lifecycle policies

Ajouter une nouvelle policy

définir les différentes étapes

sauvegarder

Ajouter la policy à l'index

PUT uk-pnb/_settings
{
  "index": {
    "lifecycle" : {
      "name": "uk-policy"
    }
  }
}





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