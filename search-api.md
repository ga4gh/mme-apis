# OVERVIEW

**Submit patient matching request:**
`HTTP POST` to remote server: `<base_remote_url>/mmapi/v1/match`
For example: `https://yourmatchmaker.org/mmapi/v1/match`

**Receive asynchronous response:**
`HTTP POST` from remote server to: `<base_origin_url>/mmapi/v1/matchResults`
For example: `https://mymatchmaker.org/mmapi/v1/matchResults`

**Update previous request:**
`HTTP PUT` to remote server: `<base_remote_url>/mmapi/v1/match/<queryID>`
For example: `https://yourmatchmaker.org/mmapi/v1/match/a32fa90vd`

**Delete previous request:**
`HTTP DELETE` to remote server: `<base_remote_url>/mmapi/v1/match/<queryID>`
For example: `https://yourmatchmaker.org/mmapi/v1/match/a32fa90vd`

## Search Request

`HTTP POST` request to `<base_remote_url>/mmapi/v1/match`, with an `application/json` body with the following format:

### Example

```json
{
  "id" : <identifier>,
  "queryType" : "once"|"periodic",

  "contact": {
    "name": "Full Name",
    "href": <URL>
  },
  "label" : <identifier>,
  "gender" : "M"|"F",
  "ageOfOnset" : <HPO code>,
  "inheritanceMode" : <inheritance code>,

  "disorders" : [
    "MIM:######",
    "ORPHA#####",
    …
  ],
  "features" : [
    {
      "id" : <ICHPT or HPO code>,
      "observed" : "yes"|"no"|"unknown",
      "ageOfOnset" : "…"
    },
    …
  ],
  "genes" : [
    {
      "gene" : <gene name>|<ensembl gene ID>|<entrez gene ID>,
      "referenceName" : "1"|"2"|…|"X"|"Y",
      "start" : <number>,
      "end" : <number>,
      "referenceBases" : "A"|"ACG"|…,
      "alternateBases" : "A"|"ACG"|…,
      "zygosity" : <number>,
      "type" : <mutation type>,
      "assembly" : "NCBI36"|"GRCh37.p13"|"GRCh38.p1"|…
    },
    …
  ]
}
```

#### ID
* ***Mandatory***
* The internal identifier (obfuscated or not) that can be used by the originating system to reference the patient data.
* Transparent string, limited to 255 characters in utf-8.

#### Contact
* ***Mandatory***
* The contact information describes how the eventual recipient of the match response can contact the owner of the matched patient record to follow-up on the match. It contains two components, both required:
  1. A public (no login required) URL for contacting the owner of the patient record to follow up with a match. This must be a valid URL (of the form `<scheme>:<address>`), and could take a number of forms:
    * an `HTTP` URL: in this case, the URL could be a contact form which would allow the user to contact the owner of the matched patient.
    * a `mailto` URL: in this case, the URL could be a (potentially-anonymized) email address to contact regarding the patient match.
  1. The human-readable name of the clinician or organization that the user is contacting with the provided URL. A transparent string, limited to 255 characters in utf-8.

#### Label
* *Optional*
* A name/identifier assigned by the user which can be used to reference the patient in a recognizable manner (in an email for example); it should not contain any *personally identifiable information*.
* Transparent string, limited to 255 characters in utf-8.

#### Query type
* *Optional*
* Accepted values:
  * `once`: only search once in the current database
  * `periodic`: repeat the search monthly until canceled, reporting new and updated matches
* The default value is `once`
* If a system doesn’t support the requested type, the `once` behavior is used

#### Gender
* *Optional*
* Accepted values: `"M"`, `"F"`
* Any other value is treated as `"unknown"`

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
* Accepted values:
  * `ad` - Autosomal dominant
  * `ar` - Autosomal recessive
  * `xd` - X-linked dominant
  * `xr` - X-linked recessive
  * `yl` - Y-linked
  * `mi` - Mitochondrial
  * `ic` - Isolated cases
  * `un` - Uncertain

