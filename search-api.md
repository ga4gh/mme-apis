# OVERVIEW

**Submit patient matching request:**
`HTTP POST` to remote server: `<base_remote_url>/match`
For example: `https://yourmatchmaker.org/match`


## Security

This is a protected resource and as such requires the [use of an API key to access](/join-protocol.md).

## Versioning

Every request must specify the API version within the HTTP `Content-Type` header.

`Content-Type: application/vnd.ga4gh.matchmaker.<version>+json`

Where `<version>` takes the form `vX.Y`, where `X` is a major version and `Y` is a minor version. Minor versions are cross-compatible. For example:

`Content-Type: application/vnd.ga4gh.matchmaker.v1.0+json`

After receiving a request, the remote server can respond in one of two ways:
  * If a compatible version (with the same major version `X`) is supported by the remote server, it should provide a response using this version. The response should include the version of the server in the `Content-Type` header of the response.
  * If no appropriate version is supported by the remote server, it should respond with `Not Acceptable (406)`, containing a JSON body with a description of the error. The response should contain a `Content-Type` header with the latest API version supported by the server. This will enable the user to potentially re-submit the request using this version of the API.


## Search Request

`HTTP POST` request to `<base_remote_url>/match`, with an `application/json` body with the following format:

### Example

```
{
  "disclaimer" : "Disclaimer text...",
  "terms" : : "Terms text...",
  "patient" : {
    "id" : <identifier>,
    "label" : <identifier>,

    "contact" : {
      "name" : "Full Name",
      "institution" : "Contact Institution",
      "href" : <URL>,
      "email" : "full.name@institution.edu",
      "roles" : [
        "clinician"|"researcher"|"patient",
        …
      ]
    },

    "species" : <NCBI taxon identifier>,
    "sex" : "FEMALE"|"MALE"|"OTHER"|"MIXED_SAMPLE"|"NOT_APPLICABLE",
    "ageOfOnset" : <HPO code>,
    "inheritanceMode" : <HPO code>,

    "disorders" : [
      {
        "id" : "MIM:######"|"Orphanet:#####"|…,
        "label" : "Disease name"
      },
      …
    ],
    "features" : [
      {
        "id" : <HPO code>,
        "label" : "Feature description",
        "observed" : "yes"|"no",
        "ageOfOnset" : "…"
      },
      …
    ],
    "genomicFeatures" : [
      {
        "gene" : {
          "id" : <ensembl gene ID>|<entrez gene ID>|<gene symbol>
        },
        "variant" : {
          "assembly" : "NCBI36"|"GRCh37.p13"|"GRCh38.p1"|…,
          "referenceName" : "1"|"2"|…|"X"|"Y",
          "start" : <number>,
          "end" : <number>,
          "referenceBases" : "A"|"ACG"|…,
          "alternateBases" : "A"|"ACG"|…
        },
        "zygosity" : <number>,
        "type" : {
          "id" : <SO code>,
          "label" : "STOPGAIN"
        }
      },
      …
    ]
  }
}
```

#### Disclaimer
* *Optional*
* Disclaimer text for the service, this is optional and can be included in all requests and responses. If included it is assumed that it supersedes the disclaimer stored in the GitHub repository.
* Note that the diclaimer should be shown with the returned data per item 6 of the MME service agreement.
* [See existing disclaimers](/disclaimers/).

