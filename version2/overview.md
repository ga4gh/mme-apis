## Current state ##
Below is the current form of the API.

The data in the patient object serves two purposes:
* To specify a query that the service will try to match
* To accurately describe the patient, so that matched users can evaluate the quality of the match as quickly as possible.

These two goals are sometimes at odds. For example, you may know the patient's age of onset (neonatal), but do not want to match on it. Currently, you would just leave that information out, but then the owner of the matched record will not know the age of onset, which could be important.

This proposal attempt to address this by adding a separate "query" field that stores constraints that are provided by a user or a system and exist outside of the patient data. There is specific focus on the "genome" component and the queries that it supports to enable 1-sided hypothesis matchmaking.

To set the stage, this is current the structure of a request:
<pre>
{
  "patient": {
    "contact": {...}
    "sex"
    "species"
    "ageOfOnset"
    "inheritanceMode"
    "diagnoses"
    "features"
    "genomicFeatures"
  }
}
</pre>

And a response:
<pre>
{
  "results": [
    {
      "score": {...}
      "patient": {...}
    }
  ]
}
</pre>



Three larger additions:

1. The addition of the required query "meta" field, to hold versions of components and other query metadata
1. The addition of the optional "query" field at the top level, to support constraints that extend beyond the match-by-example framework
1. The addition of the optional "genome" component

To support the following forms of exome/genome queries:

1. Return patients with a rare (AF < 0.01 in ExAC) missense or nonsense (as annotated by VEP, Annovar, or Jannovar) variant in GENEX


## Data "components" ##
Proposal: top-level data categories become "components", with their own versioning. This will allow backwards-incompatible changes in a component to only affect those services that use that component, unlike if we did a full API version increment.

Proposal: the addition of a "meta" field on the query, to hold versions of components and any other query-level metadata. This seems to be the obvious place, since component versions are tied to the service not the patient data. Further, this allows specifying versions for "functional" components that impose constraints but don't match on specific data in the patient (such as the "genome" component in 1-sided matchmaking).

Proposal: these versions be integer versions that are incremented when backwards incompatible changes are made

Proposed match query:
<pre>
{
  <strong style="background-color: #ff9">"meta": {
    "components": {
      "ageOfOnset": {
        "version": 1
      }
      "features": {
        "version": 3
      }
      ...
    }
  }</strong>
  "patient": {
    "contact"
    "sex"
    "species"
    "ageOfOnset"
    "inheritanceMode"
    "diagnoses"
    "features"
    "genomicFeatures"
  }
}
</pre>


## Per-component filters/constraints and scores ##
Proposal: the addition of the optional "query" field at the top level. This would support constraints (e.g. gene must match) that extend beyond the match-by-example framework, on a per-component basis. This is important to ensure that the results are useful for the querier and meet their expectations about what makes a match relevant.

Backwards-compatibility: If a component doesn't have a query, then it should fall back to performing a `LIKE` query on the data in the patient description.

Proposal: the addition of an optional per-component score field associated with each result. TODO: decide whether to keep these as raw scores, replace with bins/confidence levels, or allow both.

How this would fit into the overall query:
<pre>
{
  "meta": {...}
  "patient": {...}
  <strong style="background-color: #ff9">"query": {
    "components": {
      "features": {...}
      "genomicFeatures": {...}
    }
  }</strong>
}
</pre>

Each component's query object has the following reserved fields:
* `component`:
    * `optional` (default)
    * `mandatory`: a compatible version of the component must be supported by the service, else no results should be returned.
* `value`:
    * `optional` (default)
    * `mandatory`: the case should have data for the component in order to match. For example, if `value: mandatory` for the "genomicFeatures" component, only cases with some candidate genomic feature should be returned.

A service must therefore inspect all `query.components.*` objects to ensure that mandatory components are supported.

Each component may provide a numerical score (0-1) in the response:
<pre>
{
  "results": [
    {
      "score": {
        "patient": 0.5
        <strong style="background-color: #ff9">"features": 0.2
        "genomicFeatures": 0.8</strong>
      }
      "patient": {...}
    }
  ]
}
</pre>



### Component: `genome` sequencing results ###

