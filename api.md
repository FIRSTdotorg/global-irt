# Global IRT REST API v1

[Global IRT (Incident Response Team)](https://github.com/FIRSTdotorg/global-irt) is a project to describe common IRT and abuse contact information. Global IRT participant directories are encouraged to provide a RESTful API that can provide basic querying capabilities to the repository, and return information following [Global IRT specification](https://github.com/adulau/global-irt/wiki/ExistingFormatReview).

Each directory is encouraged to be conformant to this specification and register the compatible API endpoint with the the Global IRT initiative for sourcename reservation and public listing of the API for the CERT community.

Existing repositories:

| Repository (sourcename) | Endpoint URL                        | Status      |
|-------------------------|-------------------------------------|-------------|
| FIRST.Org               | https://api.first.org/global-irt/v1 | Beta        |

## Overview

Each repository will be treated in this document as the **application** responsible for the API. There are some rules that are requirements for these applications: these rules will be informed that the application **MUST** comply. The optional alternatives will be written like what the application **SHOULD** do, but by not doing doesn't mean that the application is not conformant to this documentation.  

The URLs listed in this document are prefixed by the HTTP method required to use them, followed by the address relative to the Endpoint URL of each repository. Take for example the teams listing, which uses the address `GET /teams`. It must be read as a `GET` request to the `[Endpoint URL]/teams` — to query FIRST.Org repository, you could use `curl -X GET https://api.first.org/global-irt/v1/teams`.

The default output format is `application/json`. The applications are encouraged to support additional formats but it's not a requirement. The accepted format should be requested either as a `Accept: ` header or as an extension to the requested URL. These are suggestions for additional formats to be supported:

| HTTP Accept       | Extension           | Format                                                                                                                             |
|-------------------|---------------------|------------------------------------------------------------------------------------------------------------------------------------|
| application/json  | **.json** (default) | JSON. Should be pretty-printed by default. This will be the format used if no extension or `Accept:` header is provided.           |
| application/yaml  | **.yml**            | YAML, a more human-readable format. Applications should use two spaces as indentation and maximum column width of 80 characters.   |
| application/xml   | **.xml**            | XML. Should be pretty-printed by default. Should not contain namespaces and different encodings.                                   |
| application/csv   | **.csv**            | Comma-separated values. The first line contains the column names.                                                                  |


The applications must use UTF8 as the request/response encoding. Applications are also encouraged to compress output with Gzip if the request header `Accept-Encoding: compress, gzip` is present.

## Global parameters

Parameters can be set at the query string of the request. Parameters must be keypair values assigned with an `=` (equal sign): `key=value`. The values must be URL encoded (RFC 3986), and multiple parameters must be separated by `&` (ampersand sign).

There are some parameters that are reserved for pagination and output control. These parameters are eligible for several methods and should not be used for a application-specific purpose.

| Parameter      | Type      | Description                                                                                                                                                                                                                                         |
|----------------|-----------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **fields**     | string    | Comma-separated list of fieldnames to be retrieved. Used only for limiting the available resultset.                                                                                                                                                 |
| **limit**      | integer   | Limits the maximun number of records to be shown. Should be a number between 1 and 100.                                                                                                                                                             |
| **offset**     | integer   | Offsets the list of records by this number. The first item is 0.                                                                                                                                                                                    |
| **sort**       | string    | Comma-separated list of fieldnames to be used to sort the resultset. Fields starting with "-" (minus sign) indicate a descending order. Each application should define its default sorting options.                                                 |
| **envelope**   | boolean   | Use `true`, `false`, `0` or `1`. If set to true will add an object wrapping the resultset, with details on the status, total records found, offset and limit. When `false` this information is returned at the response header. Defaults to `true`. |
| **pretty**     | boolean   | Use `true`, `false`, `0` or `1`. If the result should be pretty-printed or no. Defaults to `true`.                                                                                                                                                  |
| **callback**   | string    | Only for JSONP resultsets, adds the `callback`  as a function call. Only alphanumerical characters are allowed.                                                                                                                                     |

## Response object

Each `GET` request will return a proper response object, either using the HTTP response header (if the parameter `envelope` was not set to `false`) or at the response body. For example, the request for one brazilian CERT listing only team name and e-mail at FIRST.Org repository:

```json
curl -i "https://api.first.org/global-irt/v1/teams?country=br&fields=short-team-name,email&limit=1"
HTTP/1.1 200 OK
Date: Wed, 03 Jun 2015 15:23:09 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 90
X-Version: 1.0
X-Total-Count: 6

[
    {
        "short-team-name": "CAIS/RNP",
        "email": "cais@cais.rnp.br"
    }
]
```

The `Last-Modified` will refer to the last time the resultset that corresponds to this URL was modified, this information can be used for caching (browsers would request the `If-Modified-Since` header to check if it was modified).

The `X-Version` points out the actual API version used.

The `X-Total-Count` header indicates how many fields were matched for this request. Using the parameters `offset` and `limit` is possible to paginate the results properly.

This information, when queried with `envelope=true` will be present at the response body, and the resultset will be available at the `data` property:  

```json
curl -i "https://api.first.org/global-irt/v1/teams?country=br&fields=short-team-name,email&limit=1&envelope=true"
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Content-Length: 297
Last-Modified: Wed, 13 May 2015 01:56:15 +0000

{
    "status": "OK",
    "status_code": 200,
    "version": "1.0",
    "total": 6,
    "last-modified": "Wed, 13 May 2015 01:56:15 +0000",
    "limit": 1,
    "offset": 0,
    "data": [
        {
            "short-team-name": "CAIS/RNP",
            "email": "cais@cais.rnp.br"
        }
    ]
}
```

**Note**: using a `limit` of "0" (zero) will still return a valid response object with the proper count of matches. 

# Methods

## GET /teams

Retrieves or queries the Teams public listing. Applications should enable filtering of these lists.

**Properties** 

Each response object may contain the following description fieldnames. Empty values are omitted from the resultset. Every property might be used to query/fiter the list.

| Property                     | Type      | Description                                                                                                                   |
|------------------------------|-----------|-------------------------------------------------------------------------------------------------------------------------------|
| **short-team-name**          | string    | Short Team name. This should be preferreably under 50 characters. |
| **official-team-name**       | string    | Official Team name. |
| **postal-address**           | string    | Valid contact postal address. |
| **country-code**             | string    | Official country of the team. Uses ISO 3166-1 alpha-2 codes. |
| **additional-country-code**  | list      | Other countries the team supports. Uses ISO 3166-1 alpha-2 codes. |
| **website**                  | list      | List of valid websites, news and social media URLs. |
| **email**                    | string    | Official contact e-mail. |
| **host**                     | string    | Host organization of the team. |
| **establishment**            | date      | Team date of establishment. Uses ISO 8601 format. |
| **phone-numbers**            | list      | List of valid phone and fax numbers for contacting the team. |
| **enckeys**                  | list      | List of valid PGP key ids that can be used for encrypting content to the team. |
| **operating-hours**          | string    | Textual description of operating hours of the team. |
| **constituency**             | string    | Official team constituency. Should use a controlled vocabulary. |
| **constituency-code**        | string    | Repository-specific code for representing the constituency. |
| **constituency-description** | string    | Free textual constituency description. |
| **source-name**              | string    | Name of the repository. |
| **last-modified**            | string    | Date the entity was last updated or verified. |

**Application-specific properties**

Properties that are specific to one repository should be prefixed with `-[repository code]`. For example, FIRST.Org repository might make available the `member-type` property as `-first-member-type`.

| Property                     | Type      | Description                                                                                                                   |
|------------------------------|-----------|-------------------------------------------------------------------------------------------------------------------------------|
| -first-**member-type**       | string    | FIRST.Org membership type. Should be either *Full Member* or *Liaison*. |


**Additional parameters** 

All output fields, with the exception of last-modified, can be used as params for queries. They are used as exact terms (case insensitive) compared to the field value. In addition, these following parameters permit the querying in several fields at once:

| Parameter      | Type      | Description                                                                                                                   |
|----------------|-----------|-------------------------------------------------------------------------------------------------------------------------------|
| **team**       | string    | Free text search on both `short-team-name` and `official-team-name`.                                                          |
| **country**    | string    | Accepts a comma-separated list of country codes to filter the resultset of both `country-code` and `additional-country-code`. |
| **region**     | string    | Region to search the teams for. Uses ISO 3166 region names.                                                                   |
| **q**          | string    | Searches for the words in all available fields (except `last-modified`).                                                      |

**Example response object** 

```json
curl -X GET "https://api.first.org/global-irt/v1/teams?q=circl"

[
    {
        "short-team-name": "CIRCL",
        "official-team-name": "CIRCL - Computer Incident Response Center Luxembourg",
        "postal-address": "CIRCL - Computer Incident Response Center Luxembourg\r\nc/o smile - \"security made in Letzebuerg\" GIE\r\n41, avenue de la gare\r\nL-1611 Luxembourg\"",
        "country-code": "LU",
        "website": [
            "https://www.circl.lu/"
        ],
        "email": "info@circl.lu",
        "host": "security made in Lëtzebuerg\" (SMILE) g.i.e.",
        "establishment": "2008-01-05",
        "phone-numbers": [
            "+352-247-88-444"
        ],
        "enckeys": [
            "0xCA572205C0024E06BA70BE89EAADCFFC22BD4CD5"
        ],
        "operating-hours": "During legal workdays (Monday to Friday) from 9:00 to 12:00 and 14:00 to 17:00 Central European Time (GMT+0100, GMT+0200 from April to October according to daylight saving periode)",
        "constituency": "Government, Private and Public sectors",
        "constituency-code": "008",
        "constituency-description": "CIRCL is the CERT for the private sector, communes and non-governmental entities for the Grand Duchy of Luxembourg.",
        "source-name": "FIRST.Org",
        "-first-member-type": "Full Member",
        "last-modified": "2015-06-03T14:05:27+00:00"
    }
]
```

Last modified: June 12th, 2016
version: 1.0.1-beta