#### Terms
* *Optional*
* Terms and conditions text for the service, this is optional and can be included in all requests and responses.
* Note that the diclaimer should be shown with the returned data per item 6 of the MME service agreement.

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
  * `name` : The human-readable name of the person or organization that the user is contacting with the provided URL. A transparent string, limited to 255 characters in utf-8. (***Mandatory***)
  * `institution` : The human-readable institution of the contact person, if available. A transparent string, limited to 255 characters in utf-8. (*Optional*)
  * `roles` : A list of roles of the contact person. These may be self-declared by the submitting user, and might not have been verified by the sending server. (*Optional*) (*since v1.1)
    * `clinician`: The contact person is a clinician responsible for the patient's care
    * `researcher`: The contact person is a researcher with the necessary consent to submit the patient record
    * `patient`: The contact person is the patient or a caregiver for the patient
  * `href` : A public (no login required) URL for contacting the owner of the patient record to follow up with a match. This must be a valid URL (of the form `<scheme>:<address>`), and could take a number of forms: (***Mandatory***)
    * An `HTTP` URL: in this case, the URL could be a contact form which would allow the user to contact the owner of the matched patient.
    * A `mailto` URL: in this case, the URL could be a (potentially-anonymized) email address to contact regarding the patient match. It is preferred to use the `email` field for this purpose.
  * `email` : A (potentially-anonymized) email address for contacting the owner of the patient record to follow up with a match. (*Optional*) (*since v1.1*)

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
* Is a (potentially empty) ***list of disorders*** described by:
  * `id`: an [OMIM](http://omim.org/) (`MIM:######`) or [OrphaNet](http://www.orphadata.org/) (`Orphanet:#####`) identifier (***mandatory***)
  * `label` : a human readable description of the disorder (*optional*) (*since v1.1*)
* NOTE: we may want to support other sources later.

#### Features
* It is ***mandatory*** to have at least one of these two: `features`, `genomicFeatures` (having both is preferred)
* Is a **list of features** described by:
  * `id`: an HPO term identifier of the form: `HP:#######` (***mandatory***)
  * `label`: a human readable description of the phenotypic feature, such as the HPO term name (*optional*) (*since v1.1*)
  * `observed`: `"yes"`|`"no"` defines if the feature has been _explicitly observed_ (`yes`) or _explicitly not observed_ (`no`). Omission of this optional field assumes the feature has been _explicitly observed_. (*optional*)
  * `ageOfOnset`: same as the global age of onset described above (*optional*; system which do not support this type of information per symptom should ignore it)
* More metadata can be later added to each feature if necessary.

#### GenomicFeatures
* It is ***mandatory*** to have at least one of these two: `features`, `genomicFeatures` (having both is preferred)
* Is a **list of candidate causal genes and variants** described by:
  * `gene`: (***mandatory***)
    * `id`: A gene symbol or identifier (***mandatory***):
      * `<ensembl gene ID>` OR
      * `<entrez gene ID>` OR
      * `<gene symbol>` from the [HGNC database](http://www.genenames.org/)
      * The use of ensembl gene ID is ***strongly*** encouraged and will become mandatory in 2.0 
  * `variant` (*optional*): the specific variant
    * `assembly`: reference assembly identifier, including patch number if relevant, of the form: `<assembly>[.<patch>]` (***mandatory*** if `variant` is provided)
      * example valid values: `"NCBI36"`, `"GRCh37"`, `"GRCh37.p13"`, `"GRCh38"`, `"GRCh38.p1"`
      * If the patch is not provided, the assembly is assumed to represent the initial (unpatched) release of that assembly.
    * `referenceName`: `"1"`, `"2"`, …, `"22"`, `"X"`, `"Y"`; the chromosome this variant is on (***mandatory*** if `variant` is provided)
    * `start`: `<number>`; the start position of the variant. (0-based) (***mandatory*** if `variant` is provided)
    * `end`: `<number>`; the end position of the variant. (0-based, exclusive) (*optional*)
    * `referenceBases`: `"A"`|`"ACG"`|…, VCF-style reference of at least one base (*optional*)
    * `alternateBases`: `"A"`|`"ACG"`|…, VCF-style alternate allele of at least one base (*optional*)
  * `zygosity`: `<number>` (`1` for heterozygous or hemizygous, `2` for homozygous) (*optional*)
  * `type`: the effect of the mutation. This enables describing the broad category of cDNA effect predicted to result from a mutation to improve matchmaking, without necessarily disclosing the actual mutation. (*optional*)
    * `id`: a Sequence Ontology term identifier (`"SO:#######"`). This will usually (but not necessarily) be a descendant of [SO:0001576 [transcript variant]](http://www.sequenceontology.org/browser/current_svn/term/SO:0001576). (***mandatory***, if `type` is provided)
    * `label`: a human-readable description of the effect. For example, the JANNOVAR effect annotation. (*optional*)

## Search Results Response
A synchronous `application/json` response, of the following form:

### Example

```
{
  "disclaimer" : "Disclaimer text...",
  "terms" : : "Terms text...",
  "results" : [
    {
      "score" : {
        "patient" : <number>
      },
      "patient" : {…},
    },
    …
  ]
}
```

#### Disclaimer & Terms
* *Optional*
* Disclaimer & terms for the data, this is symetrical to the request.
* [See existing disclaimers](/disclaimers/).

#### Results
* ***Mandatory***, but can be empty.
* Is a **list of matches**, where each match has a `patient` object, and a `score` object with information about how well the `patient` object matched.

##### Score
* ***Mandatory***
* Information about how well this results patient matched the query patient.
* Currently, this has a single ***mandatory*** field, `patient`, with a numerical value corresponding to the overall score of the match. This score must be in the range [0, 1], where 0.0 is a poor match and 1.0 is a perfect match.

##### Patient
* ***Mandatory***
* A `patient` object of the same form as the one described above for the query.

### Non-Standard Fields
Non-standard fields can be added to requests and responses with an underscore prefix as follows:
```
"gene": {
        "_ensemblGeneID": "ENSG00000133392", 
        "_entrezGeneID": "4629", 
        "_geneName": "MYH11", 
        "id": "ENSG00000133392"
    }, 
```
The goal is to allow implementations to add data that are useful for tracking/debugging/reference but are not part of the official specification.

### Error handling
The remote server should use HTTP status codes to report any errors encountered processing the match request. Here are a list of status codes and their meanings with regards to this API:

| HTTP Status Code | Reason Phrase | Description
| ---------------- | -------- | -----------
| 200 | OK | no error |
| 400 | Bad Request | missing/invalid data
| 401 | Unauthorized | invalid API key
| 405 | Method Not Allowed | invalid method (GET)
| 406 | Not Acceptable | unsupported API version
| 415 | Unsupported Media Type | missing/invalid content type
| 422 | Unprocessable Entity | missing/invalid request body
| 500 | Internal Server Error | default error

The error response should include a json-formatted body with a human-readable `"message"` containing further details about the error. The exact error message is up to the implementer, and additional fields can be provided with further information. For example, if the match request specifies an unsupported API version, the server should respond with `Not Acceptable (406)` and a content body such as:

```
{
  "message" : "unsupported API version",
  "supportedVersions" : [ "0.1", "1.0", "1.1" ]
}
```

### Testing
Matchmakers are strongly encouraged to test the ability of their systems to query, match, and respond to requests. Standardized test data is provided to simplify this process. There are several ways to test the API:
  * Internal queries: One option is to send the request to your own system and verify that the query and response are formatted correctly and the match is accurate.
  * External queries: A second option is to query another matchmaker with [test data](testing/). In these cases, an additional property of the `"patient"` object should be specified: `"test" : true`. This informs the system being queried that the query is a test, allowing it to respond according. Specifically, the system being queried should suppress any automatic email notifications and include test data in the response (if it is normally hidden).
