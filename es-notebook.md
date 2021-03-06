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

Afin de cr??er un index, il faut utiliser l'instruction :

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

Sur la page d'accueil de Kibana, il y a un encart "Upload a file" dans la partie "Ingest your data". Il suffit de suivre les ??tapes pour ajouter un fichier puis d??finir l'index et ses propri??t??s.

### Index template

Un template permet de pr?? d??finir une configuration qui sera appliqu??e au moment de la cr??ation d'un index. Si on a besoin d'avoir des morceaux de templates communs ?? diff??rents index, on peut d??finir des composants de templates (d??finition du mapping, des settings et des alias). Entre une configuration venant d'un composant et une configuration venant de l'appel Rest cr??ant l'index, c'est ce dernier qui sera prioritaire en cas de recoupement.

Pour d??finir un template d'index, on peut soit d??finir des composants de templates via :

```console
PUT _component_template/template_1
{ ... }
```
Soit mettre la configuration directement. On pourra ensuite choisir d'utiliser ou non des composants de template au moment de le d??finir le template via :
```console
PUT _index_template/template_1
{ ... }
```
Parmi les options, il y a le mapping, le pattern des index auquel le template va s'appliquer, les settings de l'index... Pour choisir quel composant va s'appliquer en cas de recoupements des configurations, il faut d??finir une priorit?? diff??rente pour chaque composant. (attention aussi aux templates built-in ES : `logs-*-*` , `metrics-*-*` et `synthetics-*-*`)

https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-component-template.html
https://www.elastic.co/guide/en/elasticsearch/reference/current/index-templates.html


### Dynamic template

Pour activer les templates dynamiques, il faut passer le param??tre "dynamic" ?? "true" ou ?? "runtime". Cela permet de d??finir un mapping pour des champs en fonction, par exemple, d'un format de nom d'un champ.  

https://www.elastic.co/guide/en/elasticsearch/reference/7.14/dynamic-templates.html



### Index Lifecycle Management

Cela permet de d??finir certaines actions sur les indexes de mani??re automatique. Il y a quatre phases pour un index : hot, warm, cold, frozen, delete. (hot -> actif, warm -> plus de mise ?? jour mais des requ??tes, cold ->quelques requ??tes, avec un temps de r??ponse qui sera un peu plus long, frozen -> de rare requ??tes qui auront un temps de r??ponse tr??s long, delete -> pr??t ?? ??tre supprim??)

Le but est de pouvoir rationaliser l'infrastructure, en d??pla??ant les donn??es sur diff??rents types de machines en fonction de leur ??tape dans le cycle de vie.

### Data stream

Un data stream permet de stocker des donn??es temporel en ajout uniquement. Il va cr??er des indices cach??s. Notre point d'entr??e pour requ??ter et indexer les donn??es sera une sorte d'alias.

Un DS est compos?? d'un ou plusieurs indices cach??s, auto g??n??r??s.

Un DS a besoin d'un index template  "matching" qui contiennent le mapping et les settings des indices cach??s.

Il faut un index template et un champ @timestamp de type date ou date_nanos afin de pouvoir cr??er des data streams. Un data stream va permettre de cr??er automatiquement des indices en fonction de la date du document.

Un index template peut ??tre r??utilis?? pour plusieurs DS.

La lecture se fait dans tous les indices cach??s.

L'??criture se fait uniquement dans l'indice le plus r??cent.

Pour cr??er un nouvel index d'??criture, il faut faire un rollover (possiblement g??r?? par un ILM, manuellement sinon)

Quand on cr??e un component template, il faut inclure u**n champ @timestamp de type date et une lifecycle policy dans l'index setting**.

Puis il faut cr??er un index template. Il faut que **le pattern des indices corresponde avec le nom du data stream**. ??galement **cocher l'activation des DS, ajouter les components templates avec le mapping et le setting d'index et une priorit?? sup??rieur ?? 200** pour ??viter une collision avec les templates pr?? d??finis.

Pour ajouter des documents, il faut passer par un POST et non par un PUT (ou un bulk avec l'ordre "create")

## Searching data

https://www.elastic.co/guide/en/elasticsearch/reference/current/term-level-queries.html

https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query-phrase.html

https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-bool-query.html

### Asynchronous search

Cela se passe comme une recherche classique, seul le endpoint change (de \_search vers \_async_search). Un ID est retourn??, celui ci permet de requ??ter les r??sultats plus tard via l'API `GET /_async_search/ID`.

### Remote cluster

Il faut commencer par d??finir l'adresse des clusters distants dans les param??tres du cluster (point d'entr??es `_cluster/settings`) puis apr??s il faut pr??fixer le nom de l'index par `nom-du-cluster:` pour sp??cifier le cluster sur lequel on effectue la requ??te. Bien pens?? ?? activer la licence (trial) afin d'avoir le droit de le faire.



https://www.elastic.co/guide/en/elasticsearch/reference/7.15//modules-cross-cluster-search.html

### To do si pas trait?? dans les autres sujets :
- mappings
- settings
- alias

