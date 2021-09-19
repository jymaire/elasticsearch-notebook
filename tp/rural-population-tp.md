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



