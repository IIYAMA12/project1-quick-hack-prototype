# Amsterdam - Street information
With this tool, you find your own street photo's with meta-data just by clicking on a loaded/(white) street.

## Bugs
- The longest streets are not supported yet, because of some problems with the geo locations.

## Controls
- To load a new area with streets: `Click double on the map`
- To get information about a street: `Click on a street`

## Future features
- Add user-interface for filtering at streets size. (Functionality is already created)


[Website link](https://iiyama12.github.io/project1-quick-hack-prototype/map)


## SPARQL queries

### Retrieving streets

```JS
const sparqlquery = `
    PREFIX hg: <http://rdf.histograph.io/>
    PREFIX geo: <http://www.opengis.net/ont/geosparql#>
    PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>

    SELECT DISTINCT ?street ?size ?streetgeom ?streetName ?hasEarliestBeginTimeStamp  WHERE {
        ?street rdf:type hg:Street .
        ?street geo:hasGeometry/geo:asWKT ?wkt .
        ?street <http://www.w3.org/2000/01/rdf-schema#label> ?streetName .

        BIND (bif:st_geomfromtext(?wkt) as ?streetgeom) .
        BIND (bif:st_geomfromtext("POINT(` + lng + ` ` + lat + `)") as ?point) .

        FILTER (!REGEX(?wkt, 'Array')) .

        BIND(bif:st_get_bounding_box(?streetgeom) as ?boundingBox ) .
        BIND(((bif:ST_XMax (?boundingBox) - bif:ST_XMin(?boundingBox)) + (bif:ST_YMax (?boundingBox) - bif:ST_YMin(?boundingBox)))  as ?size) .

        OPTIONAL {
            ?street <http://semanticweb.cs.vu.nl/2009/11/sem/hasEarliestBeginTimeStamp> ?hasEarliestBeginTimeStamp .
        } .

        FILTER (bif:st_may_intersect (?point, ?streetgeom, 0.006)) .
    }
    ORDER BY (?size)
    LIMIT 500
`;
```
#### Data
- The street (URI)
- Street size
- Street geo information
- Street name
- (OPTIONAL)
    - The begin date of street

### Retrieving photo's

```JS
const sparqlquery = `
    PREFIX dct: <http://purl.org/dc/terms/>
    PREFIX foaf: <http://xmlns.com/foaf/0.1/>

    SELECT ?item ?img ?creator ?subject ?startYear WHERE {
        ?item dct:spatial <` + JSON.parse(feature.properties.uri).value + `>  .
        ?item foaf:depiction ?img .

        optional {
            ?item <http://purl.org/dc/elements/1.1/creator> ?creator .
        }
        optional {
            ?item <http://purl.org/dc/elements/1.1/description> ?description .
        }
        optional {
            ?item <http://purl.org/dc/elements/1.1/subject> ?subject .
        }
        optional {
            ?item <http://semanticweb.cs.vu.nl/2009/11/sem/hasBeginTimeStamp> ?startYear .
        }
    }
    ORDER BY DESC(?startYear)
    LIMIT 100
`;
```

#### Data
- Image (URI)
- Image depiction
- (OPTIONAL)
    - Creator
    - Description
    - Subject
    - Start year


## - - TODO - -
