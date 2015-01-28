# System Registration and Authentication

![protocol](https://cloud.githubusercontent.com/assets/88663/5034442/eeb1509a-6b3c-11e4-90e9-dfef879428cf.png)

Allowing an external site (**A**) to submit match requests to a local site (**B**) requires manual acceptance and configuration of the **A** system into the **B** system. Once **A** is accepted by **B**, then **all requests sent by A are to be considered trusted**, which means that **A** must ensure that no fraudulent requests are going to be sent from it.

It is not mandatory for **A** to automatically also accept **B**’s requests, this has to be negotiated between the two parties; the same protocol is then followed with **A** and **B** reversed.

**1.a.** When requesting access to **B**, the administrators of **A** must send to the administrators of **B**:
* a human readable **name** identifying **A**, to be presented to data owners on **B** when a search submitted by **A** matches patients in **B** and **B** wants to notify the data owners of a match (optional behavior that can be implemented by **B**)

**1.b.** Upon acceptance of **A** as a trusted source of queries, the administrators of **B** must respond with:
* a suggested human readable **name** and **description** identifying **B**, to be presented to users of **A** as a possible remote site to search; **A** could ignore these and use their preferred name and description, but for consistency across systems **B**’s preference should be used
* a **base URL** to be used for requests, including scheme (`https://`), hostname, eventual port, and path prefix (trailing `/` is optional); **A** will append `/match` to this path when sending queries
* an **authentication token** that must be used by **A** in the search requests

It is recommended to send this information in an encrypted email message, for example using PGP/GPG encryption with the recipient's public key. Plain text communication of the keys should be avoided as much as possible, and decrypted messages/keys should not be kept on personal computers.

For example:

```bash
# Generating a random key on A's administrator computer:
dd if=/dev/urandom count=1 2>/dev/null | shasum -a1 -b | cut '-d ' -f1 > key
# Sign and encrypt the key file to key.gpg
gpg --encrypt --sign --sign-with admin@a.org --recipient admin@b.org key
# Send key.gpg by email to admin@b.org

# Reading the key on B's administrator computer:
gpg --decrypt --output key key.gpg
# Now both systems have the same "key" file
```

**For data security, HTTPS is mandatory, with a valid, globally acceptable certificate!**

Since authentication tokens are the only means of identifying a site, this token should be unique to each remote site accepted by **B** to correctly determine where a query comes from. There is no imposed format for the token, except that it must be less than 255 characters long. A random 40-chars SHA1 key is recommended.

**2.a.** Configuring **B** to accept **A** as a trusted origin of remote searches requires that **B** stores somehow:
* the token that **B** expects in the requests from **A**
* the name that identifies **A**

**2.b.** Configuring **A** to support **B** as a possible destination of remote searches requires that **A** stores somehow:
* the URL prefix for HTTP requests sent to **B**
* the token that must be sent to **B**
* the name that identifies **B**
* the description of **B**

**3.** Every HTTP request that **A** submits to **B** must contain the token that **B** told **A** to use in HTTP header called 'X-Auth-Token'. For example:

    https://phenomecentral.org/rest/remoteMatcher/match
    
    POST /rest/remoteMatcher/match HTTP/1.1
	Host: phenomecentral.org
	Accept: application/vnd.ga4gh.matchmaker+json
	Content-Type: application/json; charset=UTF-8
	X-Auth-Token: 854a439d278df4283bf5498ab020336cdc416a7d

In this case:
* `https://phenomecentral.org/rest/remoteMatcher` is the base URL
* `/match` is the API method for submitting queries
* `854a439d278df4283bf5498ab020336cdc416a7d` is the authentication token that **B** told **A** to use in all match requests
* `X-Auth-Token: 854a439d278df4283bf5498ab020336cdc416a7d` would be in the Request Header sent to **B**

If the token is missing from a request, or if the token is not recognized, then the query is refused and a `401 Unauthorized` response is given.