The query for the `genome` component consists of:
* A list of filters for selecting rare, predicted-pathogenic variants in particular genes. If multiple filters are provided, only variants that pass all filters are matched.
* One or more possible modes of inheritance that must be consistent with the number and types of variants in the gene.

Each variant is annotated with a number of attributes, including its position (`start`, `end`, etc.), consequence information (`gene`, `consequence`), and population data (`alleleFrequency`). Each annotation has a data type, which defines which filters are supported for that annotation.

Annotation types:
* integer
    * `operator`: `EQ` (default), `NEQ`, `LT`, `LTE`, `GT`, `GTE`
        * single value (`value`)
* float
    * `operator`: `LTE`, `GTE`
        * single value (`value`)
* nominal/categorical/ontological
    * `operator`:
        * single value (`term`):
            * `EQ` (default): Match must contain the term or a descendant of the term
        * multiple values (`terms`): `LIKE` (default), `ANY`, `ALL`
            * `ANY`: For at least one of the terms, the term or a descendant of that term must be present in the match
            * `ALL`: For every term, the term or a descendant of that term must be present in the match

Fields:
* `inheritanceMode`: contains a field `terms` with a list of HPO terms for modes of inheritance (e.g. AD, AR, X-linked, sporadic). Matches should have genes with variants that are consistent with at least one of the included modes of inheritance. For example, if the inheritanceMode is AR ("HP:0000007"), matching cases must have at least 2 variant alleles that pass the filters (2 het variants or 1 hom variant).
* `filters`: a list of filters, each with one or more of the following subfields (depending on the annotation):
    * `annotation`
    * `source` (used by many annotations)
    * `operator`
    * `population` (used by `alleleFrequency` annotation)
    * `value|term|terms` (depending on number and type of annotation)

Annotations:
* `referenceName`: nominal
* `start`: integer
* `end`: integer
* `size`: integer
* `id`: nominal; dbSNP identifier (or other)
* `gene`: nominal; additional fields:
    * `source`: Ensembl, RefSeq, Entrez, UCSC
    * `terms`: list of gene terms
* `consequence`: nominal (SO term); additional fields:
    * `source`: VEP, ANNOVAR, Jannovar
    * `terms`: list of SO terms
* `alleleFrequency`: float; additional fields:
    * `source`: 1000GP, ESP5600, ExAC, local_db_name
    * `population`: ALL, CEU, ...
* `score`: float; additional fields:
    * `source`: SIFT, PolyPhen2, MutationTaster, CADD


For example, to match against cases with 2 or more rare (AF < 0.01) harmful (missense or stopgain) variants in NGLY1 or TTN:
<pre>
  "query": {
    "components": {
      "genome": {
        "inheritanceMode": {
          "terms": [
            {
              "id": "HP:0000007",
              "label": "Autosomal recessive inheritance"
            }
          ]
        }
        "filters": [
          {
            "annotation": "gene"
            "source": "Ensembl"
            "operator": "ANY"
            "terms": [
              {"id": "ENSG...", "label": "EFTUD2"}
              {"id": "ENSG...", "label": "TTN"}
            ]
          }
          {
            "annotation": "alleleFrequency"
            "source": "ExAC"
            "population": "ALL"
            "operator": "LT"
            "value": 0.01
          }
          {
            "annotation": "consequence"
            "source": "VEP"
            "operator": "ANY"
            "terms": [
              {"id": "SO:...", "label": "Stopgain"}
              {"id": "SO:...", "label": "Missense"}
            ]
          }
        ]
      }
    }
  }
</pre>


### Component: `features` ###

Provides support for requiring the presence of specific phenotype terms in a matching patient.

Fields:
* `filters`: a list of one or more filters that are each required to be true for a match to be returned
    * `terms`: A list of phenotype terms, containing one or more of the following fields:
        * `id`: the HPO term ID; must match term or a descendant
        * `ageOfOnset`: the age of onset as an HPO term; must match term or a descendant
    * `operator`:
        * `LIKE`: A fuzzy match
        * `ALL`: For every term in `terms`, the term or a descendant of that term must be present in the match

Backwards-compatibility: If `terms` is not provided, the phenotypes from `patient.features` should be used.


Example: If the patient has any phenotypic data, they must have a facial abnormality

