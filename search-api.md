# OVERVIEW

**Submit patient matching request:**
`HTTP POST` to remote server: `<base_remote_url>/match`
For example: `https://yourmatchmaker.org/match`


## Versioning

Every request must specify the API version within the HTTP `Accept` header.

`Accept: application/vnd.ga4gh.matchmaker.<version>+json`

Where `<version>` takes the form `vX.Y`. For example:

`Accept: application/vnd.ga4gh.matchmaker.v0.1+json`

The remote server must provide the API version of the response in the `Content-Type` header of every response:

`Content-Type: application/vnd.ga4gh.matchmaker.v0.7+json`

After receiving a request, the remote server can respond in one of two ways:
  * If a compatible version (`vX.Z` where `Z>=Y`) is supported by the remote server, it should provide a response using this version.
  * If no appropriate version is supported by the remote server, it should respond with `Not Acceptable (406)`, containing a JSON body with a description of the error. All responses, including this one, should contain a `Content-Type` header with the latest API version supported by the server. This will enable the user to re-submit the request using this version of the API.


## Search Request

`HTTP POST` request to `<base_remote_url>/match`, with an `application/json` body with the following format:

### Example

```
{
  "patient" : {
    "id" : <identifier>,
    "label" : <identifier>,

    "contact" : {
      "name" : "Full Name",
      "institution" : "Contact Institution",
      "href" : <URL>
    },

    "species" : <NCBI taxon identifier>,
    "sex" : "FEMALE"|"MALE"|"OTHER"|"MIXED_SAMPLE"|"NOT_APPLICABLE",
    "ageOfOnset" : <HPO code>,
    "inheritanceMode" : <HPO code>,

    "disorders" : [
      {
        "id" : "MIM:######"|"Orphanet:#####"|…
      },
      …
    ],
    "features" : [
      {
        "id" : <HPO code>,
        "observed" : "yes"|"no"|"unknown",
        "ageOfOnset" : "…"
      },
      …
    ],
    "genes" : [
      {
        "id" : <gene symbol>|<ensembl gene ID>|<entrez gene ID>
      },
      …
    ],
    "variants" : [
      {
        "assembly" : "NCBI36"|"GRCh37.p13"|"GRCh38.p1"|…,
        "referenceName" : "1"|"2"|…|"X"|"Y",
        "start" : <number>,
        "end" : <number>,
        "referenceBases" : "A"|"ACG"|…,
        "alternateBases" : "A"|"ACG"|…,
        "zygosity" : <number>,
        "type" : <mutation type>
      },
      …
    ]
  }
}
```

#### ID
* ***Mandatory***
* An identifier for the patient record, unique within the matchmaker where the patient data is stored. This identifier should be unchanged by modifications to the patient record over time (e.g. adding phenotypes). It may become invalid (e.g. if the record is deleted), but it should never be "replaced" and refer to a different patient.
* Transparent string, limited to 255 characters in utf-8.

#### Label
* *Optional*
* A name/identifier assigned by the user which can be used to reference the patient in a recognizable manner (in an email for example); it should not contain any *personally identifiable information*.
* Transparent string, limited to 255 characters in utf-8.

#### Contact
* ***Mandatory***
* The contact information describes how the eventual recipient of the match response can contact the owner of the matched patient record to follow-up on the match.
  1. `name` : The human-readable name of the clinician or organization that the user is contacting with the provided URL. A transparent string, limited to 255 characters in utf-8. (***Mandatory***)
  1. `institution` : The human-readable institution of the clinician, if available. A transparent string, limited to 255 characters in utf-8. (*Optional*)
  1. `href` : A public (no login required) URL for contacting the owner of the patient record to follow up with a match. This must be a valid URL (of the form `<scheme>:<address>`), and could take a number of forms: (***Mandatory***)
    * an `HTTP` URL: in this case, the URL could be a contact form which would allow the user to contact the owner of the matched patient.
    * a `mailto` URL: in this case, the URL could be a (potentially-anonymized) email address to contact regarding the patient match.

