Ajouter les données dans ES (index uk-pnb)

### Création d'une gestion de cycle de vie d'index (index lifecycle management) et d'un data stream

#### creation policy

aller dans Stack management / index  lifecycle policies

Ajouter une nouvelle policy

définir les différentes étapes

sauvegarder

Ajouter la policy à l'index

`PUT uk-pnb/_settings`
`{`
  `"index": {`
    `"lifecycle" : {`
      `"name": "uk-policy"`
    `}`
  `}`
`}`

#### Ajout champ timestamp

Nécessaire pour créer un data stream :

`PUT rural-population/_mapping`
`{`
  `"properties" : {`
    `"@timestamp": {`
      `"type" : "date"`
    `}`
  `}`
`}`

#### Création d'un component template

Aller dans stack management / index management / component templates puis create component template



```
PUT _component_template/uk-component
{
  "template": {
    "mappings": {
      "properties": {
        "@timestamp": {
          "type": "date"
        	}
      	}
       }
	}
}
```

Puis on va mettre à jour l'index template créé juste avant (bouton éditer sur l'index template puis suivre les flèches)





