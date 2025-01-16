## Getting started

Very simple query to get the first 5 strains and their genus:

```sparql
PREFIX d3o: <https://purl.dsmz.de/schema/>
SELECT * WHERE {
    ?strain a d3o:Strain .
    ?strain d3o:hasGenus ?genus .
} 
LIMIT 5
```

Let's add more info on a strain, e.g. the label to the previous query. We need a new `PREFIX` for rdfs and we also shorten the query by using `;` to combine the two triples:

```sparql
PREFIX d3o: <https://purl.dsmz.de/schema/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT * WHERE {
    ?strain a d3o:Strain ;
            rdfs:label ?label ;
            d3o:hasGenus ?genus .
}
LIMIT 5
```

We want to combine the genus and species of a strain in one column. We can use `CONCAT` for this:

```sparql
PREFIX d3o: <https://purl.dsmz.de/schema/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT ?strain ?label ?genus ?species (CONCAT(?genus, " ", ?species) AS ?genusSpecies) WHERE {
    ?strain a d3o:Strain ;
            rdfs:label ?label ;
            d3o:hasGenus ?genus ;
            d3o:hasSpecies ?species .
}
LIMIT 5
```


## Filtering
Now we want to use the location of origin and filter for strains that are from Germany:

```sparql
PREFIX d3o: <https://purl.dsmz.de/schema/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?strain ?name ?origin WHERE {
    ?location a d3o:LocationOfOrigin ;
        d3o:describesStrain ?strain ;
        d3o:hasCountry ?country ;
		rdfs:label ?origin .

    ?strain a d3o:Strain ;
            rdfs:label ?name .

    FILTER (CONTAINS(?country, "Germany"))
}
LIMIT 5
```

We can also use numeric filters, e.g. for the optimal growth temperature of a strain:

```sparql
PREFIX d3o: <https://purl.dsmz.de/schema/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?strain ?name ?temperature  WHERE {
    ?tempObj a d3o:CultureTemperature ;
        d3o:describesStrain ?strain ;
        d3o:hasTemperatureRangeStart ?temperatureStart ;
        d3o:growthObserved 'positive' ;
        d3o:hasTestType 'optimum' ;
		rdfs:label ?temperature .

    ?strain a d3o:Strain ;
            rdfs:label ?name.

    FILTER (?temperatureStart > 30)
}
```

## Sorting

We can also use `ORDER BY` to sort the results:

```sparql
PREFIX d3o: <https://purl.dsmz.de/schema/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?strain ?name ?temperature  WHERE {
    ?tempObj a d3o:CultureTemperature ;
        d3o:describesStrain ?strain ;
        d3o:hasTemperatureRangeStart ?temperatureStart ;
        d3o:growthObserved 'positive' ;
        d3o:hasTestType 'optimum' ;
        rdfs:label ?temperature .

    ?strain a d3o:Strain ;
            rdfs:label ?name.

    FILTER (?temperatureStart > 30)
}
ORDER BY ?temperatureStart
```

## Aggregations

We can also use aggregations, e.g. to count the number of strains:

```sparql
PREFIX d3o: <https://purl.dsmz.de/schema/>
SELECT (COUNT(?strain) AS ?count) WHERE {
    ?strain a d3o:Strain .
}
```

Or to get the average temperature of all strains:

```sparql
PREFIX d3o: <https://purl.dsmz.de/schema/>
SELECT (AVG(?temperatureStart) AS ?averageTemperature) WHERE {
    ?tempObj a d3o:CultureTemperature ;
        d3o:describesStrain ?strain ;
        d3o:hasTemperatureRangeStart ?temperatureStart ;
        d3o:growthObserved 'positive' ;
        d3o:hasTestType 'optimum' .
}
```

## Grouping

We can also group the results, e.g. to count the number of strains per genus:

```sparql
PREFIX d3o: <https://purl.dsmz.de/schema/>
SELECT ?genus (COUNT(?strain) AS ?count) WHERE {
    ?strain a d3o:Strain ;
            d3o:hasGenus ?genus .
}
GROUP BY ?genus
```

Or to get the average temperature per genus:

```sparql
PREFIX d3o: <https://purl.dsmz.de/schema/>
SELECT ?genus (AVG(?temperatureStart) AS ?averageTemperature) WHERE {
    ?tempObj a d3o:CultureTemperature ;
        d3o:describesStrain ?strain ;
        d3o:hasTemperatureRangeStart ?temperatureStart ;
        d3o:growthObserved 'positive' ;
        d3o:hasTestType 'optimum' .
    ?strain d3o:hasGenus ?genus .
}
GROUP BY ?genus
```

## Optional

We can also use `OPTIONAL` to get additional information, e.g. the location of origin of a strain:

```sparql
PREFIX d3o: <https://purl.dsmz.de/schema/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?strain ?name ?origin WHERE {
    ?strain a d3o:Strain ;
            rdfs:label ?name .
    OPTIONAL {
        ?location a d3o:LocationOfOrigin ;
            d3o:describesStrain ?strain ;
            d3o:hasCountry ?country ;
            rdfs:label ?origin .
    }
}
LIMIT 5
```


