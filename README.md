# SOMtoday REST API docs
## some misc
Endpoint for authentication: https://production.somtoday.nl

Endpoint for the API is returned when you fetch the access token

Always include the header "Accept" with the value of "application/json" so you won't get XML. (except if you want XML :-) ) (the authentication stuff always returns JSON)

you can do sample requests using curl, for example:
```bash
curl http://example.com/user/blah?active=true&limit=3 -d "key=value&otherkey=value" -H "AHeader: Value"
```

which will be listed here as

|Name    |Type  |Value|
|--------|------|-----|
|id      |URL   |blah |
|active  |Query |true |
|limit   |Query |3    |
|key     |Body  |value|
|otherkey|Body  |value|
|AHeader |Header|Value|

When there is a value that is unique to you (like username, password, or token), it will have a value like `[username]`

I don't recommend using curl in your programming language, except for PHP but even there it's a pain. There are much better libraries.

<details>
  <summary>A list of libraries for your language </summary>
   JavaScript: [window.fetch](https://developers.google.com/web/updates/2015/03/introduction-to-fetch)

   NodeJS: [node-fetch](https://github.com/bitinn/node-fetch), [HTTP from stdlib](https://nodejs.org/api/http.html), [Request](https://github.com/request/request), [Axios](https://github.com/axios/axios)

   Go: [net/http](https://golang.org/pkg/net/http/)

   Ruby: [Faraday](https://github.com/lostisland/faraday), [HTTParty](https://github.com/jnunemaker/httparty)

   Please add more if you know more.
</details>

## SOMtoday metadata
### Getting a list of schools: `GET https://servers.somtoday.nl`
Returns an array of schools
```json
[
  {
    "baseURL": "https://www.somtoday.nl/",
    "instellingen": [
      {
        "naam": "Abel Tasman",
        "afkorting": "luzacatc",
        "brin": "29ZP",
        "uuid": "a6088f01-7f39-41f0-94a5-78b100c1816f"
      },
      {
        "naam": "Almeerse scholengroep",
        "afkorting": "asg",
        "brin": "17DN",
        "uuid": "932cb2e1-bb3e-47dd-9a37-e5553e9c1b3c"
      },
      {
        "naam": "Altena College",
        "afkorting": "altena",
        "brin": "02XS",
        "uuid": "bb55c368-2823-4bfc-97b3-4a2cd98d2010"
      },
      ...
]
```

## Authentication / authorization
baseurl: https://production.somtoday.nl

### Fetching the access token: `POST baseurl/oauth2/token`

#### Parameters

|Name|Type|Value|
|----|----|-----|
|grant_type|Body|password|
|username|Body|[school uuid]\\[username]|
|password|Body|[password]|
|scope|Body|openid|
|Authorization|Header|Basic RDUwRTBDMDYtMzJEMS00QjQxLUExMzctQTlBODUwQzg5MkMyOnZEZFdkS3dQTmFQQ3loQ0RoYUNuTmV5ZHlMeFNHTkpY|

**Note: that authorization header is the result of this java code:**
```java
Base64.getEncoder().encodeToString(String.format("%s:%s", new Object[] { "D50E0C06-32D1-4B41-A137-A9A850C892C2" /* client ID */, "vDdWdKwPNaPCyhCDhaCnNeydyLxSGNJX" /* client secret */ }).getBytes(Charset.forName("UTF-8")))
```

#### Returns
```json
{
    "access_token": "<REDACTED>",
    "refresh_token": "<REDACTED>",
    "somtoday_api_url": "https://bonhoeffer-api.somtoday.nl",
    "scope": "openid",
    "somtoday_tenant": "bonhoeffer",
    "id_token": "<REDACTED>",
    "token_type": "Bearer",
    "expires_in": 3600
}
```

The `somtoday_api_url` is used for all non-authentication requests, like for getting grades.

token_type, scope and (probably) expires_in are always the same, the other values change depending on the user, and school (the tokens are of course randomly generated).

#### Example
```bash
school_uuid='4213a402-b898-4d16-9ebb-8c5f02b57474' username='450000@live.bc-enschede.nl' password='MYSECRETPASSWORD123'
curl "https://production.somtoday.nl/oauth2/token" -d "grant_type=password&username=$school_uuid\\$username&password=$password&scope=openid" -H "Authorization: Basic RDUwRTBDMDYtMzJEMS00QjQxLUExMzctQTlBODUwQzg5MkMyOnZEZFdkS3dQTmFQQ3loQ0RoYUNuTmV5ZHlMeFNHTkpY"
```

**Note: We use `\\` here, because `\` is normally used to escape things like quotes (e.g. `\"`) (and only bash double quote strings can escape using `\`), so `\\` will translate to `\`, and you can just use `\` if you use single quotes**

### Refreshing the token: `POST baseurl/oauth2/token`
#### Parameters

|Name|Type|Value|
|----|----|-----|
|grant_type|Body|refres_token|
|refresh_token|Body|[refresh_token]|
|client_id|Body|D50E0C06-32D1-4B41-A137-A9A850C892C2|
|client_secret|Body|vDdWdKwPNaPCyhCDhaCnNeydyLxSGNJX|

You get that `refresh_token` when you fetch the access token with the username and password.

Notice how here, we are not using the `Authorization` header which includes the `client_id` and `client_secret`, but we are just sending them in the HTTP Body.

#### Returns
```json
{
    "access_token": "<REDACTED>",
    "refresh_token": "<REDACTED>",
    "somtoday_api_url": "https://bonhoeffer-api.somtoday.nl",
    "scope": "openid",
    "somtoday_tenant": "bonhoeffer",
    "id_token": "<REDACTED>",
    "token_type": "Bearer",
    "expires_in": 3600
}
```

The `somtoday_api_url` is used for all non-authentication requests, like for getting grades.

token_type, scope and (probably) expires_in are always the same, the other values change depending on the user, and school (the tokens are of course randomly generated).

#### Example
```bash
token='<REDACTED>'
curl "https://production.somtoday.nl/oauth2/token" -d "grant_type=refresh_token&refresh_token=$token&client_id=D50E0C06-32D1-4B41-A137-A9A850C892C2&client_secret=vDdWdKwPNaPCyhCDhaCnNeydyLxSGNJX"
```

## Getting information
### Current student: GET /rest/v1/leerlingen
#### Headers
```
Authorization: "Bearer " + access_token
```

#### Returns
```
items array containing an object:
  $type (person type)
  links array containing an object:
    id (person ID, very important)
    rel (= "self")
    type (= $type)
    href (somtoday_api_url + "/rest/v1/leerlingen/" + id)
  permissions array containing an object:
    full
    type
    operations array containing one string
    instances array containing one string
  additionalObjects (= empty object afaik)
  leerlingnummer
  roepnaam
  achternaam
  email
  mobielNummer
  geboortedatum
  geslacht
```

### Grades: GET /rest/v1/resultaten/huidigVoorLeerling/{id}
#### URL parameters
{id} = id from /rest/v1/leerlingen

#### Headers
```
Authorization: "Bearer" + access_token
```
