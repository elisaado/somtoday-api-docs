# SOMtoday REST API docs

#### Discord

[![Discord Chat](https://img.shields.io/discord/789249810032361502.svg)](https://discord.gg/yE3e3erCut)

## Table of contents

<!-- TOC -->

- [SOMtoday REST API docs](#somtoday-rest-api-docs)
  - [Discord](#discord)
  - [Table of contents](#table-of-contents)
  - [Some miscellaneous stuff](#some-miscellaneous-stuff)
  - [SOMtoday metadata](#somtoday-metadata)
    - [Getting a list of schools: `GET https://servers.somtoday.nl/organisaties.json`](#getting-a-list-of-schools-get-httpsserverssomtodaynlorganisatiesjson)
  - [Authentication / authorization](#authentication--authorization)
    - [Fetching the access token via Somtoday login: `POST /oauth2/token`](#fetching-the-access-token-via-somtoday-login-post-oauth2token)
      - [Parameters](#parameters)
      - [Returns](#returns)
      - [Example](#example)
    - [Refreshing the token: `POST /oauth2/token`](#refreshing-the-token-post-oauth2token)
      - [Parameters](#parameters-1)
      - [Returns](#returns-1)
      - [Example](#example-1)
    - [Fetching the access token via SSO: `POST /oauth2/token`](#fetching-the-access-token-via-sso-post-oauth2token)
      - [Parameters](#parameters-2)
      - [Returns](#returns-2)
      - [Example](#example-2)
      - [Example](#example-2)
      - [The Url format](#the-url-format)
  - [Fetching information](#fetching-information)
    - [Current student(s): `GET /rest/v1/leerlingen`](#current-students-get-restv1leerlingen)
      - [Parameters](#parameters-3)
      - [Returns](#returns-3)
      - [Example](#example-3)
    - [Student by ID: `GET /rest/v1/leerlingen/[id]`](#student-by-id-get-restv1leerlingenid)
      - [Parameters](#parameters-4)
      - [Returns](#returns-4)
      - [Example](#example-4)
    - [Grades: `GET /rest/v1/resultaten/huidigVoorLeerling/[id]`](#grades-get-restv1resultatenhuidigvoorleerlingid)
      - [Parameters](#parameters-5)
      - [Returns](#returns-5)
    - [Schedule: `GET /rest/v1/afspraken`](#schedule-get-restv1afspraken)
      - [Parameters](#parameters-6)
      - [Returns](#returns-6)
      - [Example](#example-5)
    - [Absence Reports: `GET /rest/v1/absentiemeldingen`](#absence-reports-get-restv1absentiemeldingen)
      - [Parameters](#parameters-7)
      - [Returns](#returns-7)
    - [Study Guides: `GET /rest/v1/studiewijzers`](#study-guides-get-restv1studiewijzers)
      - [Parameters](#parameters-8)
      - [Returns](#returns-8)

<!-- /TOC -->

## Some miscellaneous stuff

Endpoint for authentication: https://somtoday.nl

Endpoint for the API is returned when you fetch the access token

Always include the header "Accept" with the value of "application/json" so you won't get XML. (except if you want XML :-) ) (the authentication stuff always returns JSON)

you can do sample requests using curl, for example:

```bash
curl http://example.com/user/blah?active=true&limit=3 -d "key=value&otherkey=value" -H "AHeader: Value"
```

which will be listed here as

| Name     | Type   | Value |
| -------- | ------ | ----- |
| id       | URL    | blah  |
| active   | Query  | true  |
| limit    | Query  | 3     |
| key      | Body   | value |
| otherkey | Body   | value |
| AHeader  | Header | Value |

When there is a value that is unique to you (like username, password, or token), it will have a value like `[username]`

I don't recommend using curl in your programming language, except for PHP but even there it's a pain. There are much better libraries.

<details>
  <summary>A list of libraries for your language </summary>
   JavaScript: [window.fetch](https://developers.google.com/web/updates/2015/03/introduction-to-fetch)

NodeJS: [node-fetch](https://github.com/bitinn/node-fetch), [HTTP from stdlib](https://nodejs.org/api/http.html), [Request](https://github.com/request/request), [Axios](https://github.com/axios/axios)

Go: [net/http](https://golang.org/pkg/net/http/)

Ruby: [Faraday](https://github.com/lostisland/faraday), [HTTParty](https://github.com/jnunemaker/httparty)

Python: [requests](http://docs.python-requests.org/en/master/)

Please add more if you know more.

</details>

## SOMtoday metadata

### Getting a list of schools: `GET https://servers.somtoday.nl/organisaties.json`

Returns an array of schools

```json
[
  {
    "instellingen": [
      {
        "uuid": "099ce144-c400-4468-95d4-ad36f9f5cb5c",
        "naam": "Etty Hillesum Lyceum",
        "plaats": "DEVENTER"
      },
      {
        "uuid": "ee8c456e-a227-4b7f-bb33-8601147d3264",
        "naam": "Scholengemeenschap Marianum",
        "plaats": "GROENLO"
      },
      {
        "uuid": "dda02c4c-82e5-42a7-a80d-bba133fd0430",
        "naam": "R.-K. Sg. Canisius",
        "plaats": "ALMELO"
      },
      ...
    ]
  }
]
```

## Authentication / authorization

baseurl: https://somtoday.nl

All routes here are prefixed with that baseurl.

### Fetching the access token via Somtoday login: `POST /oauth2/token`

#### Parameters

| Name       | Type | Value                                |
| ---------- | ---- | ------------------------------------ |
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

### Refreshing the token: `POST /oauth2/token`

#### Parameters

| Name          | Type | Value                                |
| ------------- | ---- | ------------------------------------ |
| grant_type    | Body | refresh_token                        |
| refresh_token | Body | [refresh_token]                      |
| client_id     | Body | D50E0C06-32D1-4B41-A137-A9A850C892C2 |
| client_secret | Body | vDdWdKwPNaPCyhCDhaCnNeydyLxSGNJX     |

**Note: Since April 1st of 2021, SOMToday started using a different OAuth2 implementation in their app (SSO). The requests used to contain a `client_secret`, along with the `client_id`, currently, only the `client_id` is needed. The documentation has been adapted accordingly. Thanks to everyone on Discord for giving me a heads up about this problem, and special thanks to @jktechs for figuring out that omitting the `client_secret` makes it work again.**

You get the `refresh_token` when you fetch the access token with the username and password.

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

This example uses the `client_id` and `client_secret` in body method of authorization.

```bash
token='<REDACTED>'
curl "https://somtoday.nl/oauth2/token" -d "grant_type=refresh_token&refresh_token=$token&client_id=D50E0C06-32D1-4B41-A137-A9A850C892C2&client_secret=vDdWdKwPNaPCyhCDhaCnNeydyLxSGNJX"
```
### Fetching the access token via SSO: `POST /oauth2/token`

#### Parameters

| Name          | Type | Value                                |
| ------------- | ---- | ------------------------------------ |
| grant_type    | Body | authorization_code                   |
| redirect_uri  | Body | [redirect_uri]                       |
| code_verifier | Body | [code_verifier]                      |
| code          | Body | [code]                               |
| scope         | Body | openid                               |
| client_id     | Body | D50E0C06-32D1-4B41-A137-A9A850C892C2 |

`redirect_uri` is the link redirected to after the user logged in. (Must be the same as in the login link and one of a few specified values. An example is: `somtodayleerling://oauth/callback`)
`code_verifier` is the string that was encoded and send in the login link. (Must be the same as in the login link when encoded using the method specified in the login link)
`code` is the code that has been send to the redirect uri. it is a JWT token (5 base64 url encoded blocks sepperated by '.')

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

To generate the verifier you need to generate a random 32-byte url encoced base64 value and use some algorithm to encode it. I would advise to use sha256. Here is a node.js example.

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
| --------------------- | ---- | ------------------------------------ |
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
`code_challenge_method` is the method used to encode the `code_verifier`. It is highly advised to use 'S256' wich stands for Sha256.
`state` is an optional parameter.
`custom_state` will be included in the callback and can be used for identification while fetching multiple tokens.

After the user has logged in the page will redirect to the `uri` with these paramaters

| Name           | Type | Value        |
| ------- | ---- | ------------------- |
| code    | Body | [code]              |
| (state) | Body | [custom_state]      |
| iss     | Body | https://somtoday.nl |

`custom_state` is the previously defined value.
`code` has already been described

## Fetching information

baseurl: returned when you fetch a token (`somtoday_api_url`), usually [lowercase snakecased schoolname]-api.somtoday.nl

All routes here are prefixed with that baseurl.

### Current student(s): `GET /rest/v1/leerlingen`

This REST method might return multiple students (I cannot test), since it says /leerlingen (Dutch plural for student).

I suppose it returns all students the current user has access to (so if a school administrator runs it, it will return all students on the school).

#### Parameters

| Name          | Type   | Value                 |
| ------------- | ------ | --------------------- |
| Authorization | Header | Bearer [access_token] |

#### Returns

```json
{
  "items": [
    {
      "$type": "leerling.RLeerling",
      "links": [
        {
          "id": 1234,
          "rel": "self",
          "type": "leerling.RLeerling",
          "href": "https://bonhoeffer-api.somtoday.nl/rest/v1/leerlingen/1234"
        }
      ],
      "permissions": [
        {
          "full": "leerling.RLeerlingPrimer:READ:INSTANCE(1234)",
          "type": "leerling.RLeerlingPrimer",
          "operations": ["READ"],
          "instances": ["INSTANCE(1234)"]
        }
      ],
      "additionalObjects": {},
      "leerlingnummer": 450000,
      "roepnaam": "Eli",
      "achternaam": "Saado",
      "email": "450000@live.bc-enschede.nl",
      "mobielNummer": "06-00000000",
      "geboortedatum": "2000-00-00",
      "geslacht": "Man"
    }
  ]
}
```

#### Example

```bash
token='<REDACTED>' school_url=https://bonhoeffer-api.somtoday.nl
curl "$school_url/rest/v1/leerlingen" -H "Authorization: Bearer $token" -H "Accept: application/json"
```

---

### Student by ID: `GET /rest/v1/leerlingen/[id]`

#### Parameters

| Name          | Type   | Value                 |
| ------------- | ------ | --------------------- |
| id            | URL    | [user id]             |
| Authorization | Header | Bearer [access_token] |

#### Returns

```json
{
  "links": [
    {
      "id": 1234,
      "rel": "self",
      "type": "leerling.RLeerling",
      "href": "https://bonhoeffer-api.somtoday.nl/rest/v1/leerlingen/1234"
    }
  ],
  "permissions": [
    {
      "full": "leerling.RLeerlingPrimer:READ:INSTANCE(1234)",
      "type": "leerling.RLeerlingPrimer",
      "operations": ["READ"],
      "instances": ["INSTANCE(1234)"]
    }
  ],
  "additionalObjects": {},
  "leerlingnummer": 450000,
  "roepnaam": "Eli",
  "achternaam": "Saado",
  "email": "450000@live.bc-enschede.nl",
  "mobielNummer": "06-00000000",
  "geboortedatum": "2000-00-00",
  "geslacht": "Man"
}
```

#### Example

```bash
token='<REDACTED>' school_url=https://bonhoeffer-api.somtoday.nl id=1234
curl "$school_url/rest/v1/leerlingen/$id" -H "Authorization: Bearer $token" -H "Accept: application/json"
```

---

### Grades: `GET /rest/v1/resultaten/huidigVoorLeerling/[id]`

Fetches the grades of the student. Note that all average grades are also grade items returned by the API. There are the different types of columns: the `type` property in the json (e.g. 'Toetskolom', 'ToetssoortGemiddeldeKolom').

#### Parameters

| Name          | Type      | Value                           |
| ------------- | --------- | ------------------------------- |
| id            | URL       | [user id]                       |
| Authorization | Header    | Bearer [access_token]           |
| Range         | Header    | items=[LowerBound]-[UpperBound] |
| additional    | Parameter | berekendRapportCijfer           |
| additional    | Parameter | samengesteldeToetskolomId       |
| additional    | Parameter | resultaatkolomId                |
| additional    | Parameter | cijferkolomId                   |
| additional    | Parameter | toetssoortnaam                  |
| additional    | Parameter | huidigeAnderVakKolommen         |

These LowerBound and UpperBound values are the amount of grades you want to request (the API uses pagination here). The value may not exceed 100, so the way to request **all** grades is by doing the following:

1. Request 0-99
2. Request 100-199
3. Request 200-299
4. Request .00-.99
5. Continue until the response contains less than 99 records
6. Profit!

#### Returns

```js
{
"items": [
    {
        "$type": "resultaten.RResultaat",
        "links": [
            {
                "id": 1234,
                "rel": "self",
                "type": "resultaten.RResultaat",
                "href": "https://api.somtoday.nl/rest/v1/resultaten/1234"
            }
        ],
        "permissions": [
            {
                "full": "resultaten.RResultaat:READ:INSTANCE(<REDACTED>)",
                "type": "resultaten.RResultaat",
                "operations": [
                    "READ"
                ],
                "instances": [
                    "INSTANCE(<REDACTED>)"
                ]
            }
        ],
        "additionalObjects": {},
        "herkansingstype": "Geen",
        "resultaat": "7.9",
        "geldendResultaat": "7.9",
        "datumInvoer": "2019-09-10T13:41:11.805+02:00",
        "teltNietmee": false,
        "toetsNietGemaakt": false,
        "leerjaar": 0,
        "periode": 0,
        "examenWeging": 0,
        "isExamendossierResultaat": true,
        "isVoortgangsdossierResultaat": false,
        "type": "ToetssoortGemiddeldeKolom",
        "vak": {
            "links": [
                {
                    "id": 1234,
                    "rel": "self",
                    "type": "onderwijsinrichting.RVak",
                    "href": "https://api.somtoday.nl/rest/v1/vakken/1234"
                }
            ],
            "permissions": [
                {
                    "full": "onderwijsinrichting.RVak:READ:INSTANCE(<REDACTED>)",
                    "type": "onderwijsinrichting.RVak",
                    "operations": [
                        "READ"
                    ],
                    "instances": [
                        "INSTANCE(<REDACTED>)"
                    ]
                }
            ],
            "additionalObjects": {},
            "afkorting": "ckv",
            "naam": "culturele en kunstzinnige vorming"
        },
        "leerling": {
            "links": [
                {
                    "id": 1234,
                    "rel": "self",
                    "type": "leerling.RLeerlingPrimer",
                    "href": "https://api.somtoday.nl/rest/v1/leerlingen/1234"
                }
            ],
            "permissions": [
                {
                    "full": "leerling.RLeerlingPrimer:READ:INSTANCE(<REDACTED>)",
                    "type": "leerling.RLeerlingPrimer",
                    "operations": [
                        "READ"
                    ],
                    "instances": [
                        "INSTANCE(<REDACTED>)"
                    ]
                }
            ],
            "additionalObjects": {},
            "UUID": "070dabd4-3449-4af3-8c38-788faac283a3",
            "leerlingnummer": 1234,
            "roepnaam": "<REDACTED>",
            "voorvoegsel": "<REDACTED>",
            "achternaam": "<REDACTED>"
        }
    },
    ...
}
```

---

### Schedule: `GET /rest/v1/afspraken`

Fetch the appointments from the schedule of the student.

#### Parameters

| Name          | Type      | Value                 |
| ------------- | --------- | --------------------- |
| Authorization | Header    | Bearer [access_token] |
| sort          | Parameter | asc-id                |
| additional    | Parameter | vak                   |
| additional    | Parameter | docentAfkortingen     |
| additional    | Parameter | leerlingen            |
| begindatum    | Parameter | yyyy-MM-dd            |
| einddatum     | Parameter | yyyy-MM-dd            |

#### Returns

```json
{
  "items": [
    {
      "$type": "participatie.RAfspraak",
      "links": [
        {
          "id": 8849104409,
          "rel": "self",
          "type": "participatie.RAfspraak",
          "href": "AFSPRAAK_URL"
        }
      ],
      "permissions": [
        {
          "full": "participatie.RAfspraak:READ:INSTANCE(8849104409)",
          "type": "participatie.RAfspraak",
          "operations": ["READ"],
          "instances": ["INSTANCE(8849104409)"]
        }
      ],
      "additionalObjects": {
        "vak": {
          "$type": "onderwijsinrichting.RVak",
          "links": [
            {
              "id": 126211284,
              "rel": "self",
              "type": "onderwijsinrichting.RVak",
              "href": "VAK_URL"
            }
          ],
          "permissions": [
            {
              "full": "onderwijsinrichting.RVak:READ:INSTANCE(126211284)",
              "type": "onderwijsinrichting.RVak",
              "operations": ["READ"],
              "instances": ["INSTANCE(126211284)"]
            }
          ],
          "additionalObjects": {},
          "afkorting": "wisB",
          "naam": "wiskunde B"
        },
        "docentAfkortingen": "Stk",
        "leerlingen": {
          "$type": "LinkableWrapper",
          "items": [
            {
              "$type": "leerling.RLeerlingPrimer",
              "links": [
                {
                  "id": 546308480,
                  "rel": "self",
                  "type": "leerling.RLeerlingPrimer",
                  "href": "LEERLING_URL"
                }
              ],
              "permissions": [
                {
                  "full": "leerling.RLeerlingPrimer:READ:INSTANCE(546308480)",
                  "type": "leerling.RLeerlingPrimer",
                  "operations": ["READ"],
                  "instances": ["INSTANCE(546308480)"]
                }
              ],
              "additionalObjects": {},
              "UUID": "UUID",
              "leerlingnummer": 119371,
              "roepnaam": "Christos",
              "achternaam": "Karapasias"
            }
          ]
        }
      },
      "afspraakType": {
        "links": [
          {
            "id": 144662674,
            "rel": "self",
            "type": "participatie.RAfspraakType",
            "href": "AFSPRAAK_TYPE_URL"
          }
        ],
        "permissions": [
          {
            "full": "participatie.RAfspraakType:READ:INSTANCE(144662674)",
            "type": "participatie.RAfspraakType",
            "operations": ["READ"],
            "instances": ["INSTANCE(144662674)"]
          }
        ],
        "additionalObjects": {},
        "naam": "Les",
        "omschrijving": "Les",
        "standaardKleur": -2394583,
        "categorie": "Rooster",
        "activiteit": "Verplicht",
        "percentageIIVO": 0,
        "presentieRegistratieDefault": true,
        "actief": true,
        "vestiging": {
          "$type": "instelling.RVestiging",
          "links": [
            {
              "id": 126208855,
              "rel": "self",
              "type": "instelling.RVestiging",
              "href": "VESTIGING_URL"
            }
          ],
          "permissions": [
            {
              "full": "instelling.RVestiging:READ:INSTANCE(126208855)",
              "type": "instelling.RVestiging",
              "operations": ["READ"],
              "instances": ["INSTANCE(126208855)"]
            }
          ],
          "additionalObjects": {},
          "naam": "Fortes Lyceum"
        }
      },
      "locatie": "217",
      "beginDatumTijd": "2020-05-04T11:15:00.000+02:00",
      "eindDatumTijd": "2020-05-04T12:00:00.000+02:00",
      "beginLesuur": 4,
      "eindLesuur": 4,
      "titel": "217 - A5wisB_2 - Stk",
      "omschrijving": "217 - A5wisB_2 - Stk",
      "presentieRegistratieVerplicht": true,
      "presentieRegistratieVerwerkt": false,
      "afspraakStatus": "ACTIEF",
      "vestiging": {
        "links": [
          {
            "id": 126208855,
            "rel": "self",
            "type": "instelling.RVestiging",
            "href": "VESTIGING_URL"
          }
        ],
        "permissions": [
          {
            "full": "instelling.RVestiging:READ:INSTANCE(126208855)",
            "type": "instelling.RVestiging",
            "operations": ["READ"],
            "instances": ["INSTANCE(126208855)"]
          }
        ],
        "additionalObjects": {},
        "naam": "SCHOOL_NAAM"
      }
    }
  ]
}
```

#### Example

```bash
curl "$school_url/rest/v1/afspraken?sort=asc-id&additional=vak&additional=docentAfkortingen&additional=leerlingen&begindatum=2020-05-01&einddatum=2020-05-19" -H "Authorization: Bearer $token" -H "Accept: application/json"
```

---

### Absence Reports: `GET /rest/v1/absentiemeldingen`

Fetches the absence reports of the user

#### Parameters

| Name           | Type      | Value                 |
| -------------- | --------- | --------------------- |
| Authorization  | Header    | Bearer [access_token] |
| begindatumtijd | Parameter | yyyy-MM-dd            |
| einddatumtijd  | Parameter | yyyy-MM-dd            |

#### Returns

Array of absance reports

```json
{
  "items": [
    {
      "$type": "participatie.RAbsentieMelding",
      "links": [
        {
          "id": 1234567890123,
          "rel": "self",
          "type": "participatie.RAbsentieMelding",
          "href": "{{api_url}}/rest/v1/waarnemingen/1234567890123"
        }
      ],
      "permissions": [],
      "additionalObjects": {},
      "leerling": {
        "links": [
          {
            "id": 1234567890,
            "rel": "self",
            "type": "leerling.RLeerlingPrimer",
            "href": "{{api_url}}/rest/v1/leerlingen/1234567890"
          }
        ],
        "permissions": [],
        "additionalObjects": {},
        "UUID": "12abc34e-12a3-1a2b-a1b2-1a2b34cd5e67",
        "leerlingnummer": 100000,
        "roepnaam": "Name",
        "achternaam": "Name"
      },
      "absentieReden": {
        "links": [
          {
            "id": 1234567890,
            "rel": "self",
            "type": "participatie.RAbsentieRedenPrimer",
            "href": "{{api_url}}/rest/v1/absentieredenen/1234567890"
          }
        ],
        "permissions": [],
        "additionalObjects": {},
        "absentieSoort": "Absent",
        "afkorting": "XC",
        "omschrijving": "Onbekend",
        "geoorloofd": false
      },
      "datumTijdInvoer": "yyyy-MM-dd'T'HH:mm:ss.SSS+HH:mm",
      "beginDatumTijd": "yyyy-MM-dd'T'HH:mm:ss.SSS+HH:mm",
      "eindDatumTijd": "yyyy-MM-dd'T'HH:mm:ss.SSS+HH:mm",
      "beginLesuur": 3,
      "eindLesuur": 3,
      "afgehandeld": true,
      "eigenaar": {
        "links": [
          {
            "id": 1234567890,
            "rel": "self",
            "type": "medewerker.RMedewerker",
            "href": "{{api_url}}/rest/v1/medewerkers/1234567890"
          }
        ],
        "permissions": [],
        "additionalObjects": {},
        "UUID": "12abc34e-12a3-1a2b-a1b2-1a2b34cd5e67",
        "nummer": 100000,
        "afkorting": "HH",
        "achternaam": "Henk",
        "geslacht": "MAN",
        "voorletters": "H.H.",
        "roepnaam": "Hans"
      }
    }
  ]
}
```

### Study Guides: `GET /rest/v1/studiewijzers`

Fetches the study guides for the user

#### Parameters

| Name          | Type      | Value                 |
| ------------- | --------- | --------------------- |
| Authorization | Header    | Bearer [access_token] |
| additional    | Parameter | leerlingen            |
| additional    | Parameter | bijlagen              |
| additional    | Parameter | externeMaterialen     |
| additional    | Parameter | bijlageMappen         |

The additional parameters are optional GET parameters to include information in the result. `leerlingen` will only give back 1 result when queried by a student, but will fetch all students when queried by a teacher/school admin.

#### Returns

Depending on the additional parameters, some of the items in the result may not be present. Assuming all 4 are set:

```json
{
    "items": [
        {
            "$type": "studiewijzer.RStudiewijzer",
            "links": [
                {
                    "id": 3709468886305,
                    "rel": "self",
                    "type": "studiewijzer.RStudiewijzer",
                    "href": "https://api.somtoday.nl/rest/v1/studiewijzers/3709468886305"
                }
            ],
            "permissions": [
                {
                    "full": "studiewijzer.RStudiewijzer:READ:INSTANCE(3709468886305)",
                    "type": "studiewijzer.RStudiewijzer",
                    "operations": [
                        "READ"
                    ],
                    "instances": [
                        "INSTANCE(3709468886305)"
                    ]
                }
            ],
            "additionalObjects": {
                "bijlageMappen": {
                    "$type": "LinkableWrapper",
                    "items": []
                },
                "bijlagen": {
                    "$type": "LinkableWrapper",
                    "items": []
                },
                "leerlingen": {
                    "$type": "LinkableWrapper",
                    "items": [
                        {
                            "$type": "leerling.RLeerlingPrimer",
                            "links": [
                                {
                                    "id": 9496745174,
                                    "rel": "self",
                                    "type": "leerling.RLeerlingPrimer",
                                    "href": "https://api.somtoday.nl/rest/v1/leerlingen/9496745174"
                                }
                            ],
                            "permissions": [
                                {
                                    "full": "leerling.RLeerlingPrimer:READ:INSTANCE(9496745174)",
                                    "type": "leerling.RLeerlingPrimer",
                                    "operations": [
                                        "READ"
                                    ],
                                    "instances": [
                                        "INSTANCE(9496745174)"
                                    ]
                                }
                            ],
                            "additionalObjects": {},
                            "UUID": "f8cf6f6c-c213-4526-8ba1-6a306cf724a4",
                            "leerlingnummer": 123456,
                            "roepnaam": "{{first_name}}",
                            "achternaam": "{{last_name}}"
                        }
                    ]
                },
                "externeMaterialen": {
                    "$type": "LinkableWrapper",
                    "items": []
                }
            },
            "uuid": "4d2188a0-03d8-4dca-9f51-0e54d3c353c6",
            "naam": "vwo5.schka",
            "vestiging": {
                "links": [
                    {
                        "id": 9496567717,
                        "rel": "self",
                        "type": "instelling.RVestiging",
                        "href": "https://api.somtoday.nl/rest/v1/vestigingen/9496567717"
                    }
                ],
                "permissions": [
                    {
                        "full": "instelling.RVestiging:READ:INSTANCE(9496567717)",
                        "type": "instelling.RVestiging",
                        "operations": [
                            "READ"
                        ],
                        "instances": [
                            "INSTANCE(9496567717)"
                        ]
                    }
                ],
                "additionalObjects": {},
                "naam": "Stella Maris College Meerssen"
            },
            "lesgroep": {
                "links": [
                    {
                        "id": 3543707887108,
                        "rel": "self",
                        "type": "lesgroep.RLesgroep",
                        "href": "https://api.somtoday.nl/rest/v1/lesgroepen/3543707887108"
                    }
                ],
                "permissions": [
                    {
                        "full": "lesgroep.RLesgroep:READ:INSTANCE(3543707887108)",
                        "type": "lesgroep.RLesgroep",
                        "operations": [
                            "READ"
                        ],
                        "instances": [
                            "INSTANCE(3543707887108)"
                        ]
                    }
                ],
                "additionalObjects": {},
                "UUID": "d4afb5b8-fbf6-4bbd-ac73-cb50cc883392",
                "naam": "vwo5.schka",
                "schooljaar": {
                    "$type": "onderwijsinrichting.RSchooljaar",
                    "links": [
                        {
                            "id": 40851957,
                            "rel": "self",
                            "type": "onderwijsinrichting.RSchooljaar",
                            "href": "https://api.somtoday.nl/rest/v1/schooljaren/40851957"
                        }
                    ],
                    "permissions": [
                        {
                            "full": "onderwijsinrichting.RSchooljaar:READ:INSTANCE(40851957)",
                            "type": "onderwijsinrichting.RSchooljaar",
                            "operations": [
                                "READ"
                            ],
                            "instances": [
                                "INSTANCE(40851957)"
                            ]
                        }
                    ],
                    "additionalObjects": {},
                    "naam": "2021/2022",
                    "vanafDatum": "2021-08-01",
                    "totDatum": "2022-07-31",
                    "isHuidig": true
                },
                "vak": {
                    "links": [
                        {
                            "id": 9505018979,
                            "rel": "self",
                            "type": "onderwijsinrichting.RVak",
                            "href": "https://api.somtoday.nl/rest/v1/vakken/9505018979"
                        }
                    ],
                    "permissions": [
                        {
                            "full": "onderwijsinrichting.RVak:READ:INSTANCE(9505018979)",
                            "type": "onderwijsinrichting.RVak",
                            "operations": [
                                "READ"
                            ],
                            "instances": [
                                "INSTANCE(9505018979)"
                            ]
                        }
                    ],
                    "additionalObjects": {},
                    "afkorting": "schk",
                    "naam": "Scheikunde"
                },
                "heeftStamgroep": false,
                "examendossierOndersteund": true,
                "vestiging": {
                    "links": [
                        {
                            "id": 9496567717,
                            "rel": "self",
                            "type": "instelling.RVestiging",
                            "href": "https://api.somtoday.nl/rest/v1/vestigingen/9496567717"
                        }
                    ],
                    "permissions": [
                        {
                            "full": "instelling.RVestiging:READ:INSTANCE(9496567717)",
                            "type": "instelling.RVestiging",
                            "operations": [
                                "READ"
                            ],
                            "instances": [
                                "INSTANCE(9496567717)"
                            ]
                        }
                    ],
                    "additionalObjects": {},
                    "naam": "Stella Maris College Meerssen"
                }
            }
        }
        ...
    ]
}
```

---

### Undocumented:

- `GET /rest/v1/medewerkers/ontvangers`
- `GET/rest/v1/account/2555374351`
- `GET/rest/v1/maatregeltoekenningen`
- `GET/rest/v1/leerlingadresseringen`
- `GET/rest/v1/schooljaren/huidig`
- `GET/rest/v1/vakkeuzes`
- `GET/rest/v1/verzorgers/`
- `GET /rest/v1/waarnemingen/`
- `GET /rest/v1/onderwijsopafstandperiodes/`
