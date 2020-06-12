# SOMtoday REST API docs

## Table of contents
<!-- MarkdownTOC -->

- [Some miscellaneous stuff](#some-miscellaneous-stuff)
- [SOMtoday metadata](#somtoday-metadata)
  - [Getting a list of schools: `GET https://servers.somtoday.nl/organisaties.json`](#getting-a-list-of-schools-get-httpsserverssomtodaynlorganisatiesjson)
- [Authentication / authorization](#authentication--authorization)
  - [Fetching the access token: `POST /oauth2/token`](#fetching-the-access-token-post-oauth2token)
    - [Parameters](#parameters)
    - [Returns](#returns)
    - [Example](#example)
  - [Refreshing the token: `POST /oauth2/token`](#refreshing-the-token-post-oauth2token)
    - [Parameters](#parameters-1)
    - [Returns](#returns-1)
    - [Example](#example-1)
- [Getting information](#getting-information)
  - [Current student\(s\): `GET /rest/v1/leerlingen`](#current-students-get-restv1leerlingen)
    - [Parameters](#parameters-2)
    - [Returns](#returns-2)
    - [Example](#example-2)
  - [Student by ID: `GET /rest/v1/leerlingen/[id]`](#student-by-id-get-restv1leerlingenid)
    - [Parameters](#parameters-3)
    - [Returns](#returns-3)
    - [Example](#example-3)
  - [Grades: `GET /rest/v1/resultaten/huidigVoorLeerling/[id]`](#grades-get-restv1resultatenhuidigvoorleerlingid)
    - [Parameters](#parameters-4)
    - [Returns](#returns-4)
  - [Schedule: `GET /rest/v1/resultaten/afspraken`](#schedule-get-restv1resultatenafspraken)
    - [Parameters](#parameters-5)
    - [Returns](#returns-5)
  - [Homework: `GET /rest/v1/studiewijzeritemafspraaktoekenningen`](#homework-get-restv1studiewijzeritemafspraaktoekenningen)
    - [Parameters](#parameters-6)
    - [Returns](#returns-6)

<!-- /MarkdownTOC -->

## Some miscellaneous stuff
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
baseurl: https://production.somtoday.nl

All routes here are prefixed with that baseurl.

### Fetching the access token: `POST /oauth2/token`

#### Parameters

|Name|Type|Value|
|----|----|-----|
|grant_type|Body|password|
|username|Body|[school uuid]\\[username]|
|password|Body|[password]|
|scope|Body|openid|
|Authorization|Header|Basic RDUwRTBDMDYtMzJEMS00QjQxLUExMzctQTlBODUwQzg5MkMyOnZEZFdkS3dQTmFQQ3loQ0RoYUNuTmV5ZHlMeFNHTkpY|

OR

|Name|Type|Value|
|----|----|-----|
|grant_type|Body|password|
|username|Body|[school uuid]\\[username]|
|password|Body|[password]|
|scope|Body|openid|
|client_id|Body|D50E0C06-32D1-4B41-A137-A9A850C892C2|
|client_secret|Body|vDdWdKwPNaPCyhCDhaCnNeydyLxSGNJX|

Both of them are tested to work.

**Note: that authorization header is the result of the following parameters:**
```
app ID (username): D50E0C06-32D1-4B41-A137-A9A850C892C2
app secret (password): vDdWdKwPNaPCyhCDhaCnNeydyLxSGNJX
```

The app ID is used as the username and the app secret as the password for HTTP Basic authorization.

This means that it will be formatted as:
`app ID:app secret`, so `D50E0C06-32D1-4B41-A137-A9A850C892C2:vDdWdKwPNaPCyhCDhaCnNeydyLxSGNJX`.

This is then encoded with base64, as is standard for HTTP Basic authorization.

This yields the authorization header we see above.

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
This example uses the HTTP Basic authorization header method of authorization.

```bash
school_uuid='4213a402-b898-4d16-9ebb-8c5f02b57474' username='450000@live.bc-enschede.nl' password='MYSECRETPASSWORD123'
curl "https://production.somtoday.nl/oauth2/token" -d "grant_type=password&username=$school_uuid\\$username&password=$password&scope=openid" -H "Authorization: Basic RDUwRTBDMDYtMzJEMS00QjQxLUExMzctQTlBODUwQzg5MkMyOnZEZFdkS3dQTmFQQ3loQ0RoYUNuTmV5ZHlMeFNHTkpY"
```

**Note: We use `\\` here, because `\` is normally used to escape things like quotes (e.g. `\"`) (and only bash double quote strings can escape using `\`), so `\\` will translate to `\`, and you can just use `\` if you use single quotes**

### Refreshing the token: `POST /oauth2/token`
#### Parameters

|Name|Type|Value|
|----|----|-----|
|grant_type|Body|refresh_token|
|refresh_token|Body|[refresh_token]|
|client_id|Body|D50E0C06-32D1-4B41-A137-A9A850C892C2|
|client_secret|Body|vDdWdKwPNaPCyhCDhaCnNeydyLxSGNJX|

OR

|Name|Type|Value|
|----|----|-----|
|grant_type|Body|refresh_token|
|refresh_token|Body|[refresh_token]|
|Authorization|Header|Basic RDUwRTBDMDYtMzJEMS00QjQxLUExMzctQTlBODUwQzg5MkMyOnZEZFdkS3dQTmFQQ3loQ0RoYUNuTmV5ZHlMeFNHTkpY|

Both of them are tested to work.

You get that `refresh_token` when you fetch the access token with the username and password.

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
curl "https://production.somtoday.nl/oauth2/token" -d "grant_type=refresh_token&refresh_token=$token&client_id=D50E0C06-32D1-4B41-A137-A9A850C892C2&client_secret=vDdWdKwPNaPCyhCDhaCnNeydyLxSGNJX"
```

## Getting information
baseurl: returned when you fetch a token (`somtoday_api_url`), usually [lowercase snakecased schoolname]-api.somtoday.nl

All routes here are prefixed with that baseurl.

### Current student(s): `GET /rest/v1/leerlingen`
This REST method might return multiple pupils (I cannot test), since it says /leerlingen (Dutch plural for Pupil).

I suppose it returns all pupils the current user has access to (so if a school administrator runs it, it will return all pupils on the school).

#### Parameters

|Name|Type|Value|
|----|----|-----|
|Authorization|Header|Bearer [access_token]|

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
          "operations": [
            "READ"
          ],
          "instances": [
            "INSTANCE(1234)"
          ]
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

### Student by ID: `GET /rest/v1/leerlingen/[id]`
#### Parameters

|Name|Type|Value|
|----|----|-----|
|id|URL|[user id]|
|Authorization|Header|Bearer [access_token]|

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
      "operations": [
        "READ"
      ],
      "instances": [
        "INSTANCE(1234)"
      ]
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

### Grades: `GET /rest/v1/resultaten/huidigVoorLeerling/[id]`
#### Parameters

|Name|Type|Value|
|----|----|-----|
|id|URL|[user id]|
|Authorization|Header|Bearer [access_token]|
|Range|Header|items=[LowerBound]-[UpperBound]|

#### Returns
I do not have any grades yet, so I cannot put some sample JSON here. I can however put the java class here that they use in SOMtoday.

```java
public class Resultaat {
  public static final String Descriptor = "resultaten.RResultaat";
  private Date datumInvoer;
  private Integer examenWeging;
  private String geldendResultaat;
  private RResultaat herkansing;
  private Integer herkansingsNummer;
  private RHerkansing herkansingstype;
  private Boolean isExamendossierResultaat;
  private Boolean isVoortgangsdossierResultaat;
  private int leerjaar;
  private RLeerlingPrimer leerling;
  private String omschrijving;
  private RResultaat overschrevenDoor;
  private int periode;
  private String resultaat;
  private String resultaatAfwijkendNiveau;
  private String resultaatLabel;
  private String resultaatLabelAfkorting;
  private String resultaatLabelAfwijkendNiveau;
  private String resultaatLabelAfwijkendNiveauAfkorting;
  private Integer score;
  private boolean teltNietmee;
  private boolean toetsNietGemaakt;
  private RResultaatkolomType type;
  private RVak vak;
  private Integer volgnummer;
  private Integer weging;
}
```

I ommited some stuff here like all the methods.

that java class would result in the following JSON (I have not tested this)

```json
{
  "datumInvoer": -,
  "examenWeging": -,
  "geldendResultaat": -,
  "herkansing": -,
  "herkansingsNummer": -,
  "herkansingstype": -,
  "isExamendossierResultaat": -,
  "isVoortgangsdossierResultaat": -,
  "leerjaar": -,
  "leerling": -,
  "omschrijving": -,
  "overschrevenDoor": -,
  "periode": -,
  "resultaat": -,
  "resultaatAfwijkendNiveau": -,
  "resultaatLabel": -,
  "resultaatLabelAfkorting": -,
  "resultaatLabelAfwijkendNiveau": -,
  "resultaatLabelAfwijkendNiveauAfkorting": -,
  "score": -,
  "teltNietmee": -,
  "toetsNietGemaakt": -,
  "type": -,
  "vak": -,
  "volgnummer": -,
  "weging": -,
}
```

### Schedule: `GET /rest/v1/resultaten/afspraken`
#### Parameters

|Name|Type|Value|
|----|----|-----|
|id|URL|[user id]|
|Authorization|Header|Bearer [access_token]|
|sort|Parameter|asc-id|
|additional|Parameter|vak|
|additional|Parameter|docentAfkortingen|
|additional|Parameter|leerlingen|
|begindatum|Parameter|yyyy-MM-dd|
|einddatum|Parameter|yyyy-MM-dd|
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
                    "operations": [
                        "READ"
                    ],
                    "instances": [
                        "INSTANCE(8849104409)"
                    ]
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
                            "operations": [
                                "READ"
                            ],
                            "instances": [
                                "INSTANCE(126211284)"
                            ]
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
                                    "operations": [
                                        "READ"
                                    ],
                                    "instances": [
                                        "INSTANCE(546308480)"
                                    ]
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
                        "operations": [
                            "READ"
                        ],
                        "instances": [
                            "INSTANCE(144662674)"
                        ]
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
                            "operations": [
                                "READ"
                            ],
                            "instances": [
                                "INSTANCE(126208855)"
                            ]
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
                        "operations": [
                            "READ"
                        ],
                        "instances": [
                            "INSTANCE(126208855)"
                        ]
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


### Homework: `GET /rest/v1/studiewijzeritemafspraaktoekenningen`
This method fetches homework that the pupil can see.

#### Parameters

|Name|Type|Value|
|----|----|-----|
|begintNaOfOp|Parameter|Date as yyyy-MM-dd|
|Authorization|Header|Bearer [access_token]|

#### Returns
```json
{
        "items": [
            {
                "$type": "studiewijzer.RSWIAfspraakToekenning",
                "additionalObjects": {
                },
                "datumTijd": "<REDACTED>",
                "lesgroep": {
                    "additionalObjects": {
                    },
                    "examendossierOndersteund": false,
                    "heeftStamgroep": true,
                    "links": [
                        {
                            "href": "https://nassau-api.somtoday.nl/rest/v1/lesgroepen/<REDACTED>",
                            "id": <REDACTED>,
                            "rel": "self",
                            "type": "lesgroep.RLesgroep"
                        }
                    ],
                    "naam": "<REDACTED>",
                    "permissions": [
                        {
                            "full": "lesgroep.RLesgroep:READ:INSTANCE(<REDACTED>)",
                            "instances": [
                                "INSTANCE(<REDACTED>)"
                            ],
                            "operations": [
                                "READ"
                            ],
                            "type": "lesgroep.RLesgroep"
                        }
                    ],
                    "schooljaar": {
                        "$type": "onderwijsinrichting.RSchooljaar",
                        "additionalObjects": {
                        },
                        "isHuidig": true,
                        "links": [
                            {
                                "href": "https://nassau-api.somtoday.nl/rest/v1/schooljaren/<REDACTED>",
                                "id": <REDACTED>,
                                "rel": "self",
                                "type": "onderwijsinrichting.RSchooljaar"
                            }
                        ],
                        "naam": "2019/2020",
                        "permissions": [
                            {
                                "full": "onderwijsinrichting.RSchooljaar:READ:INSTANCE(<REDACTED>)",
                                "instances": [
                                    "INSTANCE(<REDACTED>)"
                                ],
                                "operations": [
                                    "READ"
                                ],
                                "type": "onderwijsinrichting.RSchooljaar"
                            }
                        ],
                        "totDatum": "2020-07-31",
                        "vanafDatum": "2019-08-01"
                    },
                    "vak": {
                        "additionalObjects": {
                        },
                        "afkorting": "<REDACTED>",
                        "links": [
                            {
                                "href": "https://nassau-api.somtoday.nl/rest/v1/vakken/<REDACTED>",
                                "id": <REDACTED>,
                                "rel": "self",
                                "type": "onderwijsinrichting.RVak"
                            }
                        ],
                        "naam": "Latijnse taal en letterkunde",
                        "permissions": [
                            {
                                "full": "onderwijsinrichting.RVak:READ:INSTANCE(<REDACTED>)",
                                "instances": [
                                    "INSTANCE(<REDACTED>)"
                                ],
                                "operations": [
                                    "READ"
                                ],
                                "type": "onderwijsinrichting.RVak"
                            }
                        ]
                    }
                },
                "links": [
                    {
                        "href": "https://nassau-api.somtoday.nl/rest/v1/studiewijzeritemafspraaktoekenningen/<REDACTED>",
                        "id": <REDACTED>,
                        "rel": "self",
                        "type": "studiewijzer.RSWIAfspraakToekenning"
                    }
                ],
                "permissions": [
                    {
                        "full": "studiewijzer.RSWIAfspraakToekenning:READ:INSTANCE(<REDACTED>)",
                        "instances": [
                            "INSTANCE(<REDACTED>)"
                        ],
                        "operations": [
                            "READ"
                        ],
                        "type": "studiewijzer.RSWIAfspraakToekenning"
                    }
                ],
                "sortering": 0,
                "studiewijzer": {
                    "additionalObjects": {
                    },
                    "links": [
                        {
                            "id": <REDACTED>,
                            "rel": "koppeling",
                            "type": "studiewijzer.RAbstractStudiewijzer"
                        }
                    ],
                    "naam": "<REDACTED>",
                    "permissions": [
                    ],
                    "uuid": "<REDACTED>",
                    "vestiging": {
                        "additionalObjects": {
                        },
                        "links": [
                            {
                                "href": "https://nassau-api.somtoday.nl/rest/v1/vestigingen/<REDACTED>",
                                "id": <REDACTED>,
                                "rel": "self",
                                "type": "instelling.RVestiging"
                            }
                        ],
                        "naam": "Quintus",
                        "permissions": [
                            {
                                "full": "instelling.RVestiging:READ:INSTANCE(<REDACTED>)",
                                "instances": [
                                    "INSTANCE(<REDACTED>)"
                                ],
                                "operations": [
                                    "READ"
                                ],
                                "type": "instelling.RVestiging"
                            }
                        ]
                    }
                },
                "studiewijzerItem": {
                    "additionalObjects": {
                    },
                    "bijlagen": [
                    ],
                    "externeMaterialen": [
                    ],
                    "huiswerkType": "TOETS",
                    "inlevermomenten": [
                    ],
                    "inleverperiodes": false,
                    "lesmateriaal": false,
                    "links": [
                        {
                            "href": "https://nassau-api.somtoday.nl/rest/v1/studiewijzeritems/<REDACTED>",
                            "id": <REDACTED>,
                            "rel": "self",
                            "type": "studiewijzer.RStudiewijzerItem"
                        }
                    ],
                    "notitieZichtbaarVoorLeerling": false,
                    "omschrijving": "REDACTED",
                    "onderwerp": "<REDACTED>",
                    "permissions": [
                        {
                            "full": "studiewijzer.RStudiewijzerItem:READ:INSTANCE(<REDACTED>)",
                            "instances": [
                                "INSTANCE(<REDACTED>)"
                            ],
                            "operations": [
                                "READ"
                            ],
                            "type": "studiewijzer.RStudiewijzerItem"
                        }
                    ],
                    "projectgroepen": false,
                    "tonen": true
                }
            }
        ]
}
```
