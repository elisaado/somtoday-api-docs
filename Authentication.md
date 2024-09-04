# SOMtoday Authentication Docs

---

This guide is a step-by-step tutorial on how to authenticate mimicking the SOMToday app/webapp with Somtoday. It explains the details of the parameters you need to get an access token, how to generate a code challenge, and how to submit the authorization code and login credentials to Somtoday. If you prefer to use a different guide for fetching the access token via SOMtoday login or via SSO, you have that option too. Moreover, this page also covers how to retrieve information about your school, such as their name, UUID, and location. Plus, it gives you information on how to refresh the access token you obtained during the authentication process by using the POST /oauth2/token endpoint.


---
<!-- TOC -->

- [SOMtoday Authentication Docs](#somtoday-authentication-docs)
  - [Getting a list of schools](#getting-a-list-of-schools)
  - [Authentication by mimicking the SOMToday app/webapp](#authentication-by-mimicking-the-somtoday-appwebapp)
    - [Fetching the access token via Somtoday login](#step-1-fetching-the-access-token-via-somtoday-login-get-httpsinloggensomtodaynloauth2authorize)
    - [Deciding if it is a username+password flow or an username first-flow](#step-2-deciding-if-it-is-a-usernamepassword-flow-or-an-username-first-flow)
    - [Telling SOMToday that you are done](#step-3-telling-somtoday-that-you-are-done-post-httpsinloggensomtodaynloauth2token)
  - [Authentication using SSO](#authentication-using-sso-single-sign-on)
  - [Fetching the access token via SOMtoday login](#fetching-the-access-token-via-somtoday-login-post-oauth2token)
  - [Refreshing the access token](#refreshing-the-access-token-post-oauth2token)
---
## Getting a list of schools
<details><summary>Click to open</summary>


### Getting a list of schools `GET https://servers.somtoday.nl/organisaties.json`

Each object in the "instellingen" array represents a school and contains three values. The first is it's "uuid", which is a unique identifier for the school. The second value is "naam", which represents the name of the school. The third value pair is "plaats", which represents the location of the school.

In addition to these properties, each school can also have an array of "oidcurls" (Object Identifier Uniform Resource Locators). This array contains objects with three values: "omschrijving", "url", and "domain_hint". These properties provide additional information about the school's authentication systems (which are used when the school uses, for example, microsoft to authenticate its students).

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

If you rather have a Postman example, you van view it here, but I recommend to still read through the documentation for the best understanding of the authentication process: 

[<img src="https://run.pstmn.io/button.svg" alt="Run In Postman" style="width: 128px; height: 32px;">](https://www.postman.com/red-equinox-452973/workspace/public-workspace/collection/19370875-46472ea9-9786-4cc0-87e0-f1f144f976cb?action=share&creator=19370875)

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

`redirect_uri`: This parameter is the URL that the user will be redirected to after authentication is completed. In this case, it's `somtodayleerling://oauth/callback`, which is a custom URI scheme that will launch the Somtoday Leerling app (or any other app that has this deeplink registered) on the user's device. I suspect that `somtodayouder://oauth/callback` will also work, since SOMToday has a parent version of their app, but I'm not sure!

`state`: This parameter is used by the client to maintain state between the request and the callback. In this case, it's a randomly generated string of 8 characters.

`tenant_uuid`: This is a unique identifier for the used by SOMtoday to identify you as part of a school. This value can be found in the `uuid` property of the school object in the list of schools. This was explained above.

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

When you're finished generating the link, it will look something like this: <br>
`https://inloggen.somtoday.nl/oauth2/authorize?redirect_uri=somtodayleerling://oauth/callback&client_id=D50E0C06-32D1-4B41-A137-A9A850C892C2&response_type=code&state=[8 random characters]&scope=openid&tenant_uuid=[UUID of the school]&session=no_session&code_challenge=[code challenge]k&code_challenge_method=S256`<br><br>

You are able to send the user to that generated link. They will see the usual SOMtoday login screen and will need to log into their account. When SOMtoday has authenticated them, SOMtoday will redirect them to a callback, which will look something like this:<br>
`somtodayleerling://oauth:443/callback?code=eyJ6aXAiOiJERUYiLCJjdHkiOiJKV1QiLCJlbmMiOiJBMjU2R0NNIiwiYWxnIjoiZGlyIn0....&iss=https://somtoday.nl&state=[8 random characters, same as first URL]`<br>
The `code` parameter is the access token that you are looking for. But this is only handy when working with native apps (like the SOMtoday app). The 'url' above is a deeplink that will open an installed app, hence the `somtodayleerling://` scheme. You can use this to open the SOMtoday app (or any apps with the scheme registered) on the user's device, and the app will handle the rest of the authentication process. <br><br>
#### Returns
This will return a redirect (HTTP 302), which will redirect the user. You need to intercept that redirect url and parse the query parameters. The `code` parameter is the authorization code that you need for the next parts of the authentication process, I will refer to this as the `authorization_code`. <br><br>
You also need to save the following cookie: 'production-authenticator-stickiness' which is a cookie value that is used to keep the user logged in. This cookie is used in the next steps of the authentication process. It looks something like this: `"5929c2cf25fe1a95"`<br><br>
And at last, you need to save the `location` header, which is the url that the user is redirected to. This url is used in the next steps of the authentication process. It looks something like this: `https://inloggen.somtoday.nl/?auth=`

### Step 1.1: Getting a cookie (~~for santa~~): `GET https://inloggen.somtoday.nl/`

| Name                                | Type      | Value                                 |
|-------------------------------------|-----------|---------------------------------------|
| auth                                | Parameter | `authorization_code`                  |
| production-authenticator-stickiness | cookie    | [production-authenticator-stickiness] |


This will return another cookie that you need to save: `JSESSIONID`. To make sure you can save the cookie I recommend to disallow the HTTP request to follow redirects.


### Step 2: Deciding if it is a username+password flow or an username first-flow

When logging in somtoday, there'll be 2 options on how to send your username and password.

username+password flow: After entering your school and pressing submit, there'll appear 2 input fields for username & password.

username first-flow: After entering your school and pressing submit, there'll appear only 1 input field for the username.

To decide which flow it is, we'll have to send one request.

```
POST /0-1.-panel-signInForm&auth=<authorization_code>
Origin: https://inloggen.somtoday.nl
```
#### Returns

Don't follow the redirect, check if the ``auth`` parameter exists in the 'Location' header. If exists, then it is a **username + password flow**, otherwise it is an **username first-flow**

#### Authentication with username first-flow: `POST https://inloggen.somtoday.nl/login?2-1.-passwordForm`

#### Parameters

| Name                                                     | Type      | Value                                 |
|----------------------------------------------------------|-----------|---------------------------------------|
| loginLink                                                | body      | x                                     |
| passwordFieldPanel:passwordFieldPanel_body:passwordField | body      | [password]                            |
| origin                                                   | header    | https://inloggen.somtoday.nl          |
| JSESSIONID                                               | cookie    | [JSESSIONID]                          |
| auth                                                     | param     | `authorization_code`
| production-authenticator-stickiness                      | cookie    | [production-authenticator-stickiness] |

`auth`: This is the authorization code that you got from step 1.

`password`: This is the password of the user that you want to authenticate.


#### Returns
A redirect (HTTP 302), you need to intercept this redirect and parse the query parameters. The `code` parameter is the authorization code that you need for the next parts of the authentication process, I will refer to this as the `final_authorization_code`.

#### Authentication with username + password flow `POST https://inloggen.somtoday.nl/?0-1.-panel-signInForm`

| Name                                                     | Type      | Value                                 |
|----------------------------------------------------------|-----------|---------------------------------------|
| loginLink                                                | body      | x                                     |
| usernameFieldPanel:usernameFieldPanel_body:usernameField | body      | [username]
| passwordFieldPanel:passwordFieldPanel_body:passwordField | body      | [password]                            |
| origin                                                   | header    | https://inloggen.somtoday.nl          |
| auth                                                     | param     | `authorization_code`
| JSESSIONID                                               | cookie    | [JSESSIONID]                          |
| production-authenticator-stickiness                      | cookie    | [production-authenticator-stickiness] |


#### Returns
A redirect (HTTP 302), you need to intercept this redirect and parse the query parameters. The `code` parameter is the authorization code that you need for the next parts of the authentication process, I will refer to this as the `final_authorization_code`.

### Step 3: Telling SOMToday that you are done: `POST https://inloggen.somtoday.nl/oauth2/token`

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

`tentant_uuid`: This is a unique identifier for the used by SOMtoday to identify you as part of a school. This value can be found in the `uuid` property of the school object in the list of schools. This was explained above.

`final_authorization_code`: This is the authorization code that you got from step 2.

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

## Authentication using SSO (single sign on)
<details><summary>Click to open the guide for authentication using SSO</summary>

### Fetching the access token via SSO: `POST /oauth2/token`

#### Parameters

| Name          | Type | Value                                |
|---------------|------|--------------------------------------|
| grant_type    | Body | authorization_code                   |
| redirect_uri  | Body | [redirect_uri]                       |
| code_verifier | Body | [code_verifier]                      |
| code          | Body | [code]                               |
| scope         | Body | openid                               |
| client_id     | Body | D50E0C06-32D1-4B41-A137-A9A850C892C2 |

`redirect_uri` is the link redirected to after the user logged in. (Must be the same as in the login link and one of a few specified values. An example is: `somtodayleerling://oauth/callback`)
`code_verifier` is the string that was encoded and send in the login link. (Must be the same as in the login link when encoded using the method specified in the login link)
`code` is the code that has been sent to the redirect uri. it is a JWT token (5 base64 url encoded blocks separated by '.')

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
redirect_uri='somtodayleerling://oauth/callback' code_verifier='SOME_BASE64_CODE' code='SOME_TOKEN'
curl "https://somtoday.nl/oauth2/token" -d "grant_type=authorization_code&redirect_uri=$redirect_uri&code_verifier=$code_verifier&code=$code&scope=openid&client_id=D50E0C06-32D1-4B41-A137-A9A850C892C2"
```

#### Code verifier and challenge

To generate the verifier you need to generate a random 32-byte url encoded base64 value and use some algorithm to encode it. I would advise to use sha256. Here is a node.js example.

```javascript
// source: https://auth0.com/docs/authorization/flows/call-your-api-using-the-authorization-code-flow-with-pkce#create-code-challenge
// Dependency: Node.js crypto module
// https://nodejs.org/api/crypto.html#crypto_crypto
function base64URLEncode(str) {
    return str.toString('base64')
        .replace(/\+/g, '-')
        .replace(/\//g, '_')
        .replace(/=/g, '');
}
var verifier = base64URLEncode(crypto.randomBytes(32));
function sha256(buffer) {
    return crypto.createHash('sha256').update(buffer).digest();
}
var challenge = base64URLEncode(sha256(verifier));
console.log(verifier)
console.log(challenge)
```

### The Url format

The url that the client has to visit to get a login window is `https://somtoday.nl/oauth2/authorize`.
These are the parameters:

| Name                  | Type | Value                                |
|-----------------------|------|--------------------------------------|
| response_type         | Body | code                                 |
| redirect_uri          | Body | [uri]                                |
| code_challenge        | Body | [code_challenge]                     |
| tenant_uuid           | Body | [tenant_uuid]                        |
| oidc_iss              | Body | [oidc_iss]                           |
| code_challenge_method | Body | [code_challenge_method]              |
| (state)               | Body | [custom_state]                       |
| prompt                | Body | login                                |
| scope                 | Body | openid                               |
| client_id             | Body | D50E0C06-32D1-4B41-A137-A9A850C892C2 |

`uri` and `code_challenge` have been described already.
`tenant_uuid` and `oidc_iss` can be found in the organisaties.json inside oidcurls
`code_challenge_method` is the method used to encode the `code_verifier`. It is highly advised to use 'S256' which stands for Sha256.
`state` is an optional parameter.
`custom_state` will be included in the callback and can be used for identification while fetching multiple tokens.

After the user has logged in the page will redirect to the `uri` with these parameters

| Name    | Type | Value               |
|---------|------|---------------------|
| code    | Body | [code]              |
| (state) | Body | [custom_state]      |
| iss     | Body | https://somtoday.nl |

`custom_state` is the previously defined value.
`code` has already been described

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

**Note: Since April 1st of 2021, SOMToday started using a different OAuth2 implementation in their app (SSO). The requests used to contain a `client_secret`, along with the `client_id`, currently, only the `client_id` is needed. The documentation has been adapted accordingly. Thanks to everyone on Discord for giving me a heads-up about this problem, and special thanks to @jktechs for figuring out that omitting the `client_secret` makes it work again.**

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
