Ajouter les données dans ES (index movies). Changer la plupart des keyword en text pour faire des recherches textuelles dessus



## Quelques recherches

### Les films de 1990 - avec score

`GET /movies/_search`
`{`
  `"query": {`
    `"bool": {`
      `"must": [`
        `{"term": {`
          `"Year": {`
            `"value": "1990"`
          `}`
        `}}`
      `]      
    }`
  `}`
`}`

### Les films de 1990 - sans score

`GET /movies/_search`
`{`
  `"query": {`
    `"bool": {`
      `"filter": [`
        `{"term": {`
          `"Year": "1990"`
        `}}`
      `]`
    `}`
  `}`
`}`



### Les comédies de 1990

`GET /movies/_search`
`{`
  `"query": {`
    `"bool": {`
      `"must": [`
        `{"term": {`
          `"Year": {`
            `"value": "1990"`
          `}`
        `}},`
        `{"term": {`
          `"Subject": {`
            `"value": "comedy"`
          `}}`
        `}`
      `]      
    }`
  `}`
`}`

### Les films de Mr Allen

`GET /movies/_search`
`{`
  `"query": {`
    `"match": {`
      `"Director": "allen"`
    `}`
  `}`
`}`

### Les films avec un director, actrice ou titre contenant "alice"

`GET /movies/_search`
`{`
  `"query": {`
    `"multi_match": {`
      `"query": "alice",`
      `"fields": ["Director", "Actress", "Title"]`
    `}`
  `}`
`}`

### Les films de 1982 et 1990

`GET /movies/_search`
`{`
`"query": {`
  `"terms": {`
    `"Year": [`
      `"1982",`
      `"1990"`
    `]`
  `}`
`}`
`}`

### La longueur moyenne des films de Nicolas Cage

`GET /movies/_search`
`{`
  `"size": 0,`
  `"query": {`
    `"bool": {`
      `"filter": [`
        `{`
          `"match": {`
            `"Actor": "Cage, Nicolas"`
          `}`
        `}`
      `]`
    `}`
  `},`
  `"aggs": {`
    `"moyenne": {`
      `"avg": {`
        `"field": "Length"`
      `}`
    `}`
  `}`
`}`

### Répartition de la popularité des films

`GET /movies/_search`
`{`
  `"size": 0,`
  `"aggs": {`
    `"decennies": {`
      `"range": {`
        `"field": "Popularity",`
        `"ranges": [`
          `{"to" : "33"},`
          `{"from" : "33", "to": "66"},`
          `{"from": "66"}`
        `]`
      `}`
    `}`
  `}`
`}`

### La longueur moyenne des films de Nicolas Cage en fonction du type de film

`GET /movies/_search`
`{`
  `"runtime_mappings": {`
    `"Subject": {`
      `"type": "keyword"`
    `}`
  `},` 
  `"size": 0,`
  `"query": {`
    `"bool": {`
      `"filter": [`
        `{`
          `"match": {`
            `"Actor": "Cage, Nicolas"`
          `}`
        `}`
      `]`
    `}`
  `},`
  `"aggs": {`
    `"type": {`
      `"terms": {`
        `"field": "Subject",`
        `"size": 10`
      `},`
      `"aggs": {`
        `"moyenne": {`
          `"avg": {`
            `"field": "Length"`
          `}`
        `}`
      `}`
    `}`
  `}`
`}`