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