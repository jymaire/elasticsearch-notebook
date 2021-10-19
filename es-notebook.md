# Liste des sujets



**Data Management**

- [Define an index that satisfies a given set of requirements](#define-an-index-that-satisfies-a-given-set-of-requirements)
- [Use  the Data Visualizer to upload a text file into Elasticsearch](#data-visualizer)
- [Define and use an index template for a given pattern that satisfies a given set of requirements](#index-template)
- [Define and use a dynamic template that satisfies a given set of requirements](#dynamic-template)
- [Define an Index Lifecycle Management policy for a time-series index](index-lifecycle-management)
- [Define an index template that creates a new data stream](data-stream)

**Searching Data**

- <u>Write and execute a search query for terms and/or phrases in one or more fields of an index</u> : ok, tp movies
- <u>Write and execute a search query that is a Boolean combination of multiple queries and filters</u> : ok, tp movies
-  [Write an asynchronous search](asynchronous-search)
- <u>Write and execute metric and bucket aggregations</u> : ok tp movies
- <u>Write and execute aggregations that contain sub-aggregations</u> : ok tp movies
- [Write and execute a query that searches across multiple clusters](remote-cluster)

**Developing Search Applications**

- Highlight the search terms in the response of a query
- Sort the results of a query by a given set of requirements
- Implement pagination of the results of a search query
- Define and use index aliases
- Define and use a search template

**Data Processing**

- Define a mapping that satisfies a given set of requirements
- Define and use a custom analyzer that satisfies a given set of requirements
- Define and use multi-fields with different data types and/or analyzers
- Use the Reindex API and Update By Query API to reindex and/or update documents
- Define and use an ingest pipeline that satisfies a given set of requirements, including the use of Painless to modify documents
- Configure an index so that it properly maintains the relationships of nested arrays of objects

**Cluster Management**

- Diagnose shard issues and repair a cluster's health
- Backup and restore a cluster and/or specific indices
-  Configure a snapshot to be searchable
- Configure a cluster for cross-cluster search
- Implement cross-cluster replication
- Define role-based access control using Elasticsearch Security



## Data management

### Define an index that satisfies a given set of requirements

Afin de créer un index, il faut utiliser l'instruction :

```console
PUT /my-index-000001
```

Les settings et le mapping peuvent se rajouter dans le body

```console
PUT /test
{
  "settings": {
    "number_of_shards": 1
  },
  "mappings": {
    "properties": {
      "field1": { "type": "text" }
    }
  }
}
```

### Data visualizer

Sur la page d'accueil de Kibana, il y a un encart "Upload a file" dans la partie "Ingest your data". Il suffit de suivre les étapes pour ajouter un fichier puis définir l'index et ses propriétés.

### Index template

Un template permet de pré définir une configuration qui sera appliquée au moment de la création d'un index. Si on a besoin d'avoir des morceaux de templates communs à différents index, on peut définir des composants de templates (définition du mapping, des settings et des alias). Entre une configuration venant d'un composant et une configuration venant de l'appel Rest créant l'index, c'est ce dernier qui sera prioritaire en cas de recoupement.

Pour définir un template d'index, on peut soit définir des composants de templates via :

```console
PUT _component_template/template_1
{ ... }
```
Soit mettre la configuration directement. On pourra ensuite choisir d'utiliser ou non des composants de template au moment de le définir le template via :
```console
PUT _index_template/template_1
{ ... }
```
Parmi les options, il y a le mapping, le pattern des index auquel le template va s'appliquer, les settings de l'index... Pour choisir quel composant va s'appliquer en cas de recoupements des configurations, il faut définir une priorité différente pour chaque composant. (attention aussi aux templates built-in ES : `logs-*-*` , `metrics-*-*` et `synthetics-*-*`)

https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-component-template.html
https://www.elastic.co/guide/en/elasticsearch/reference/current/index-templates.html


### Dynamic template

Pour activer les templates dynamiques, il faut passer le paramètre "dynamic" à "true" ou à "runtime". Cela permet de définir un mapping pour des champs en fonction, par exemple, d'un format de nom d'un champ.  

https://www.elastic.co/guide/en/elasticsearch/reference/7.14/dynamic-templates.html



### Index Lifecycle Management

Cela permet de définir certaines actions sur les indexes de manière automatique. Il y a quatre phases pour un index : hot, warm, cold, frozen, delete. (hot -> actif, warm -> plus de mise à jour mais des requêtes, cold ->quelques requêtes, avec un temps de réponse qui sera un peu plus long, frozen -> de rare requêtes qui auront un temps de réponse très long, delete -> prêt à être supprimé)

Le but est de pouvoir rationaliser l'infrastructure, en déplaçant les données sur différents types de machines en fonction de leur étape dans le cycle de vie.

### Data stream

Un data stream permet de stocker des données temporel en ajout uniquement. Il va créer des indices cachés. Notre point d'entrée pour requêter et indexer les données sera une sorte d'alias.

Un DS est composé d'un ou plusieurs indices cachés, auto générés.

Un DS a besoin d'un index template  "matching" qui contiennent le mapping et les settings des indices cachés.

Il faut un index template et un champ @timestamp de type date ou date_nanos afin de pouvoir créer des data streams. Un data stream va permettre de créer automatiquement des indices en fonction de la date du document.

Un index template peut être réutilisé pour plusieurs DS.

La lecture se fait dans tous les indices cachés.

L'écriture se fait uniquement dans l'indice le plus récent.

Pour créer un nouvel index d'écriture, il faut faire un rollover (possiblement géré par un ILM, manuellement sinon)

Quand on crée un component template, il faut inclure u**n champ @timestamp de type date et une lifecycle policy dans l'index setting**.

Puis il faut créer un index template. Il faut que **le pattern des indices corresponde avec le nom du data stream**. Également **cocher l'activation des DS, ajouter les components templates avec le mapping et le setting d'index et une priorité supérieur à 200** pour éviter une collision avec les templates pré définis.

Pour ajouter des documents, il faut passer par un POST et non par un PUT (ou un bulk avec l'ordre "create")

## Searching data

https://www.elastic.co/guide/en/elasticsearch/reference/current/term-level-queries.html

https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query-phrase.html

https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-bool-query.html

### Asynchronous search

Cela se passe comme une recherche classique, seul le endpoint change (de \_search vers \_async_search). Un ID est retourné, celui ci permet de requêter les résultats plus tard via l'API `GET /_async_search/ID`.

### Remote cluster

Il faut commencer par définir l'adresse des clusters distants dans les paramètres du cluster (point d'entrées `_cluster/settings`) puis après il faut préfixer le nom de l'index par `nom-du-cluster:` pour spécifier le cluster sur lequel on effectue la requête. Bien pensé à activer la licence (trial) afin d'avoir le droit de le faire.



https://www.elastic.co/guide/en/elasticsearch/reference/7.15//modules-cross-cluster-search.html

### To do si pas traité dans les autres sujets :
- mappings
- settings
- alias

