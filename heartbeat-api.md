# OVERVIEW

**Submit heartbeat request:**
`HTTP GET` to remote server: `<base_remote_url>/heartbeat`
For example: `https://yourmatchmaker.org/heartbeat`


## Heartbeat Request

`HTTP GET` request to `<base_remote_url>/heartbeat`

## Heartbeat Response
A synchronous `application/json` response, of the following form:

### Example

```
{
    "heartbeat": {
        "production": true,
        "version": "software release version",
        "accept": ["application/vnd.ga4gh.matchmaker.v1.0+json", "application/vnd.ga4gh.matchmaker.v1.1+json"]
    },
    "disclaimer": "...", 
}
```

#### Heartbeat
* ***Mandatory***, 
* Contains the heartbeat response.

##### Production
* ***Mandatory***
* Set to 'true' if the responding system is a production system, 'false' if not.

##### Version
* ***Mandatory***
* Internal software version of the responding system, for example GeneMatcher would return "6.2".

##### Accept
* ***Mandatory***
* Array of MME version mime types that responding system supports.

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

No content will be returned if the response code is anything other than 200.

