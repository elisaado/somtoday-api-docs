This guide is a step-by-step tutorial on how to authenticate mimicking the SOMToday app/webapp with Somtoday. It explains the details of the parameters you need to get an access token, how to generate a code challenge, and how to submit the authorization code and login credentials to Somtoday. If you prefer to use a different guide for fetching the access token via SOMtoday login, you have that option too. Moreover, this guide also covers how to retrieve information about your school, such as their name, UUID, and location. Plus, it teaches you how to refresh the access token you obtained during the authentication process by using the POST /oauth2/token endpoint.

---
## Getting a list of schools
<details><summary>Click to open</summary>


### Refreshing the access token `GET https://servers.somtoday.nl/organisaties.json`

Each object in the "instellingen" array represents an school and contains three values. The first is it's "uuid", which is a unique identifier for the school. The second value is "naam", which represents the name of the school. The third value pair is "plaats", which represents the location of the school.

In addition to these properties, each school can also have an array of "oidcurls" (Object Identifier Uniform Resource Locators). This array contains objects with three values: "omschrijving", "url", and "domain_hint". These properties provide additional information about the school's authentication systems (which are used when the school uses, for example, microsoft to authenticate it's students).

#### Returns
```json
[
  {
    "instellingen": [
      {
        "uuid": "099ce144-c400-4468-95d4-ad36f9f5cb5c",
        "naam": "Etty Hillesum Lyceum",
        "plaats": "DEVENTER",
        "oidcurls": [
          {
            "omschrijving":"Carmel",
            "url":"https://idpcluster.stichtingcarmelcollege.nl/nidp/oauth/nam",
            "domain_hint":""
          }
        ]
      },
      {
        "uuid": "ee8c456e-a227-4b7f-bb33-8601147d3264",
        "naam": "Scholengemeenschap Marianum",
        "plaats": "GROENLO",
        "oidcurls": []
      },
      {
        "uuid": "dda02c4c-82e5-42a7-a80d-bba133fd0430",
        "naam": "R.-K. Sg. Canisius",
        "plaats": "ALMELO",
        "oidcurls": []
      },
      ...
    ]
  }
]
```

</details>

## Authentication by mimicking the SOMToday app/webapp
<details><summary>Click to open the guide for authentication by mimicking SOMToday</summary>

### Step 1: Fetching the access token via Somtoday login: `GET https://inloggen.somtoday.nl/oauth2/authorize`

#### Parameters

| Name                  | Type      | Value                                |
|-----------------------|-----------|--------------------------------------|
| redirect_uri          | Parameter | somtodayleerling://oauth/callback    |
| client_id             | Parameter | D50E0C06-32D1-4B41-A137-A9A850C892C2 |
| state                 | Parameter | [state]                              |
| response_type         | Parameter | code                                 |
| scope                 | Parameter | openid                               |
| tenant_uuid           | Parameter | [tenant_uuid]                        |
| session               | Parameter | no_session                           |
| code_challenge        | Parameter | [code_challenge]                     |
| code_challenge_method | Parameter | S256                                 |

`redirect_uri`: This parameter is the URL that the user will be redirected to after authentication is completed. In this case, it's `somtodayleerling://oauth/callback`, which is a custom URI scheme that will launch the Somtoday Leerling app (or any other app that has this deeplink registered) on the user's device. I suspect that `somtodayouder://oauth/callback` will also work, since SOMToday has an parent version of thier app, but I'm not sure!

`state`: This parameter is used by the client to maintain state between the request and the callback. In this case, it's a randomly generated string of 8 characters.

`tenant_uuid`: This is a unique identifier for the used by SOMtoday to identify you as part of an school. This value can be found in the `uuid` property of the school object in the list of schools. This was explained above.

`code_challenge`: This parameter is used to prevent replay attacks by generating a unique value that is used to verify the client's identity when exchanging the authorization code for an access token.

```C#
    public void GenerateTokens()
    {
        string CodeVerifier = GenerateNonce();
        string CodeChallenge = GenerateCodeChallenge(CodeVerifier);
    }

    private static string GenerateNonce()
    {
        const string chars = "abcdefghijklmnopqrstuvwxyz123456789";
        var nonce = new char[128];
        for (int i = 0; i < nonce.Length; i++)
        {
            nonce[i] = chars[Random.Range(0, chars.Length)];
        }

        return new string(nonce);
    }

    private static string GenerateCodeChallenge(string codeVerifier)
    {
        using var sha256 = SHA256.Create();
        var hash = sha256.ComputeHash(Encoding.UTF8.GetBytes(codeVerifier));
        var b64Hash = Convert.ToBase64String(hash);
        var code = Regex.Replace(b64Hash, "\\+", "-");
        code = Regex.Replace(code, "\\/", "_");
        code = Regex.Replace(code, "=+$", "");
        return code;
    }
    
```

The GenerateNonce() function generates a 128-character string composed of lowercase letters and numbers.

The GenerateCodeChallenge() function first creates a SHA256 hash of the codeVerifier string using the SHA256.Create() method, and then encodes it as a base64 string using Convert.ToBase64String(). The resulting string is then modified to be safe for use in a URL by replacing certain characters with URL-safe equivalents using regular expressions.

#### Returns

