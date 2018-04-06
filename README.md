# SOMtoday API docs, routes with parameters and more
## some misc
Endpoint for the auth: https://production.somtoday.nl

Endpoint for the API is returned by the login function route

Always include the header "Accept" with the value of "application/json" so you won't get XML

you can do sample requests using curl, for example:
```bash
curl aaaa/blah -d "key=value&otherkey=value" -H "Header-Title: Value"
```

which will be listed here as

URL parameters:
id = "blah"

POST body:
```
key = "value"
otherkey = "value"
```

Headers:
```
Header-Title: "Value"
```

I don't recommend using curl in your programming language, except for PHP but even there it's a pain. There are much better libraries.

<details> 
  <summary>A list of libraries for your language </summary>
   JavaScript (client side): [fetch()](https://developers.google.com/web/updates/2015/03/introduction-to-fetch)

   NodeJS: [HTTP from stdlib](https://nodejs.org/api/http.html), [Request](https://github.com/request/request), [Axios](https://github.com/axios/axios)
   
   Go: [net/http](https://golang.org/pkg/net/http/)
   
   Ruby: [Faraday](https://github.com/lostisland/faraday), [HTTParty](https://github.com/jnunemaker/httparty)

   Please add more if you know more.
</details>

## Authentication / authorization
### Logging in: POST /oauth2/token
#### POST body
```
grant_type = "password"
username = school uuid + "\\" + username # get your school uuid from https://servers.somtoday.nl
```

#### Headers
```
Authorization: "Basic RDUwRTBDMDYtMzJEMS00QjQxLUExMzctQTlBODUwQzg5MkMyOnZEZFdkS3dQTmFQQ3loQ0RoYUNuTmV5ZHlMeFNHTkpY"
```
that random string is the output of this java code:

```java
Base64.getEncoder().encodeToString(String.format("%s:%s", new Object[] { "D50E0C06-32D1-4B41-A137-A9A850C892C2" /* client ID */, "vDdWdKwPNaPCyhCDhaCnNeydyLxSGNJX" /* client secret */ }).getBytes(Charset.forName("UTF-8")))
```

#### Returns
access_token, refresh_token, id_token, expires_in (= 3600), token_type (= "Bearer"), somtoday_api_url, scope (= "openid")

### Refreshing the token: POST /oauth2/token
#### POST body
```
grant_type = "refresh_token"
client_id = "D50E0C06-32D1-4B41-A137-A9A850C892C2" # get this from the latest app if it doesn't work
client_secret = "vDdWdKwPNaPCyhCDhaCnNeydyLxSGNJX" # get this from the latest app if it doesn't work
refresh_token = refresh_token # the refresh_token is returned by the logging in route
```

#### Headers
None

#### Returns
access_token, refresh_token, id_token, expires_in (= 3600), token_type (= "Bearer"), somtoday_api_url, scope (= "openid")

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
