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
The wiki contains an up-to-date list of active endpoints: [https://github.com/ga4gh/mme-apis/wiki/Endpoints](https://github.com/ga4gh/mme-apis/wiki/Endpoints)

## Open-source API implementations
The wiki contains a list of open-source implementations of the MME API: [https://github.com/ga4gh/mme-apis/wiki/Implementations](https://github.com/ga4gh/mme-apis/wiki/Implementations)
