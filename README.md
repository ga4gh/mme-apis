# Matchmaker Exchange API

[![DOI](https://zenodo.org/badge/doi/10.5281/zenodo.17235.svg)](http://dx.doi.org/10.5281/zenodo.17235)

The Matchmaker Exchange (MME) API provides a shared language that databases can use to query each other to find similar patients. The API is build around a standardized patient profile that includes both phenotype and genotype information. Implementing the API involves parsing a query profile, identifying cases in your database that are similar, and returning a list of profiles for those cases.

## Implementing the MME API

Version 1.0 (latest release): 
* API specification: [blob/v1.0a/search-api.md](https://github.com/ga4gh/mme-apis/blob/v1.0a/search-api.md)
* Publication in Human Mutation: [doi:10.1002/humu.22850](http://dx.doi.org/10.1002/humu.22850)

Development version (master branch):
* API specification: [search-api.md](search-api.md)

Have a question? Send us an email at api@matchmakerexchange.org or create a GitHub issue.

## How to connect to another endpoint
* To query most MME services, you need to request an authentication token from that service. You can find a more detailed description here: [join-protocol.md](join-protocol.md).
* To help securely share authentication tokens, the [KEYS](KEYS) file contains GPG keys for at least one member of each MME team.

## Endpoints
Below are a list of live implementations of the API, and the associated base URLs.

| Matchmaker | description | base URL | IP Address |
| ---------- | ----------- | -------- | ---------- |
| DECIPHER | production | https://decipher.sanger.ac.uk/mmapi/v1 | |
| GeneMatcher | staging | https://staging.phenodbmatcher.net/mmapi | 76.127.141.233 |
| GeneMatcher | production | https://genematcher.org/mmapi | 128.220.229.7 |
| Monarch | production | http://mme.monarchinitiative.org | |
| PhenomeCentral | production | https://phenomecentral.org/rest/remoteMatcher/ | |

## Open-source API implementations

Here are the open-source implementations of the MME API that we are aware of:

1. MME reference server: https://github.com/MatchmakerExchange/reference-server. This implementation provides a concrete but simple example of one method for performing phenotype and gene-level matchmaking using an elasticsearch index.
1. Monarch Initiative: https://github.com/monarch-initiative/monarch-mme. This is an adapter that implements the MME API on top of existing Monarch APIs.
1. PhenomeCentral: https://github.com/phenotips/remote-matching. This package extends [PhenoTips](http://phenotips.org) and PhenomeCentral to support the MME API. 