#### Disorders
* *Optional*
* Is a list of [OMIM](http://omim.org/) (`MIM:######`) or [OrphaNet](http://www.orphadata.org/) (`ORPHA#####`) identifiers, can be empty
* NOTE: we may want to support other sources later.

#### Features
* It is ***mandatory*** to have at least one of these two: `features`, `genes` (having both is preferred)
* Is a **list of features** described by:
  * `id`: an ICHPT or HPO term identifier
  * `observed`: `"yes"`|`"no"`|`"unknown"`
  * `ageOfOnset`: same as the global age of onset described above (*optional*; system which do not support this type of information per symptom should ignore it)
* More metadata can be later added to each feature if necessary.
* By default we shouldn’t sent any features with the `observed` status (or value) `"unknown"`

#### Genes
* It is ***mandatory*** to have at least one of these two: `features`, `genes` (having both is preferred)
* Is a **list of possible causes** described by:
  * `gene`:
    * `<gene symbol>` from the [HGNC database](http://www.genenames.org/) OR
    * `<ensembl gene ID>` OR
    * `<entrez gene ID>`
  * `referenceName`: `"1"`, `"2"`, …, `"22"`, `"X"`, `"Y"`; the chromosome this variant or gene is on
  * `start`: `<number>`; the start position of the variant. (0-based)
  * `end`: `<number>`; the end position of the variant. (0-based, exclusive)
      * **NOTE:** The location (`referenceName`, `start`, `end`) is *optional*
  * `referenceBases`: `"A"`|`"ACG"`|…, VCF-style reference of at least one base (*optional*)
  * `alternateBases`: `"A"`|`"ACG"`|…, VCF-style alternate allele of at least one base (*optional*)
  * `zygosity`: `<number>` (`1` for heterozygous or hemizygous, `2` for homozygous; *optional*)
  * `type`: the (*optional*) type of mutation, as a means to describe the broad category of cDNA effect predicted to result from a mutation to improve matchmaking, without disclosing the actual mutation:
    * `TRUNCATING` (e.g. stopgain, stoploss, startloss, frameshift indel)
    * `ALTERING` (e.g. missense, non-frameshift indel)
    * `SPLICING`
    * `UTR` (UTR3, UTR5)
    * `INTRONIC`
    * `PROXIMAL` (e.g. upstream, downstream)
    * `OTHER` (e.g. motif disruption, synonymous)
  * `assembly`: reference assembly identifier, including patch number if relevant, of the form: `<assembly>[.<patch>]` (***mandatory***)
    * example valid values: `"NCBI36"`, `"GRCh37"`, `"GRCh37.p13"`, `"GRCh38"`, `"GRCh38.p1"`
    * If the patch is not provided, the assembly is assumed to represent the initial (unpatched) release of that assembly.
* This should list either *candidate genes*, using the `gene` field with optionally other more specific fields, or precise *genomic variants*, specifying the assembly, the location (`referenceName`, `start`, `end`), and the reference and alternate bases

## Search Results Response
Either a synchronous `application/json` response to a `/match` request, an asynchronous `application/json` `HTTP POST` request to `<base_origin_url>/mmapi/v1/matchResults`, or a human-readable email sent to the user’s email address.

The response to the search request looks like:

### Example

```json
{
  "queryID" : <identifier>,
  "responseType" : "inline"|"asynchronous"|"email",
  "results" : [
    {
      "contact": {
        "name": "Full Name",
        "href": <URL>
      },
      "label" : <identifier>,
      "gender" : "M"|"F",
      "ageOfOnset" : <HPO code>,
      "inheritanceMode" : <inheritance code>,
      "disorders" : […],
      "features" : […],
      "genes" : […]
    },
    …
  ]
}
```

#### Query identifier
* ***Mandatory***
* Helps match the results to the original query for asynchronous results, and allows the submitter to manage the search submission
* This does not have to be the same as the id sent in the request since it represents how the remote host stores queries
* Transparent string, limited to 255 characters in utf-8.

#### Response type
* *Optional*
* `inline` responses are sent in the same response (the default value if the results property exists)
* `asynchronous` responses will be sent by the remote server at a later time, in a separate request to the origin server (the default value if the results property is missing)
* `email` responses will be sent by email directly to the contact email, in a human readable format

#### Results
* *Absent* for asynchronous results
* ***Mandatory*** for inline results, but can be empty
* Is a **list of matches**, where each match has the same format as the one described above for the query

### Asynchronous responses
* Are sent through a HTTPS request to the originating server
* Same format as the synchronous response, but placed in an array and wrapped in an object, so that multiple responses can be sent at the same time

```json
{"responses":
  [
    {
      "queryID" : <identifier>,
      "results" : […]
    },
    {
      "queryID" : <identifier>,
      "results" : […]
    },
    …
  ]
}
```

### Email responses
The format of email responses is not restricted, and is left up to each site to implement in a user-friendly way.

## Search Request Update
`HTTP PUT` request to `<base_remote_url>/mmapi/v1/match/<queryID>`, with an `application/json` body with the same format as a search request:

### Example

```json
{
  "id" : <identifier>,
  …
}
```

A search request update is exactly the same as the search request with two differences:

* The HTTP method used is a `PUT` (as opposed to a `POST`).
* The URL includes the `queryID` that was returned in the search result response when the search was originally submitted is required.
* The `id` has to match the original `id` in the search request.
* All other information in the search request is replaced with a search request update.

The search request update returns a search results response.

## Search Request Delete
`HTTP DELETE` request to `<base_remote_url>/mmapi/v1/match/<queryID>`, with an `application/json` body with the following format:

### Example

```json
{
  "id" : <identifier>
}
```

A search request delete:

* The HTTP method used is a `DELETE`.
* The URL includes the `queryID` that was returned in the search result response when the search was originally submitted is required.
* The `id` has to match the original `id` in the search request.

The search request delete returns an `OK (200)` status to indicate that the search was deleted, nothing more.
