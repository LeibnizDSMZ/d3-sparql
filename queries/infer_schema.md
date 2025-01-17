
# Infer schema by using SPARQL

We can also infer the schema of the data by using SPARQL.

## Get all classes

```sparql
SELECT DISTINCT ?type WHERE {
    [] a ?type .
}
```

## Get all properties of a class

We can also get all properties of a class:

```sparql
PREFIX d3o: <https://purl.dsmz.de/schema/>
SELECT DISTINCT ?predicate WHERE {
  [] a d3o:Strain ;
           ?predicate ?object .
}
```

## Get all objects of a property

We can also get all objects of a property:

```sparql
PREFIX d3o: <https://purl.dsmz.de/schema/>
SELECT DISTINCT ?object WHERE {
    [] a d3o:Strain ;
        d3o:hasGenus ?object .
}
```