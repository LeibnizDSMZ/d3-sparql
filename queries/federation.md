# Federation


## MediaDive

A MediaDive query that retrieves the medium and its ingredients for a given BacDive.
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX d3o: <https://purl.dsmz.de/schema/>
PREFIX md: <https://purl.dsmz.de/mediadive/>
PREFIX dcterms: <http://purl.org/dc/terms/>
SELECT ?medium ?ingredient
WHERE {
	?strain a d3o:Strain ;
        d3o:hasBacDiveID 4907 .

    ?growth a d3o:GrowthCondition ;
        d3o:partOfMedium ?m ;
        d3o:relatedToStrain ?strain .

    ?m d3o:includesSolution ?solution ;
        rdfs:label ?medium .

	?recipe a d3o:SolutionRecipe ;
        d3o:partOfSolution ?solution ;
	    d3o:includesIngredient ?i .
    
    ?i rdfs:label ?ingredient .
}
```

Now we can use this in a federation query to get the ingredients for strains that are chemolithoautotroph.

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX d3o: <https://purl.dsmz.de/schema/>
PREFIX bd: <https://purl.dsmz.de/bacdive/>
SELECT ?strain ?bacdiveid ?medium ?ingredient
WHERE {
    ?nutrition a d3o:NutritionType;
     d3o:describesStrain ?dbStrain;
     rdfs:label ?nutritionType .
	 
  ?dbStrain rdfs:label ?strain ;
      d3o:hasBacDiveID ?bacdiveid .
  
  FILTER(?nutritionType = "chemolithoautotroph")
	
    # Use the BacDive ID to query the MediaDive endpoint
    SERVICE <https://sparql.dsmz.de/api/mediadive> {
	    ?mdStrain a d3o:Strain ;
            d3o:hasBacDiveID ?bacdiveid .
        ?growth a d3o:GrowthCondition ;
            d3o:partOfMedium ?m ;
            d3o:relatedToStrain ?mdStrain .
        ?m d3o:includesSolution ?solution ;
            rdfs:label ?medium .
        ?recipe a d3o:SolutionRecipe ;
            d3o:partOfSolution ?solution ;
            d3o:includesIngredient ?i .
        ?i rdfs:label ?ingredient .
    }

} 
```



<!-- 
## RIKEN

https://knowledge.brc.riken.jp/bioresource/sparql

### Find possible predicates

```sparql
PREFIX brso: <http://purl.jp/bio/10/brso/>

SELECT DISTINCT ?pred
WHERE {
	[] a brso:BiologicalResource ;
         ?pred [].
} LIMIT 100
```

### Example query for RIKEN JCM

```sparql
PREFIX brso: <http://purl.jp/bio/10/brso/>
PREFIX jcm: <http://metadb.riken.jp/db/rikenbrc_jcm_microbe/>

SELECT *
WHERE {
	jcm:JCM_7641 ?pred ?obj .
} LIMIT 100
```

### Federation query

> Note: RIKEN does apparently not support federated queries... I have to look at this in more detail.

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX d3o: <https://purl.dsmz.de/schema/>
PREFIX brso: <http://purl.jp/bio/10/brso/>
PREFIX jcm: <http://metadb.riken.jp/db/rikenbrc_jcm_microbe/>
PREFIX strain: <https://purl.dsmz.de/bacdive/strain/>
PREFIX terms: <http://purl.org/dc/terms/>

SELECT DISTINCT ?ccno ?predicate ?object
WHERE {
    # Query BacDive endpoint for the strain and its culture collection number
    ?cc a d3o:CultureCollectionNumber ;
            rdfs:label ?ccno ;
			d3o:describesStrain strain:4907 .
    FILTER(STRSTARTS(?ccno, "JCM"))

	# Use the ccno (e.g., "JCM 7641") to query the JCM endpoint
    SERVICE <https://knowledge.brc.riken.jp/endpoint> {
        ?jcmStrain terms:identifier ?ccno .
        ?jcmStrain ?predicate ?object .
    }
}
``` -->
