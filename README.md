# SOMtoday REST API docs

#### Discord

[![Discord Chat](https://img.shields.io/discord/789249810032361502.svg)](https://discord.gg/yE3e3erCut)

## Table of contents

<!-- TOC -->

- [SOMtoday REST API docs](#somtoday-rest-api-docs)
  - [Discord](#discord)
  - [Table of contents](#table-of-contents)
  - [Some miscellaneous stuff](#some-miscellaneous-stuff)
  - [Authentication / authorization](Authentication.md)
    - [Getting a list of schools](Authentication.md#getting-a-list-of-schools)
    - [Authentication by mimicking the SOMToday app/webapp](Authentication.md#authentication-by-mimicking-the-somtoday-appwebapp)
    - [Fetching the access token via SSO](Authentication.md#authentication-using-sso-single-sign-on)
    - [Fetching the access token via Somtoday login: `POST /oauth2/token`](Authentication.md#fetching-the-access-token-via-somtoday-login)
    - [Refreshing the token: `POST /oauth2/token`](Authentication.md#refreshing-the-access-token)
  - [Fetching information](#fetching-information)
    - [Current student(s): `GET /rest/v1/leerlingen`](#current-students-get-restv1leerlingen)
    - [Student by ID: `GET /rest/v1/leerlingen/[id]`](#student-by-id-get-restv1leerlingenid)
    - [Grades: `GET /rest/v1/resultaten/huidigVoorLeerling/[id]`](#grades-get-restv1resultatenhuidigvoorleerlingid)
    - [Schedule: `GET /rest/v1/afspraken`](#schedule-get-restv1afspraken)
    - [Absence Reports: `GET /rest/v1/absentiemeldingen`](#absence-reports-get-restv1absentiemeldingen)
    - [Study Guides: `GET /rest/v1/studiewijzers`](#study-guides-get-restv1studiewijzers)
    - [Subjects: `GET /rest/v1/vakken`](#subjects-get-restv1vakken)
    - [User Account: `GET /rest/v1/account`](#account-get-restv1account--get-restv1accountid)
    - [School Years: `GET /rest/v1/schooljaren`](#schooljaren-get-restv1schooljaren--get-restv1schooljarenid)
    - [Vakkeuzes: `GET /rest/v1/vakkeuzes`](#vakkeuzes-get-restv1vakkeuzes)
    - [Waarnemingen: `GET /rest/v1/waarnemingen`](#waarnemingen-get-restv1waarnemingen)
    - [ICalendar: `GET /rest/v1/icalendar`](#icalendar-get-restv1icalendar)
    - [ICalendar: `DELETE /rest/v1/icalendar`](#icalendar-delete-restv1icalendar)
  - [Homework](Homework.md)
    - [1. Homework from appointments: `GET /rest/v1/studiewijzeritemafspraaktoekenningen`](Homework.md#1-homework-from-appointments-get-restv1studiewijzeritemafspraaktoekenningen)
    - [2. Homework from days: `GET /rest/v1/studiewijzeritemdagtoekenningen`](Homework.md#2-homework-from-days-get-restv1studiewijzeritemdagtoekenningen)
    - [3. Homework from weeks: `GET /rest/v1/studiewijzeritemweektoekenningen`](Homework.md#3-homework-from-weeks-get-restv1studiewijzeritemweektoekenningen)<br><br>
    - [1. Homework Made `PUT /rest/v1/swigemaakt/[id]`](Homework.md#1-homework-made-put-restv1swigemaaktid)
    - [2. Homework Made `PUT /rest/v1/swigemaakt/cou`](Homework.md#2-homework-made-put-restv1swigemaaktcou)



<!-- /TOC -->

## Some miscellaneous stuff
<details><summary>Click to open miscellaneous stuff</summary>


 - Endpoint for the API is returned when you fetch the access token
 - Always include the header "Accept" with the value of "application/json" so you won't get XML. (except if you want XML :-) ) (the authentication stuff always returns JSON)<br><br>

 - you can do sample requests using curl, for example:

```bash
curl http://example.com/user/blah?active=true&limit=3 -d "key=value&otherkey=value" -H "AHeader: Value"
```

which will be listed here as

| Name     | Type   | Value |
|----------|--------|-------|
| id       | URL    | blah  |
| active   | Query  | true  |
| limit    | Query  | 3     |
| key      | Body   | value |
| otherkey | Body   | value |
| AHeader  | Header | Value |

When there is a value that is unique to you (like username, password, or token), it will have a value like `[username]`

I don't recommend using curl in your programming language, except for PHP but even there it's a pain. There are much better libraries.

<details><summary>A list of libraries for your language </summary>  

JavaScript: [window.fetch](https://developers.google.com/web/updates/2015/03/introduction-to-fetch)<br>
NodeJS: [node-fetch](https://github.com/bitinn/node-fetch), [HTTP from stdlib](https://nodejs.org/api/http.html), [Request](https://github.com/request/request), [Axios](https://github.com/axios/axios)<br>
Go: [net/http](https://golang.org/pkg/net/http/)<br>
Ruby: [Faraday](https://github.com/lostisland/faraday), [HTTParty](https://github.com/jnunemaker/httparty)<br>
Python: [requests](http://docs.python-requests.org/en/master/)<br>

Please add more if you know more.

</details>

</details>

## Fetching information

baseurl: returned when you fetch a token (`somtoday_api_url`), usually [lowercase snakecased schoolname]-api.somtoday.nl

All routes here are prefixed with that baseurl.

### Current student(s): `GET /rest/v1/leerlingen`
<details><summary>Click to open</summary>

This REST method might return multiple students (I cannot test), since it says /leerlingen (Dutch plural for student).

I suppose it returns all students the current user has access to (so if a school administrator runs it, it will return all students on the school).

#### Parameters

| Name          | Type      | Value                 |
|---------------|-----------|-----------------------|
| Authorization | Header    | Bearer [access_token] |
| additional    | Parameter | pasfoto               |

The additional parameter is an optional GET parameter.

#### Returns

Depending on the additional parameters, some of the items in the result may not be present. Assuming `pasfoto` is set:

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
      "additionalObjects": {
        "pasfoto": {
          "$type": "leerling.RLeerlingpasfoto",
          "links": [
            {
              "id": 1234,
              "rel": "self"
            }
          ],
          "permissions": [],
          "additionalObjects": {},
          "datauri": "<base64 image>"
        }
      },
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
|---------------|--------|-----------------------|
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

</details>

### Grades: `GET /rest/v1/resultaten/huidigVoorLeerling/[id]`
<details><summary>Click to open</summary>

Fetches the grades of the student. Note that all average grades are also grade items returned by the API. There are the different types of columns: the `type` property in the json (e.g. 'Toetskolom', 'ToetssoortGemiddeldeKolom').

#### Parameters

| Name          | Type      | Value                           |
|---------------|-----------|---------------------------------|
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

</details>

### Schedule: `GET /rest/v1/afspraken`
<details><summary>Click to open</summary>

Fetch the appointments from the schedule of the student.

#### Parameters

| Name          | Type      | Value                 |
|---------------|-----------|-----------------------|
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

</details>

### Absence Reports: `GET /rest/v1/absentiemeldingen`
<details><summary>Click to open</summary>

Fetches the absence reports of the user

#### Parameters

| Name           | Type      | Value                 |
|----------------|-----------|-----------------------|
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
</details>

### Study Guides: `GET /rest/v1/studiewijzers`
<details><summary>Click to open</summary>

Fetches the study guides for the user

#### Parameters

| Name          | Type      | Value                 |
|---------------|-----------|-----------------------|
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

</details>

### Subjects: `GET /rest/v1/vakken`
<details><summary>Click to open</summary>

Fetches the subjects for the user

#### Parameters

| Name          | Type   | Value                 |
|---------------|--------|-----------------------|
| Authorization | Header | Bearer [access_token] |

#### Returns

```json
{
  "items": [
    {
      "$type": "onderwijsinrichting.RVak",
      "links": [
        {
          "id": 123456789,
          "rel": "self",
          "type": "onderwijsinrichting.RVak",
          "href": "https://api.somtoday.nl/rest/v1/vakken/123456789"
        }
      ],
      "permissions": [
        {
          "full": "onderwijsinrichting.RVak:READ:INSTANCE(123456789)",
          "type": "onderwijsinrichting.RVak",
          "operations": [
            "READ"
          ],
          "instances": [
            "INSTANCE(123456789)"
          ]
        }
      ],
      "additionalObjects": {},
      "afkorting": "<abbreviation>",
      "naam": "<subject>"
    }
	...
  ]
}
```

</details>

### Account: `GET /rest/v1/account/` / `GET /rest/v1/account/[id]`
<details><summary>Click to open</summary>

Fetches information about the account that is connected with the Somtoday access token

#### Parameters

| Name          | Type   | Value                 |
|---------------|--------|-----------------------|
| id            | URL    | [user-id]             |
| Authorization | Header | Bearer [access_token] |

There are some parameters seeing the 'additionalObjects' field, but I don't know what they are.
#### Returns



```json
{
  "items": [
    {
      "$type": "auth.RAccount",
      "links": [
        {
          "id": 1234567890,
          "rel": "self",
          "type": "auth.RAccount",
          "href": "https://api.somtoday.nl/rest/v1/account/1234567890"
        }
      ],
      "permissions": [
        {
          "full": "auth.RAccount:READ:INSTANCE(1234567890)",
          "type": "auth.RAccount",
          "operations": [
            "READ"
          ],
          "instances": [
            "INSTANCE(1234567890)"
          ]
        }
      ],
      "additionalObjects": {},
      "gebruikersnaam": "[REDACTED]",
      "accountPermissions": [],
      "persoon": {
        "$type": "leerling.RLeerlingPrimer",
        "links": [
          {
            "id": "0123456789",
            "rel": "self",
            "type": "leerling.RLeerlingPrimer",
            "href": "https://api.somtoday.nl/rest/v1/leerlingen/0123456789"
          }
        ],
        "permissions": [
          {
            "full": "leerling.RLeerlingPrimer:READ:INSTANCE(1409824200)",
            "type": "leerling.RLeerlingPrimer",
            "operations": [
              "READ"
            ],
            "instances": [
              "INSTANCE(0123456789)"
            ]
          }
        ],
        "additionalObjects": {},
        "UUID": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "leerlingnummer": 100000,
        "roepnaam": "Name",
        "voorvoegsel": "Name",
        "achternaam": "Name"
      }
    }
  ]
}
```

</details>

### Schooljaren: `GET /rest/v1/schooljaren` / `GET /rest/v1/schooljaren/[id]`
<details><summary>Click to open</summary>

Fetches information about a school year

#### Parameters

| Name          | Type   | Value                 |
|---------------|--------|-----------------------|
| id            | URL    | [id]                  |
| id            | URL    | huidig                |
| Authorization | Header | Bearer [access_token] |

When you want info about the current school year add /huidig to the url

#### Returns

```json
{
  "items": [
    {
      "$type": "onderwijsinrichting.RSchooljaar",
      "links": [
        {
          "id": 40851958, //this id is for everyone the same (in this case for year 2022/2023)
          "rel": "self",
          "type": "onderwijsinrichting.RSchooljaar",
          "href": "https://api.somtoday.nl/rest/v1/schooljaren/40851958"
        }
      ],
      "permissions": [
        {
          "full": "onderwijsinrichting.RSchooljaar:READ:INSTANCE(40851958)",
          "type": "onderwijsinrichting.RSchooljaar",
          "operations": [
            "READ"
          ],
          "instances": [
            "INSTANCE(40851958)"
          ]
        }
      ],
      "additionalObjects": {},
      "naam": "2022/2023",
      "vanafDatum": "2022-08-01",
      "totDatum": "2023-07-31",
      "isHuidig": true
    },
    ...
  ]
}
```

</details>

### Vakkeuzes: `GET /rest/v1/vakkeuzes`
<details><summary>Click to open</summary>

Fetches all the subjects you are currently enrolled in.

#### Parameters

| Name          | Type      | Value                 |
|---------------|-----------|-----------------------|
| Authorization | Header    | Bearer [access_token] |
| additional    | Parameter | vaknormering          |
| additional    | Parameter | actiefOpPeildatum     |

#### Returns

```json
{
  "items": [
    {
      "$type": "onderwijsinrichting.RVakkeuze",
      "links": [
        {
          "id": xxxxxxxxxx,
          "rel": "self",
          "type": "onderwijsinrichting.RVakkeuze",
          "href": "https://api.somtoday.nl/rest/v1/vakkeuzes/xxxxxxxxxx"
        }
      ],
      "permissions": [
        {
          "full": "onderwijsinrichting.RVakkeuze:READ:INSTANCE(xxxxxxxxxx)",
          "type": "onderwijsinrichting.RVakkeuze",
          "operations": [
            "READ"
          ],
          "instances": [
            "INSTANCE(xxxxxxxxxx)"
          ]
        }
      ],
      "additionalObjects": {
        "vaknormering": {
          "$type": "onderwijsinrichting.RVakNormering",
          "vakId": yyyyyyyyyy,
          "toetsnormering1": "Standaard",
          "toetsnormering2": "Alternatief"
        }
      },
      "vak": {
        "links": [
          {
            "id": yyyyyyyyyy,
            "rel": "self",
            "type": "onderwijsinrichting.RVak",
            "href": "https://api.somtoday.nl/rest/v1/vakken/yyyyyyyyyy"
          }
        ],
        "permissions": [
          {
            "full": "onderwijsinrichting.RVak:READ:INSTANCE(yyyyyyyyyy)",
            "type": "onderwijsinrichting.RVak",
            "operations": [
              "READ"
            ],
            "instances": [
              "INSTANCE(yyyyyyyyyy)"
            ]
          }
        ],
        "additionalObjects": {},
        "afkorting": "ne",
        "naam": "Nederlandse taal"
      }
    },
    ...
  ]
}
```

</details>

### Waarnemingen: `GET /rest/v1/waarnemingen`
<details><summary>Click to open</summary>

Fetches all the waarnemingen currently tied to your account, filter them by date, isGeoorloofd and/or waarnemingSoort.

#### Parameters

| Name                       | Type      | Value                 |
|----------------------------|-----------|-----------------------|
| Authorization              | Header    | Bearer [access_token] |
| waarnemingSoort (optional) | Parameter | Afwezig/aanwezig      |
| isGeoorloofd (optional)    | Parameter | true/false            |

You can, if you want, provide dates to filter the results. If you don't provide any dates, it will return all the results. 
You can either provide a date range or a single date. If you provide a single date, it will return all the results from that date. If you provide a date range, it will return all the results inbetween those dates.

| Date types      | Type      | Value       |
|-----------------|-----------|-------------|
| begintNaOfOp    | Parameter | yyyy-MM-dd  |
| OR              |
| beginDatumTijd  | Parameter | yyyy-MM-dd  |
| eindDatumTijd   | Parameter | yyyy-MM-dd  |

#### Returns

```json
{
  "items": [
    {
      "$type": "participatie.RWaarneming",
      "links": [
        {
          "id": 1234567891234,
          "rel": "self",
          "type": "participatie.RWaarneming",
          "href": "https://api.somtoday.nl/rest/v1/waarnemingen/1234567891234"
        }
      ],
      "permissions": [
        {
          "full": "participatie.RWaarneming:READ:INSTANCE(1234567891234)",
          "type": "participatie.RWaarneming",
          "operations": [
            "READ"
          ],
          "instances": [
            "INSTANCE(1234567891234)"
          ]
        }
      ],
      "additionalObjects": {},
      "beginDatumTijd": "2023-01-09T11:05:00.000+01:00",
      "eindDatumTijd": "2023-01-09T11:55:00.000+01:00",
      "beginLesuur": 4,
      "eindLesuur": 4,
      "waarnemingSoort": "Aanwezig",
      "leerling": {
        "links": [
          {
            "id": 1234567890,
            "rel": "self",
            "type": "leerling.RLeerlingPrimer",
            "href": "https://api.somtoday.nl/rest/v1/leerlingen/1234567890"
          }
        ],
        "permissions": [
          {
            "full": "leerling.RLeerlingPrimer:READ:INSTANCE(1234567890)",
            "type": "leerling.RLeerlingPrimer",
            "operations": [
              "READ"
            ],
            "instances": [
              "INSTANCE(1234567890)"
            ]
          }
        ],
        "additionalObjects": {},
        "UUID": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "leerlingnummer": 100000,
        "roepnaam": "Name",
        "voorvoegsel": "Name",
        "achternaam": "Name"
      },
      "afspraak": {
        "links": [
          {
            "id": 12345678901345,
            "rel": "self",
            "type": "participatie.RAfspraakPrimer",
            "href": "https://api.somtoday.nl/rest/v1/afspraken/12345678901345"
          }
        ],
        "permissions": [
          {
            "full": "participatie.RAfspraak:READ:INSTANCE(12345678901345)",
            "type": "participatie.RAfspraak",
            "operations": [
              "READ"
            ],
            "instances": [
              "INSTANCE(12345678901345)"
            ]
          }
        ],
        "additionalObjects": {},
        "afspraakType": {
          "links": [
            {
              "id": 1234567890,
              "rel": "self",
              "type": "participatie.RAfspraakType",
              "href": "https://api.somtoday.nl/rest/v1/afspraaktype/1234567890"
            }
          ],
          "permissions": [
            {
              "full": "participatie.RAfspraakType:READ:INSTANCE(1234567890)",
              "type": "participatie.RAfspraakType",
              "operations": [
                "READ"
              ],
              "instances": [
                "INSTANCE(1234567890)"
              ]
            }
          ],
          "additionalObjects": {},
          "naam": "LES",
          "omschrijving": "LES",
          "standaardKleur": -16448251,
          "categorie": "Rooster",
          "activiteit": "Verplicht",
          "percentageIIVO": 100,
          "presentieRegistratieDefault": true,
          "actief": true,
          "vestiging": {
            "$type": "instelling.RVestiging",
            "links": [
              {
                "id": 1234567890,
                "rel": "self",
                "type": "instelling.RVestiging",
                "href": "https://api.somtoday.nl/rest/v1/vestigingen/1234567890"
              }
            ],
            "permissions": [
              {
                "full": "instelling.RVestiging:READ:INSTANCE(1234567890)",
                "type": "instelling.RVestiging",
                "operations": [
                  "READ"
                ],
                "instances": [
                  "INSTANCE(1234567890)"
                ]
              }
            ],
            "additionalObjects": {},
            "naam": "De super coole school",
          }
        },
        "locatie": "lokaal naam",
        "beginDatumTijd": "2023-01-09T11:05:00.000+01:00",
        "eindDatumTijd": "2023-01-09T11:55:00.000+01:00",
        "beginLesuur": 4,
        "eindLesuur": 4,
        "titel": "titel"
      },
      "afgehandeld": true,
      "invoerDatum": "2023-01-09T11:09:08.000+01:00",
      "laatstGewijzigdDatum": "2023-01-09T11:09:08.000+01:00",
      "herkomst": "Medewerker",
      "ingevoerdDoor": {
        "links": [
          {
            "id": 1234567890123,
            "rel": "self",
            "type": "medewerker.RMedewerkerPrimer",
            "href": "https://api.somtoday.nl/rest/v1/medewerkers/1234567890123"
          }
        ],
        "permissions": [],
        "additionalObjects": {},
        "UUID": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "nummer": 12345678,
        "afkorting": "afkorting",
        "achternaam": "name",
        "geslacht": "VROUW/MAN",
        "voorletters": "voorletter(s)",
        "roepnaam": "roepnaam"
      },
      "laatstGewijzigdDoor": {
        "links": [
          {
            "id": 1234567890123,
            "rel": "self",
            "type": "medewerker.RMedewerkerPrimer",
            "href": "https://api.somtoday.nl/rest/v1/medewerkers/1234567890123"
          }
        ],
        "permissions": [],
        "additionalObjects": {},
        "UUID": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "nummer": 12345678,
        "afkorting": "afkorting",
        "achternaam": "name",
        "geslacht": "VROUW/MAN",
        "voorletters": "voorletter(s)",
        "roepnaam": "roepnaam"
      }
    },
    ...
  ]
}
```

</details>

### ICalendar: `GET /rest/v1/icalendar`
<details><summary>Click to open</summary>

Fetches the url to the icalendar stream.

#### Parameters

| Name          | Type      | Value                 |
|---------------|-----------|-----------------------|
| Authorization | Header    | Bearer [access_token] |

#### Returns

```json
{
    "links": [],
    "permissions": [],
    "additionalObjects": {},
    "leerlingICalendarLink": "https://api.somtoday.nl/rest/v1/icalendar/stream/REDACTED"
}
```

</details>

### ICalendar: `DELETE /rest/v1/icalendar`
<details><summary>Click to open</summary>

Deletes the currently active icalendar stream

#### Parameters

| Name          | Type      | Value                 |
|---------------|-----------|-----------------------|
| Authorization | Header    | Bearer [access_token] |

#### Returns

NONE

</details>
### Undocumented:

- `GET /rest/v1/medewerkers/ontvangers`
- `GET /rest/v1/maatregeltoekenningen`
- `GET /rest/v1/leerlingadresseringen`
- `GET /rest/v1/verzorgers/`
- `GET /rest/v1/onderwijsopafstandperiodes/`