#### Species
* *Optional*
* A taxon identifier from the NCBI nomenclature, for the form: `"NCBITaxon:<ID>"`. The default is human: `"NCBITaxon:9606"`

#### Sex
* *Optional*
* This follows the [GA4GH `geneticSex` specification](https://github.com/ga4gh/schemas/blob/master/src/main/resources/avro/metadata.avdl), with the following options:
  * `FEMALE`: Genetic/chromosomal female
  * `MALE`: Genetic/chromosomal male
  * `OTHER`: sex information ambiguous, e.g. not clear XX/XY/ZZ...
  * `MIXED_SAMPLE`: Multiple samples, e.g. pooled, environmental
  * `NOT_APPLICABLE`: Used for prokaryotes, snails, etc. Not used for humans.

#### Age of onset
* *Optional*
* An [HPO](http://www.human-phenotype-ontology.org/) term identifier (HP:#######) associated with an age interval [as defined by the HPO](http://www.human-phenotype-ontology.org/hpoweb/showterm?id=HP:0011007)
  * `"HP:0003577"` (Congenital onset)
    * `"HP:0011460"` (Embryonal onset)
    * `"HP:0011461"` (Fetal onset)
  * `"HP:0003623"` (Neonatal onset)
  * `"HP:0003593"` (Infantile onset)
  * `"HP:0011463"` (Childhood onset)
  * `"HP:0003621"` (Juvenile onset)
  * `"HP:0003581"` (Adult onset)
    * `"HP:0011462"` (Young adult onset)
    * `"HP:0003596"` (Middle age onset)
    * `"HP:0003584"` (Late onset)

#### Inheritance Mode
* *Optional*
* An HPO term identifier (HP:#######) for a mode of inheritance (a descendant of `HP:0000005 (Mode of inheritance)`).
  * `"HP:0000006"` (Autosomal dominant inheritance)
    * `"HP:0001470"` (Sex-limited autosomal dominant)
      * `"HP:0001475"` (Male-limited autosomal dominant)
    * `"HP:0001444"` (Autosomal dominant somatic cell mutation)
    * `"HP:0001452"` (Autosomal dominant contiguous gene syndrome)
    * `"HP:0012274"` (Autosomal dominant inheritance with paternal imprinting)
    * `"HP:0012275"` (Autosomal dominant inheritance with maternal imprinting)
  * `"HP:0000007"` (Autosomal recessive inheritance)
  * `"HP:0001472"` (Familial predisposition)
  * `"HP:0001426"` (Multifactorial inheritance)
    * `"HP:0010984"` (Digenic inheritance)
    * `"HP:0010983"` (Oligogenic inheritance)
    * `"HP:0010982"` (Polygenic inheritance)
  * `"HP:0001427"` (Mitochondrial inheritance)
  * `"HP:0001425"` (Heterogeneous)
  * `"HP:0001428"` (Somatic mutation)
    * `"HP:0001442"` (Somatic mosaicism)
  * `"HP:0001466"` (Contiguous gene syndrome)
    * `"HP:0001452"` (Autosomal dominant contiguous gene syndrome)
  * `"HP:0003745"` (Sporadic)
  * `"HP:0003743"` (Genetic anticipation)
    * `"HP:0003744"` (Genetic anticipation with paternal anticipation bias)
  * `"HP:0010985"` (Gonosomal inheritance)
    * `"HP:0001417"` (X-linked inheritance)
      * `"HP:0001419"` (X-linked recessive inheritance)
      * `"HP:0001423"` (X-linked dominant inheritance)
    * `"HP:0001450"` (Y-linked inheritance)

#### Disorders
* *Optional*
* Is a list of [OMIM](http://omim.org/) (`MIM:######`) or [OrphaNet](http://www.orphadata.org/) (`Orphanet:#####`, where the fragment is either numeric or `C####`) identifiers, can be empty
* NOTE: we may want to support other sources later.

#### Features
* It is ***mandatory*** to have at least one of these three: `features`, `genes`, `variants` (having all is preferred)
* Is a **list of features** described by:
  * `id`: an  HPO term identifier of the form: `HP:#######`
  * `observed`: `"yes"`|`"no"`|`"unknown"`
  * `ageOfOnset`: same as the global age of onset described above (*optional*; system which do not support this type of information per symptom should ignore it)
* More metadata can be later added to each feature if necessary.
* By default we shouldn’t sent any features with the `observed` status (or value) `"unknown"`

#### Genes
* It is ***mandatory*** to have at least one of these three: `features`, `genes`, `variants` (having all is preferred)
* Is a **list of candidate causal genes** described by:
  * `gene`:
    * `<gene symbol>` from the [HGNC database](http://www.genenames.org/) OR
    * `<ensembl gene ID>` OR
    * `<entrez gene ID>`

#### Variants
* It is ***mandatory*** to have at least one of these three: `features`, `genes`, `variants` (having all is preferred)
* Is a **list of candidate genomic variants** described by:
  * `assembly`: reference assembly identifier, including patch number if relevant, of the form: `<assembly>[.<patch>]` (***mandatory***)
    * example valid values: `"NCBI36"`, `"GRCh37"`, `"GRCh37.p13"`, `"GRCh38"`, `"GRCh38.p1"`
    * If the patch is not provided, the assembly is assumed to represent the initial (unpatched) release of that assembly.
  * `referenceName`: `"1"`, `"2"`, …, `"22"`, `"X"`, `"Y"`; the chromosome this variant or gene is on (***mandatory***)
  * `start`: `<number>`; the start position of the variant. (0-based) (*optional*)
  * `end`: `<number>`; the end position of the variant. (0-based, exclusive) (*optional*)
  * `referenceBases`: `"A"`|`"ACG"`|…, VCF-style reference of at least one base (*optional*)
  * `alternateBases`: `"A"`|`"ACG"`|…, VCF-style alternate allele of at least one base (*optional*)
  * `zygosity`: `<number>` (`1` for heterozygous or hemizygous, `2` for homozygous) (*optional*)
  * `type`: the (*optional*) type of mutation, as a means to describe the broad category of cDNA effect predicted to result from a mutation to improve matchmaking, without disclosing the actual mutation:
    * `TRUNCATING` (e.g. stopgain, stoploss, startloss, frameshift indel)
    * `ALTERING` (e.g. missense, non-frameshift indel)
    * `SPLICING`
    * `UTR` (UTR3, UTR5)
    * `INTRONIC`
    * `PROXIMAL` (e.g. upstream, downstream)
    * `OTHER` (e.g. motif disruption, synonymous)

## Search Results Response
A synchronous `application/json` response, of the following form:

### Example

```
{
  "results" : [
    {
      "patient" : {…},
    },
    …
  ]
}
```

#### Results
* ***Mandatory***, but can be empty
* Is a **list of matches**, where each match has a `patient` object of the same format as the one described above for the query


### Error handling
The remote server should use HTTP status codes to report any errors encoundered processing the match request. Here are a list of status codes and their meanings with regards to this API:

| HTTP Status Code | Constant | Description
| ---------------- | -------- | -----------
| 200 | httplib.OK | no error |
| 400 | httplib.BAD_REQUEST | missing/invalid data
| 401 | httplib.UNAUTHORIZED | invalid API key
| 405 | httplib.METHOD_NOT_ALLOWED | invalid method (GET)
| 406 | httplib.NOT_ACCEPTABLE | unsupported API version
| 415 | httplib.UNSUPPORTED_MEDIA_TYPE | missing/invalid content type
| 422 | httplib.UNPROCESSABLE_ENTITY | missing/invalid request body
| 500 | httplib.INTERNAL_SERVER_ERROR | default error

The error response should include a json-formatted body with a human-readable `"message"` providing details the error. For example, if the match request specifies an unsupported API version, the server should respond with `Not Acceptable (406)` and content such as:

```
{
  "message" : "unsupported version number"
}
```