<pre>
  "query": {
    "components": {
      "features": {
        "filters": [
          {
            "operator": "ALL"
            "terms": [
              {
                "id": "HP:0000271"
                "label": "Abnormality of the face"
              }
            ]
          }
        ]
      }
    }
  }
</pre>

Example: Only return matches with facial abnormalities
<pre>
  "query": {
    "components": {
      "features": {
        <strong style="background-color: #ff9">"component": "mandatory"
        "value": "mandatory"</strong>
        "filters": [
          {
            "operator": "ALL"
            "terms": [
              {
                "id": "HP:0000271"
                "label": "Abnormality of the face"
              }
            ]
          }
        ]
      }
    }
  }
</pre>


Example: Match with patients with congenital facial abnormalities
<pre>
  "query": {
    "components": {
      "features": {
        "component": "mandatory"
        "value": "mandatory"
        "filters": [
          {
            "operator": "ALL"
            "terms": [
              {
                "id": "HP:0000271"
                "label": "Abnormality of the face"
                "ageOfOnset": {
                  "id": "HP:0003577"
                  "label": "Congenital onset"
                }
              }
            ]
          }
        ]
      }
    }
  }
</pre>


Questions:
* How to handle unobserved terms?
* Do we have a use case for "ANY"?


### Component: `genomicFeatures` ###
This provides matching for manually selected candidate genes and/or variants.

Fields:
* `filters`: a list of one or more filters that are each required to be true for a match to be returned
    * `terms`: A list of candidate genes or candidate variants, containing the following fields:
        * `gene`: the `id` must be an exact match
        * `zygosity`: must be an exact match
        * `consequence`: match must be the term or a descendant
        * `variant`: one or more of the following sub-fields may be provided and must match exactly
            * `assembly`: must be an exact match
            * `referenceName`: must be an exact match
            * `start`: must be an exact match
            * `end`: must be an exact match
            * `id`: must be an exact match
            * `referenceBases`: must be an exact match
            * `alternateBases`: must be an exact match
    * `operator`:
        * `LIKE` (default): A fuzzy match
        * `ANY`: At least one of the terms in `terms` must match

Backwards-compatibility: If `terms` is not provided, the genomic features from `patient.genomicFeatures` should be used.


Example 1. If the case has any candidate genes, SRCAP must be included:
<pre>
  "query": {
    "components": {
      "genomicFeatures": {
        "filters": [
          {
            "operator": "ANY"
            "terms": [
              {
                "gene": {"id": "ENSG00000080603", "label": "SRCAP"}
              }
            ]
          }
        ]
      }
    }
  }
</pre>


Example 2. Find other cases with delta F508
<pre>
  "query": {
    "components": {
      "genes": {
        "filters": [
          {
            "operator": "ANY"
            "terms": [
              {
                "variant": {
                  "assembly": "GRCh37.p13"
                  "referenceName": "7"
                  "start": 117199645
                  "referenceBases": "CTT"
                  "alternateBases": ""
                }
              }
            ]
          }
        ]
      }
    }
  }
</pre>




## Service status/configuration endpoint ##
Proposal: a machine-readable endpoint that enables:
* verifying that a service is online
* determining the API versions, components (and their versions), and queries (e.g. genomic annotations) supported by the endpoint

Tudor Groza has been doing work on this, but at it's simplest, you could imagine it behaving something like the very simple example below:

POST /status

Response:
<pre>
{
  "url": "https://phenomecentral.org/rest/remoteMatcher/"
  "label": "PhenomeCentral"
  "type": "production"
  "disclaimer": "By using data from PhenomeCentral, you agree..."
  "versions": [
    {
      "api": {
        "version": "2.0"
      }
      "components": {
        "genome": {
          "version": 1
          "annotations": [
            {
              "annotation": "alleleFrequency"
              "source": "1000GP"
              "population": "ALL"
              "version": "phase3"
            }
            {
              "annotation": "alleleFrequency"
              "source": "PhenomeCentral"
              "population": "matchable"
              "version": "2015-12-14"
            }
            {
              "annotation": "score"
              "source": "SIFT"
              "version": "2015-01-01"
            }
          ]
        }
        "features": {
          "version": 2
          "scores": [
            {
              "score": "simGIC"
              "description": "Intersection of..."
            }
          ]
        }
      }
    }
  ]
}
</pre>

