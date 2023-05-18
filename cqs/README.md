# Competency Questions

The list of PECO competency questions.

## SPARQL queries used for implementing the CQs

To test the CQs use the provenance_trace.ttl and https://raw.githubusercontent.com/TEC-Toolkit/cfkg/main/wikidata_labels/wikidata_labels.ttl 

### Prefixes

```sparql
PREFIX peco: <https://w3id.org/peco#> 
PREFIX ecfo: <https://w3id.org/ecfo#>  
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> 
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#> 
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>  
PREFIX prov:<http://www.w3.org/ns/prov#> 
PREFIX qudt: <http://qudt.org/schema/qudt/>
```

### CQ1: What are the emissions of an activity (e.g., training a model)?

```sparql
Select ?activityL ?score

Where {
    ?activity  rdf:type   peco:EmissionGenerationActivity ;
               rdfs:label ?activityL;
               peco:hasEmissionScore/qudt:value ?score. 
    
}
```

#### Answer

| activityL | score |
|-|-|
| ML model training | "5.25E0"^^xsd:float |

### CQ2: Who was responsible for the activity that produced emissions?



```sparql
Select ?agent

Where {
    ?activity  rdf:type   peco:EmissionGenerationActivity ;
               prov:wasassociatedWith ?agent. 
    
}
```

#### Answer

| agent |
|-|
| http://example.com#Alice |

### CQ3: What are the emissions of the activities I was responsible for in a specific year?



```sparql
Select ?activityLabel ?score

Where {
    ?activity a peco:EmissionGenerationActivity;
              rdfs:label ?activityLabel;
              peco:hasEmissionScore/qudt:value ?score;
              prov:startedAtTime ?start;
              prov:endedAtTime ?end.
    
    FILTER (?start > "2023-01-01T00:00:00"^^xsd:dateTime)
    FILTER (?end < "2023-12-31T23:59:00"^^xsd:dateTime)
}
```

#### Answer

| activityLabel | score|
|-|-|
| ML model training |"5.25E0"^^xsd:float|

### CQ4: What Emission Conversion Factor was used to calculate the emissions of this activity?


```sparql
Select ?ecf

Where {
    ?activity  rdf:type   peco:EmissionCalculationActivity ;
               prov:used ?ecf. 
    ?ecf a ecfo:EmissionConversionFactor.
    ?emissionScore a peco:EmissionScore;
                   prov:wasGeneratedBy ?activity.
}

```

#### Answer

| ecf |
|-|
| https://w3id.org/ecfkg/i/UK/BEIS/2021/CF_1826 |


### CQ5: What different steps did the emission calculation process include?



```sparql
Select ?activityLabel 

Where {
    ?activity  rdf:type   peco:EmissionCalculationActivity ;
               rdfs:label  ?activityLabel.
}
```

#### Answer

| activityLabel |
|-|
| Estimate Electricity Use in kW/h |
| Emission Score Calculation |

### CQ6: Who was responsible for calculating the emissions?



```sparql
Select ?activityL ?software ?agent

Where {
    ?activity  rdf:type   peco:EmissionCalculationActivity ;
               prov:wasAssociatedWith ?software; 
               rdfs:label ?activityL.
    ?software prov:actedOnBehalfOf ?agent.
}

```
Answer:

| activityL | software |agent|
|-|-|-|
| Emission Score Calculation | https://github.com/TEC-Toolkit/Semantic_Machine_Learning_Impact_Calculator |http://example.com#Alice|
| Estimate Electricity Use in kW/h | https://github.com/TEC-Toolkit/Semantic_Machine_Learning_Impact_Calculator |http://example.com#Alice|

### CQ7: What outputs and in what quantities did the emission calculation process use to generate the emission score?

```sparql
Select ?activityLabel ?outputLabel ?outputValue ?outputUnitLabel ?outputQuantityKindL

Where {
    ?activity  rdf:type   peco:EmissionCalculationActivity ;
               rdfs:label  ?activityLabel.
    ?output prov:wasGeneratedBy ?activity.
    
    ?output     rdf:type              peco:EmissionCalculationEntity ;
               rdfs:label            ?outputLabel ;
               qudt:value            ?outputValue ;
               qudt:hasQuantityKind/rdfs:label   ?outputQuantityKindL;
               qudt:unit/qudt:abbreviation       ?outputUnitLabel .
             
    FILTER ( lang(?outputUnitLabel) = "en" )
    
}
```

#### Answer

| activityLabel | outputLabel | outputValue | outputUnitLabel | outputQuantityKindL
|-|-|-|-|-|
| Estimate Electricity Use in kW/h | Energy Used| "25"^^xsd:float| "kWh"@en| "electricity"@en |
| Emission Score Calculation | Emission Score| "5.25E0"^^xsd:float| "kg"@en| "carbon dioxide equivalent"@en |

We can see the evolution of the conversion factor for five years. For one of the years there are two conversion factors for the same materials,

### CQ8: What inputs and in what quantities did the emission calculation process use to generate the emission score?

```sparql
Select ?activityLabel ?inputLabel ?inputValue ?inputUnitLabel ?inputQuantityKindL

Where {
    ?activity  rdf:type   peco:EmissionCalculationActivity ;
               rdfs:label  ?activityLabel ;
               prov:used   ?input.
    {
    ?input     rdf:type              peco:EmissionCalculationEntity ;
               rdfs:label            ?inputLabel ;
               qudt:value            ?inputValue ;
               qudt:hasQuantityKind/rdfs:label   ?inputQuantityKindL;
               qudt:unit/qudt:abbreviation       ?inputUnitLabel .
             
    FILTER ( lang(?inputUnitLabel) = "en" )
    }
    UNION
      { 
       #look for Emission conversion factors too
        ?input  rdf:value  ?inputValue .
        ?input ecfo:hasTargetUnit/qudt:abbreviation ?inputUnitLabel .
        ?input ecfo:hasEmissionTarget/rdfs:label ?inputQuantityKindL .
        ?input rdfs:label  ?inputLabel
        FILTER ( lang(?inputQuantityKindL) = "en" )
      }
}

```

#### Answer

| activityLabel | inputLabel | inputValue | inputUnitLabel | inputQuantityKindL
|-|-|-|-|-|
| Estimate Electricity Use in kW/h | Duration of Use| "100"^^xsd:integer| "h"@en| "time"@en |
| Estimate Electricity Use in kW/h | Watt Consumption| "250"^^xsd:float| "W"@en| "thermal design power"@en |
| Emission Score Calculation | Energy Used| "25"^^xsd:float| "kWh"@en| "electricity"@en |
| Emission Score Calculation | Electricity: UK| "0.21233"| "kg"@en| "carbon dioxide equivalent"@en |


### CQ9: What entity (e.g, hardware consuming electricity) was used to transform the emission source into emissions?

For example, a hardware (e.g., GPU) that used electricity.

```sparql
Select ?foi
Where
{
    ?activity a peco:EmissionGenerationActivity.
    ?observation peco:inEmissionActivityContext ?activity;
                 sosa:hasFeatureOfInterest/rdfs:label ?foi.
    
}
```

#### Answer

| foi |
|-|
| A100 PCIe 40/80GB |

### CQ10: What is the location associated with the activity that produced emissions?

```sparql
Select ?location 
Where
{
    ?activity a peco:EmissionGenerationActivity;
                prov:atLocation ?location.
}
```

Answer:
| location |
|-|
| https://w3id.org/ecfkg/i/mlco2/gcp/europe-west2 | 



