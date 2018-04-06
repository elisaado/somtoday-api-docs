# SOMtoday API docs, routes with parameters and more
## some misc
endpoint for auth: https://production.somtoday.nl

you can do sample requests using curl, for example:
```bash
curl <url> -d "key=value&otherkey=value" -H "Header-Title: Value"
```

which will be listed here as

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