This will return a redirect (HTTP 302), which will redirect the user. You need to intercept that redirect url and parse the query parameters. The `code` parameter is the authorization code that you need for the next parts of the authentication process, I will refer to this as the `authorization_code`.



### Step 2: Telling SOMToday who you are: `POST https://inloggen.somtoday.nl/?-1.-panel-signInForm`

#### Parameters

| Name                                                     | Type      | Value                        |
|----------------------------------------------------------|-----------|------------------------------|
| auth                                                     | Parameter | `authorization_code`         |
| loginLink                                                | body      | x                            |
| usernameFieldPanel:usernameFieldPanel_body:usernameField | body      | [username]                   |
| origin                                                   | header    | https://inloggen.somtoday.nl |

`auth`: This is the authorization code that you got from the previous step.

`username`: This is the username of the user that you want to authenticate.

#### Returns
This wil not return anything useful, you are sending your username to SOMToday, who will link you username to the `authorization_code` that you sent with the request.



### Step 3: Telling SOMToday your secret code ;): `POST https://inloggen.somtoday.nl/login?1-1.-passwordForm`

#### Parameters

| Name                                                     | Type      | Value                        |
|----------------------------------------------------------|-----------|------------------------------|
| auth                                                     | Parameter | `authorization_code`         |
| loginLink                                                | body      | x                            |
| passwordFieldPanel:passwordFieldPanel_body:passwordField | body      | [password]                   |
| origin                                                   | header    | https://inloggen.somtoday.nl |

`auth`: This is the authorization code that you got from step 1.

`password`: This is the password of the user that you want to authenticate.


#### Returns
A redirect (HTTP 302), you need to intercept this redirect and parse the query parameters. The `code` parameter is the authorization code that you need for the next parts of the authentication process, I will refer to this as the `final_authorization_code`.



### Step 4: Telling SOMToday that you are done: `POST https://inloggen.somtoday.nl/oauth2/token`

#### Parameters

| Name                  | Type      | Value                                |
|-----------------------|-----------|--------------------------------------|
| grant_type            | Parameter | authorization_code                   |
| session               | Parameter | no_session                           |
| scope                 | Parameter | openid                               |
| client_id             | Parameter | D50E0C06-32D1-4B41-A137-A9A850C892C2 |
| tenant_uuid           | Parameter | [tenant_uuid]                        |
| code                  | Parameter | `final_authorization_code`           |
| code_verifier         | Parameter | [code_verifier]                      |

`tentant_uuid`: This is a unique identifier for the used by SOMtoday to identify you as part of an school. This value can be found in the `uuid` property of the school object in the list of schools. This was explained above.

`final_authorization_code`: This is the authorization code that you got from step 3.

`code_verifier`: This is the code verifier that you generated in step 1.

#### Returns

```json
{
  "access_token": "<REDACTED>",
  "refresh_token": "<REDACTED>",
  "somtoday_api_url": "https://api.somtoday.nl",
  "scope": "openid",
  "somtoday_tenant": "bonhoeffer",
  "id_token": "<REDACTED>",
  "token_type": "Bearer",
  "expires_in": 3600
}
```
</details>

## Fetching the access token via SOMtoday login
<details><summary>Click to open the guide for fetching the access token via SOMtoday login</summary>

All routes here are prefixed with the base url: `https://somtoday.nl`

### Fetching the access token via Somtoday login: `POST /oauth2/token`

#### Parameters

| Name       | Type | Value                                |
|------------|------|--------------------------------------|
| grant_type | Body | password                             |
| username   | Body | [school uuid]\\[username]            |
| password   | Body | [password]                           |
| scope      | Body | openid                               |
| client_id  | Body | D50E0C06-32D1-4B41-A137-A9A850C892C2 |

**Note: Since April 1st of 2021, SOMToday started using a different OAuth2 implementation in their app (SSO). The requests used to contain a `client_secret`, along with the `client_id`, currently, only the `client_id` is needed. The documentation has been adapted accordingly. Thanks to everyone on Discord for giving me a heads up about this problem, and special thanks to @jktechs for figuring out that omitting the `client_secret` makes it work again.**

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
curl "https://somtoday.nl/oauth2/token" -d "grant_type=password&username=$school_uuid\\$username&password=$password&scope=openid&client_id=D50E0C06-32D1-4B41-A137-A9A850C892C2"
```

**Note: We use `\\` here, because `\` is normally used to escape things like quotes (e.g. `\"`) (and only bash double quote strings can escape using `\`), so `\\` will translate to `\`, and you can just use `\` if you use single quotes**



</details>

## Refreshing the access token
<details><summary>Click to open the guide for refreshing the access token</summary>


### Refreshing the access token `POST /oauth2/token`

#### Parameters

| Name          | Type | Value                                |
|---------------|------|--------------------------------------|
| grant_type    | Body | refresh_token                        |
| refresh_token | Body | [refresh_token]                      |
| client_id     | Body | D50E0C06-32D1-4B41-A137-A9A850C892C2 |
| scope         | Body | openid                               |

`refresh_token`: This is the refresh token that you get from the authentication process.

#### Returns

```json
{
  "access_token": "<REDACTED>",
  "refresh_token": "<REDACTED>",
  "somtoday_api_url": "https://api.somtoday.nl",
  "scope": "openid",
  "somtoday_tenant": "bonhoeffer",
  "id_token": "<REDACTED>",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

</details>
