# OVERVIEW

**Submit metrics request:**
`HTTP GET` to remote server: `<base_remote_url>/metrics`
For example: `https://yourmatchmaker.org/metrics`


## Metrics Request

`HTTP GET` request to `<base_remote_url>/metrics`

#### Security
* ***Mandatory***, 
* This is a protected resource and as such requires the [use of an API key to access](/join-protocol.md).

## Metrics Response
A synchronous `application/json` response, of the following form:

### Example

```
{
    "metrics": {
        "numberOfCases": 0,
        "numberOfSubmitters": 0,
        "numberOfGenes": 0,
        "numberOfUniqueGenes": 0,
        "numberOfVariants": 0,
        "numberOfUniqueVariants": 0,
        "numberOfFeatures": 0,
        "numberOfUniqueFeatures": 0,
        "numberOfFeatureSets": 0,
        "numberOfUniqueGenesMatched": 0,
        "numberOfCasesWithDiagnosis": 0,
        "numberOfRequestsReceived": 0,
        "numberOfPotentialMatchesSent": 0,
        "dateGenerated": "2017-08-24",
        
    },
    "disclaimer": "...", 
}
```

#### Metrics
* ***Mandatory***, 
* Contains the metrics response.

##### NumberOfCases
* ***Optional***
* The number of cases available for matching using MME in this node.

##### NumberOfSubmitters
* ***Optional***
* The number of PIs(or data-owners) who have submitted cases that are available for matching using MME in this node.

##### NumberOfGenes
* ***Optional***
* The number of genes that are available for matching using MME in this node. This includes duplicates.

##### NumberOfUniqueGenes
* ***Optional***
* The number of unique genes that are available for matching using MME in this node.

##### NumberOfVariants
* ***Optional***
* The number of variants that are available for matching using MME in this node. This includes duplicates.

##### NumberOfUniqueVariants
* ***Optional***
* The number of unique variants that are available for matching using MME in this node.

##### NumberOfFeatures
* ***Optional***
* The number of features that are available for matching using MME in this node. This includes duplicates.

##### NumberOfUniqueFeatures
* ***Optional***
* The number of unique features that are available for matching using MME in this node.

##### NumberOfFeatureSets
* ***Optional***
* The number of combinations of features that are available for matching using MME in this node. This includes duplicates.

##### NumberOfUniqueGenesMatched
* ***Optional***
* The number of unique genes that have been matched on in this node as the result of a request.

##### NumberOfCasesWithDiagnosis
* ***Optional***
* The number of cases which have a diagnosis (as specified in MME API fields) in this node.

##### NumberOfRequestsReceived
* ***Optional***
* The number of match requests received by this node.

##### NumberOfPotentialMatchesSent
* ***Optional***
* The number of potential match returned by this node as the result of all requests.

##### DateGenerated
* ***Optional***
* The date and potentially time at which these stats were generated. [ISO 8601 format](https://en.wikipedia.org/wiki/ISO_8601).

##### Disclaimer
* ***Optional***
* Optional disclaimer.

### Response Codes
Possible response codes are as follows:

| HTTP Status Code | Reason Phrase | Description
| ---------------- | -------- | -----------
| 200 | OK | no error |
| 401 | Unauthorized | invalid API key
| 500 | Internal Server Error | default error

No content or non json content may be returned if the response code is anything other than 200.






