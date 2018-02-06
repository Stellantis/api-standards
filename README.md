
# Groupe PSA API standards

Standards and guidelines for Groupe PSA REST APIs.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 [RFC2119].

# Table Of Contents

 - [HTTP Protocol](#http-protocol)
   - [Allowed HTTP Verbs List](#allowed-http-verbs-list) 
   - [Idempotent & Safe](#idempotent--safe)
     - [Idempotent](#idempotent)
     - [Safe](#safe)
     - [Summary Table](#summary-table)
   - [Status Codes](#status-codes)
     -  [Status Code Ranges](#status-codes-ranges) 
     -  [Allowed Status Codes List](#allowed-status-codes-list)
     -  [HTTP Method to Status Code Mapping](#http-method-to-status-code-mapping)
 - [Formatting](#formatting)
   - [JSON Types](#json-types)
     -  [JSON Primitive Types](#json-primitive-types)
   - [Common Types](#common-types)
	 -  [Internationalization](#internationalization)
	 -  [Date, Time and Timezone](#date-time-and-timezone)
	 -  [Date Time Common Types](#date-time-common-types)
   - [Error Handling](#error-handling)
	 - [Error Schema](#error-schema)
   - [Hypermedia](#hypermedia)
	 - [HATEOAS](#hateoas)
	 - [HAL](#hal)
	 - [HAL Examples](#hal-examples)
	 - [Paginating With HAL](#paginating-wit-hal)
 - [API Endpoint](#api-endpoint)
   - [URI Structure](#uri-structure)
   - [URI Naming Rules](#uri-naming-rules)
   - [URI Parameters](#uri-parameters)
     - [Path Parameters](#path-parameters)
     - [Query Parameters](#query-parameters) 
     - [Header Parameters](#header-parameters)
     - [Matrix Parameters](#matrix-parameters)
   - [Filtering, Selecting & Sorting ](#filtering-selecting--sorting)
     - [Filtering](#filtering)
     - [Selecting](#selecting)
     - [Sorting](#sorting)
   - [Pagination](#pagination)
     - [Offset/Limit pagination](#offsetlimit-pagination)
     - [Cursor Based Pagination](#cursor-based-pagination)
     - [Pagination & Hypermedias](#pagination---hypermedias) 
 - [Caching](#caching) 
   - [Caching Headers](#caching-headers)
   - [Caching Policies](#caching-policies)
     - [Caching Dynamic Data](#caching-dynamic-data)
     - [Caching Near-Static Data](#caching-near-static-data)
     - [No Cache](#no-cache)
   - [Caching Strategy](#caching-strategy) 
 - [Version Management](#version-management)
   - [Semantics](#semantics)
   - [Versioning Policies](#versioning-policies)
   - [Version Invocation](#version-invocation)
 - [Payload Conventions](#payload-conventions)

# HTTP Protocol

REST APIs are designed around the rich HTTP protocol.

## Allowed HTTP Verbs List

> **Normalize HTTP verbs usage to keep the API intuitive. Only the 5 verbs below SHALL BE used : use the right verb for the right operation to perform** 

|  Method | Action |
|--|--|
|  `GET` | Method **requests data** from the resource and should not produce any side effect |
|  `POST` | Method requests the server to **create** a resource in the database |
|  `PUT` | Method requests the server to **update** resource or **create** the resource, if it doesn’t exist |
|  `PATCH` | Method requests the server to **partially update** resource |
|  `DELETE` | Method requests that the resources, or its instance, **should be removed** from the database|


## Idempotent & Safe 

> **Keep idempotent and safe principles in mind when developing APIs. HTTP verbs that are idempotent MUST NOT lead to non idempotent operations and vice versa.** 


### Idempotent

An idempotent HTTP method is a HTTP method that **can be called many times without different outcomes**. It would not matter if the method is called only once, or ten times over. **The result MUST BE the same**. 

**Example**

    customer.age = 20 // idempotent

> This example  is **idempotent** : no matter how many times we execute this statement, the customer’s age will always be set to 20. 

    customer.age += 1 // non-idempotent

> The second line **isn't idempotent** : executing this 10 times will result
    in a different outcome as when running 5 times.
 
### Safe

Safe methods are HTTP **methods that do not modify resources**. Meaning if you use a safe HTTP method it MUST NOT create, update or delete resources. In addition, safe methods are methods that can be cached, prefetched without any repercussions to the resource.

**Example** 

    GET https://dev.api.inetpsa.com/project/customers/delete HTTP/1.1

> The following example is incorrect if this API call actually deletes the resource
> Notice the `GET` method and the `/delete` in the URI

### Summary Table

|               |Idempotent     |Safe       |
|---------------|-------------- |-----------|
|`GET`			|**Yes**		|**Yes**		|
|`POST`			|No			|No					|
|`PUT`			|**Yes**			|No					|
|`PATCH`			|No						|No			|
|`DELETE`			|**Yes**			|No					|

## Status codes 

### Status Code Ranges

When responding to API requests, the following status code ranges MUST be used.

|Range|Meaning|
|---|---|
|`2xx`|Returned when the request was successfully executed|
|`4xx`|Returned when a client error occurs while formulating the request. Usually these are problems with the request, the data in the request, invalid authentication or authorization, etc.|
| `5xx`| Returned when a server error occurs after a correctly formulated request was sent failed on the server. 5xx range status codes SHOULD NOT be utilized for validation or logical error handling. |

### Allowed Status Codes List

All REST APIs MUST use only the following status codes. APIs MUST NOT return a status code that is not defined in this table.

| Status Code | Description |
|-------------|-------------|
| `200 OK` | Standard response for successful HTTP requests. The actual response will depend on the request method used. In a GET request, the response will contain an entity corresponding to the requested resource. In a POST request the response will contain an entity describing or containing the result of the action. |
| `201 Created` | The request has been fulfilled and resulted in a new resource being created. Successful creation occurred (via either POST or PUT). Set the Location header to contain a link to the newly-created resource (on POST). Response body content may or may not be present. |
| `202 Accepted` | Used for asynchronous method execution to specify the server has accepted the request and will execute it at a later time. For more details, please refer [Asynchronous Operations](tobecompleted). |
| `204 No Content` | 	The server successfully processed the request, but is not returning any content. The 204 response MUST NOT include a message-body, and thus is always terminated by the first empty line after the header fields. |
| `300 Not Modified` | Used for conditional GET calls to reduce band-width usage. If used, must set the Date, Content-Location, ETag headers to what they would have been on a regular GET call. There must be no body on the response.|
| `401 Unauthorized` | The request requires authentication and none was provided. Note the difference between this and `403 Forbidden`. |
| `400 Bad Request` | 	The request could not be understood by the server due to malformed syntax. The client SHOULD NOT repeat the request without modifications.|
| `401 Unauthorized` | Similar to 403 Forbidden, but specifically for use when authentication is possible but has failed or not yet been provided |
| `403 Forbidden` | The server understood the request, but is refusing to fulfill it. It SHOULD describe the reason for the refusal in the entity. If the server does not wish to make this information available to the client, the status code 404 (Not Found) can be used instead |
| `404 Not Found` | Used when the requested resource is not found, whether it doesn't exist or if there was a 401 or 403 that, for security reasons, the service wants to mask |
| `406 Not Acceptable` | The server MUST return this status code when it cannot return the payload of the response using the media type requested by the client. For example, if the client sends an `Accept: application/xml` header, and the API can only generate `application/json`, the server MUST return `406`. |
| `500 Internal Server Error` | This is either a system or application error, and generally indicates that although the client appeared to provide a correct request, something unexpected has gone wrong on the server. A `500` response indicates a server-side software defect or site outage. `500` SHOULD NOT be utilized for client validation or logic error handling. |
| `503 Service Unavailable` | The server is unable to handle the request for a service due to temporary maintenance. |

### HTTP Method to Status Code Mapping

For each HTTP method, API developers SHOULD use only status codes marked as "X"  in this table. 

| Status Code | 200 Success | 201 Created |202 Accepted | 204 No Content | 400 Bad Request |  404 Not Found | 500 Internal Server Error |
|-------------|:------------|:------------|:------------|:---------------|:----------------|:---------------|:--------------------------|
| `GET`       | X         |               |               |                  | X             | X                      | X                       |
| `POST`      | X         | X                 |                 |  X             | X             |               | X                       |
| `PUT`       | X             |               | X           | X           | X             | X                       | X                       |
| `PATCH`     | X             |               |               | X           | X            | X                        | X                       |
| `DELETE`    | X             |               |               | X           | X             | X                            | X                       |



* `GET`: The purpose of the `GET` method is to retrieve a resource. On success, a status code `200` and a response with the content of the resource is expected. In cases where resource collections are empty (0 items in `/customers`), `200` is the appropriate status (resource will contain an empty `items` array).

* `POST`: The primary purpose of `POST` is to create a resource. If the resource did not exist and was created as part of the execution, then a status code `201` SHOULD be returned.
    * It is expected that on a successful execution, a reference to the resource created (in the form of a link or resource identifier) is returned in the response body.
    * Idempotency semantics: If this is a subsequent execution of the same invocation (including the [`Foo-Request-Id`](#http-custom-headers) header) and the resource was already created, then a status code of `200` SHOULD be returned. For more details on idempotency in APIs, refer to [idempotency](tobecompleted).
	* If a sub-resource is utilized ('controller' or data resource), and the primary resource identifier is non-existent, `404` is an appropriate response.


* `PUT`: This method SHOULD return status code `204` as there is no need to return any content in most cases as the request is to update a resource and it was successfully updated. The information from the request should not be echoed back. 

* `PATCH`: This method should follow the same status/response semantics as `PUT`, `204` status and no response body.
	* `200` + response body should be avoided at all costs, as `PATCH` performs partial updates, meaning multiple calls per resource is normal. As such, responding with the entire resource can result in large bandwidth usage, especially for bandwidth-sensitive mobile clients.

* `DELETE`: This method SHOULD return status code `204` as there is no need to return any content in most cases as the request is to delete a resource and it was successfully deleted.

    * As the `DELETE` method MUST be idempotent as well, it SHOULD still return `204`, even if the resource was already deleted. Usually the API consumer does not care if the resource was deleted as part of this operation, or before. This is also the reason why `204` instead of `404` should be returned.

# Formatting

## JSON Types

This section provides guidelines related to usage of [JSON primitive types](#json-primitive) as well as [commonly useful JSON types](#common-types) for address, name, currency, money, country, phone, among other things.

#### JSON Primitive Types

JSON Schema [draft-04][9] SHOULD be used to define all fields in APIs. As such, the following notes about the JSON Schema primitive types SHOULD be respected. Following are the guidelines governing use of JSON primitive type representations.
**String**
**Number**
**Array**
**Null** : APIs MUST NOT produce or consume `null` values.

## Common Types

Resource representations in API MUST reuse the [common data type](tobecompleted) definitions where possible. Following sections provide some details about some of these common types. Please refer to the [schema](tobecompleted) for more details.

### Internationalization

The following common types MUST be used with regard to global country, currency, language and locale.

* [`country_code`](schema/json/draft-04/country_code.json) : all APIs MUST use the [ISO 3166-1 alpha-2](http://www.iso.org/iso/country_codes.htm) two letter country code standard. For instance `FR` corresponds to France.
* [`currency_code`](schema/json/draft-04/currency_code.json) : currency type MUST use the three letter currency code as defined in [ISO 4217](http://www.currency-iso.org/). For quick reference on currency codes, see [http://en.wikipedia.org/wiki/ISO_4217](http://en.wikipedia.org/wiki/ISO_4217). For instance `EUR` corresponds to Euro.
* [`language.json`](schema/json/draft-04/language.json) : language type uses [BCP-47](https://tools.ietf.org/html/bcp47) language tag composed of :
 	* The ISO-639 alpha-2 language code (Optional) 
 	* The ISO-15924 script tag
 	* The ISO-3166 alpha-2 country code
 	For instance `fr-CA` maps to French as used in Canada (subtag here corresponds to language+region)
* [`locale.json`](schema/json/draft-04/locale.json) : locale type defines the concept of locale, which is composed of `country_code` and `language`. Optionally, IANA timezone can be included to further define the locale.
* [`province.json`](schema/json/draft-04/province.json) : province type provides detailed definition of province or state, based on [ISO-3166-2](https://en.wikipedia.org/wiki/ISO_3166-2) country subdivisions, with room for variant local, international, and abbreviated representations of province names. Useful for logistics, statistics, and building state pull-downs for on-boarding.

### Date, Time and Timezone

When dealing with date and time, all APIs MUST conform to the following guidelines.

* The date and time string MUST conform to the `date-time` universal format defined in section `5.6` of [RFC3339](https://www.ietf.org/rfc/rfc3339.txt)[21]

* All APIs MUST only emit [UTC](https://en.wikipedia.org/wiki/Coordinated_Universal_Time) time (aka [Zulu time](https://en.wikipedia.org/wiki/List_of_military_time_zones) or [GMT](https://en.wikipedia.org/wiki/Greenwich_Mean_Time)) in the responses.

* When processing requests, an API SHOULD accept `date-time` or time fields that contain an offset from UTC. For example, `2016-09-28T18:30:41.000+05:00` SHOULD be accepted as equivalent to `2016-09-28T13:30:41.000Z`. This helps ensure compatibility with third parties who may not be capable of normalizing values to UTC before sending requests. In such cases the offset SHOULD only be used to calculate the equivalent UTC time before it is persisted in the system (because of known platform/language/DB interoperability issues). A UTC offset MUST NOT be used to derive anything else. 
 
* The timezone string MUST be per [IANA timezone database](https://www.iana.org/time-zones) (aka **Olson** database or **tzdata** or **zoneinfo** database), for example *America/Los_Angeles* for Pacific Time, or *Europe/Berlin* for Central European Time.

* When expressing [floating](https://www.w3.org/International/wiki/FloatingTime) time values that are not tied to specific time zones such as user's date of birth, expiry date, publication date etc. in requests or responses, an API SHOULD NOT associate it with a timezone. The reason is that a UTC offset changes the meaning of a floating time value. For examples, all countries with timezones west of prime meridian would consider a floating time value to be the previous day.

### Date Time Common Types

The following common types MUST be used to express various date-time formats:

* [`date_time.json`](schema/json/draft-04/date_time.json) SHOULD be used to express an RFC3339 `date-time`.
* [`date_no_time.json`](schema/json/draft-04/date_no_time.json) SHOULD be used to express `full-date` from RFC 3339.
* [`time_nodate.json`](schema/json/draft-04/time_nodate.json) SHOULD be used to express `full-time` from RFC3339.
* [`date_year_month.json`](schema/json/draft-04/date_year_month.json) SHOULD be used to express a floating date that contains only the **month** and **year**. For example, card expiry date (`2016-09`).
* [`time_zone.json`](schema/json/draft-04/time_zone.json) SHOULD be used for expressing timezone of a RFC3339 `date-time` or a `full-time` field.

## Error Handling

The HTTP status codes in the `4xx` range indicate client-side errors (validation or logic errors), while those in the `5xx` range indicate server-side errors (usually defect or outage). However, these status codes and human readable reason phrase are not sufficient to convey enough information about an error in a machine-readable manner. To resolve an error, non-human consumers of RESTful APIs need additional help.

Therefore, APIs MUST return a JSON error representation that conforms to the [`error.json`](tobecompleted) schema. 

### Error Schema 

An error response following `error.json` as schema MUST include the following fields:

* `name`: A human-readable, unique name for the error. Should be mapped on the server side to insure consistency.
* `debug_id`: A unique error identifier generated on the server-side and logged for correlation purposes.
* `message`: A human-readable message, describing the error. This message MUST be a description of the problem NOT a suggestion about how to fix it. It is recommended that this value would be retrieved from the error catalog [`error_spec.json#message`][24] before sending the error response.
* `_links`: [HATEOAS](#hypermedia) links (in HAL format) specific to an error scenario. Use these links to provide more information about the error scenario and how to resolve it. 

An error response MUST NOT include any of the following information : 
* Internal code
* File path
* Stack trace

## Hypermedia

### HATEOAS

Hypermedia, an extension of the term [hypertext](https://en.wikipedia.org/wiki/Hypertext), is a nonlinear medium of information which includes graphics, audio, video, plain text and hyperlinks according to [wikipedia](https://en.wikipedia.org/wiki/Hypermedia). Hypermedia As The Engine Of Application State (`HATEOAS`) is a constraint of the REST application architecture described by Roy Fielding in his [dissertation](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm).

In the context of RESTful APIs, a client could interact with a service entirely through hypermedia provided dynamically by the service. A hypermedia-driven service provides representation of resource(s) to its clients to navigate the API dynamically by including hypermedia links in the responses. 

**Hypermedias are not mandatory**, however if you have to use it then do it using [HAL](#hal).

### HAL

HAL is a simple format that gives a consistent and easy way to hyperlink between resources in your API. For detailed information on HAL refer to this [link](http://stateless.co/hal_specification.html). In short, HAL provides a set of conventions for expressing hyperlinks in either JSON or XML. 

For simplicity reasons, all APIs using HATEOAS along with HAL formatting MUST only use the following properties. 
* `_links` : This property contains links related to the current object. For example, a `customer` object using HAL will contain links to the vehicles owned by the user.
* `_embedded` : This property contains commonly related objects to the current object. This property SHALL BE empty.

#### HAL Examples

```
GET /cars/9837127  HTTP/1.1
```


**Sample schema for a car object without using HAL**

```
{
  "id" : 9837127,
  "model" : "DS3",
  "createdAt": "2017-08-10T13:46:39.652+0000",
  "updatedAt": "2017-08-10T13:46:39.652+0000",
}
```

**Sample schema for a car object using HAL**

```
{
  "_links": {
    "self": {
      "href": "/cars/9837127"
    }
    ...
  },
  "id" : 9837127,
  "model" : "DS3",
  "createdAt": "2017-08-10T13:46:39.652+0000",
  "updatedAt": "2017-08-10T13:46:39.652+0000",
  "_embedded": {
    "owner": {
      "_links": {
        "self": {
          "href": "/customers/0981233"
        }
      }, 
      "id": "0981233",
      "name": "John Doe"
    }
}
```

For additional examples, please refer to this [list of public hypermedia APIs using HAL](https://github.com/mikekelly/hal_specification/wiki/APIs)

Example of existing PSA APIs using HAL : [Connected Car 2.0.0](https://developer-preprod.psa-peugeot-citroen.com/inc/node/644)

#### Paginating With HAL

Pagination can be made easier for the consumer by returning preformatted URIs to fetch the **next** and **previous** pages. All APIs using hypermedia MUST perform pagination using HAL.

```
GET /cars/  HTTP/1.1
```

**Sample schema for a list of car objects using HAL**
*Using Offset/Limit pagination : if an API uses cursors, include hash instead*

```json 

{
  "_links": {
    "self": {
      "href": "/cars"
    },
    "first": {
      "href": "/cars?offset=0"
    },
    "next": {
      "href": "/cars?offset=10&limit=10"
    },
    "prev": {
      "href": "/cars?offset=0"
    },
    "last": {
      "href": "/cars?offset=90&limit=10"
    }
  },
  "total": 100,
  "_embedded": {
    "trips": [{...}]
  }
}     
```
 

# API Endpoint

> **URIs must be normalized and construction rules should be followed at the Group level**

## URI Structure

APIs at PSA must always have the following components : 
 - **protocol** : must always be `https`
 - **environment** : could be `prod`, `preprod` or `dev`
 - **api** : specifies that this specific URI is an API endpoint
 - **domain** : must always point to `inetpsa.com`
 - ***classification** : the domain it belongs to (usually a business unit)
 - **api name** : the name of the API
 - **version** : must only contain the major version (`v1` instead of `v1.0`)
##### \* classification : may not always be needed in the case of enterprise APIs (as opposed to application APIs)

**Example** : API called `factory` under the `manufacturing` domain. This specific endpoint addresses `v1` of the API and accesses the ressource `customers` (filtering by name `John`) 

    https:// api.mpsa.com/manufacturing/v1/factory/customers?name=john

## URI Naming Rules

> **The URI or API endpoint must not contain actions or verbs. URIs must contain the plural form of resources and the HTTP method should define the kind of action to be performed on the resource.**

Let’s consider a **Customer** resource on which we would like to perform several actions such as **list**, **update**, **delete** and **promote**. We could define our API in a way that each URI corresponds to an operation as follows :

 - `GET `**`/getCustomers`** : **return** all customers 
 - `GET `**`/deleteCustomer`** : **delete** a customer (using `GET` here because we are not sending any data)
 - `GET `**`/promoteCustomer`** : **promote** a customer (using `GET` here because we are not sending any data)
 - `POST `**`/createCustomer`** : **create** customer according to the request **body**
 - `POST `**`/updateCustomer`** : **update** customer according to the request **body**

**What is wrong with this implementation ?**
> **The URI or API endpoint must not contain actions or verbs. URIs must contain the plural form of resources and the HTTP method should define the kind of action to be performed on the resource.** 
> 
> There will be as many endpoints as there are operations to perform and many of these endpoints will contain redundant actions. **This leads to unmaintainable APIs**

**What is the correct way ?**

If we do not use verbs in URIs, how can we tell the server about the actions to be performed on a given resource (Customer for instance) ? **This is where the HTTP methods (or verbs as seen before) play the role**. Let's consider the same example following **REST** rules :

 - **`GET`**`/customers` : **return** all customers 
 - **`GET`**`/customers/456` : **return** customer with id #456 
 - **`POST`**`/customers` : **create** customer according to the request **body**
 - **`DELETE`**`/customers/456` : **delete** customer with id #456
 - **`UPDATE`**`/customers/456` : **update** customer with id #456
 - **`GET`**`/companies/123/customers/456` : **return** customer with id #456 in company with id  #123 (if we have a resource under a resource)

Great, but **how do we promote** (or perform any other specific actions using REST) ? This is where things get tricky in REST. Since we cannot use verbs in URIs and there is no HTTP method to promote a customer, then how do we do it ? 

There are multiple ways of working around this, choose the one that makes sense to you and to your application. Ask yourself what the **promote** operation does : does it update a field in the user object ? Does it create a new record ? Does it update some kind of resource associated to the user ? Once you have defined the behavior, you can either :

 1. RESTful way : `PATCH /customers/456` which will partially update the customer object to reflect the promotion (given that your models are built this way)
 2. RESTful way : `PATCH /customers/456/grade` which will partially update a resource called **Grade** that is associated to the user 
 3. Non-RESTful way though it is clean : `POST /customers/456/promote` this contains a verb and isn't considered RESTful though it is semantically clean and explicit. 

**What is wrong with this implementation ?**
> APIs are more **precise** and **consistent**. They are **easier to understand** both for humans and machines and thus are **maintainable**. 

## URI Parameters 

> **Follow common principles to choose whether to use a path a query or a header parameter to perform specific actions.**

### Path Parameters

`/customers/{id}` : variable parts of a URI path, typically used to point to a specific resource within a collection. Each path parameter must be substituted with an actual value when the client makes an API call.

**Example** : `GET /customers/2097138971` used to target a specific customer resource with ID `2097138971` 

### Query Parameters

`/customers?name=value` : appear at the end of the request URI after a question mark (?) separated by ampersands (&). Be careful when using sensitive/private query parameters as the URI is not encrypted over HTTPS : **pass in HEADER instead**.

**Example** : `GET /customers?name=John` used to target customers with name `John` 

### Header Parameters 

`X-MyHeader : Value` : an API call may require that custom headers be sent with an HTTP request. Some sample headers are `Content-Type`, `Accept`, `Authorization` etc.

**Example** : `curl /customers/456 -H "Content-Type: application/xml"` used to retrieve customer  `456` in `XML` format

### Matrix Parameters

`/company;name=value/customers` : apply filters to a particular path element. **Matrix params are rarely used** but can be useful in some particular use case, and thus should be kept in mind.

**Example** : `GET /companies;name=PSA/customers` used to target all customers from company `PSA` 

## Filtering, Selecting & Sorting 

>  **It is recommended to use a GET request for reading operations by specifying filters directly in the URL. As opposed to doing a POST giving a request body with the filters.**

### Filtering

>  Filtering allows consumers to only fetch data that matches their conditions.

 - **Single-Value / Single-Criteria Filtering** 
 `GET /customers?name=John` : get all customers whose names is `John`
 
 - **Single-Value / Multiple-Criteria Filtering**
 `GET /customers?name=John&age=35`  : get all customers whose names is `John` **and** age is `35`
 
 - **Multiple-Value Filtering**
 `GET /customers?name=John,Mark` : get all customers whose name is `John` **or** `Mark`
 
### Selecting 

>  Selecting fields allows consumers to only fetch data they need **within a resource**. This mechanism is really useful when dealing with bad internet connection. To select fields simply specify them in the URI.

**Example** : `GET /customers?fields=id,address(city)` only fetch `id` and `address` with `city` only of customer `456` :

```json 
{
	"id": 19083974, 
	"address":{ 
		"city": "Paris",
	}, 
}
```

### Sorting

>  Sorting allows consumers to fetch data in the order they need. The API will respond with the data in the order specified in the request.

**Example** : `GET /customers?sort=name` fetch all customers and sort by `name` (by default sorting is ascending)

**Example** : `GET /customers?sort=name,age` fetch all customers and sort by `name` and `age`

**Example** : `GET /customers?sort=age&desc=age` fetch all customers and sort by `age` in `desc` order 


## Pagination

### Offset/Limit pagination

> This pagination technique is the easiest to implement and **SHALL BE used by default**. You MUST choose a default number of items to be returned  (that would mean a default *limit*).

    GET /customers?offset=5&limit=8 
    
    [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14]
                 ^----------------------^
                 offset	                limit
                 
    Returns [4, 5, 6, 7, 8, 9, 10, 11] 
     
**Request**

    GET /customers/  HTTP/1.1

**Response**

```json 
{
	"data": [
		{"name": "John Doe", "..." },
		{"name": "Mark Long", "..." },
		{"name": "Helen Ping", "..."},
		{"name": "Chris Temp", "..."},
		{"name": "Simon Hillman", "..."}
	],
	"count": 5,
	"offset" : 0,
	"total" : 200 
}
```
**Drawbacks of offset/limit**

For mostly static content where items don’t move between pages frequently, offset/limit is great. But it isn't always the case,  specifically because items are sometimes added and removed while the user is navigating to different pages. Here's what can happen when using offset/limit on dynamic data  :

 1. **Displaying the same item twice**. This can happen if a new item was added at the top of the list, causing the skip and limit approach to show the item at the boundary between pages twice.

    **Example** : let's consider a list of 6 items **at time t**. The user performs a request to fetch the first page (`GET /items?limit=3`). Now let's say a new item was added at the top of the list and the user performs another request to fetch the second page (`GET /items?limit=3&offset=3`). It the state hadn't changed, he would have gotten the second page with items `[4, 5, 6]`, though he will get a response containing items `[3, 4, 5]`, item 3 being a duplicate.
 	<img src="https://raw.githubusercontent.com/GroupePSA/api-standards/master/examples/pagination/paginating-dynamic-data.png" width="600">
 
 2. **Skipping an item**. Similarly to the example above, if we **remove** an item from the list we'll skip an item. 
 
Though offset/limit can be used in 90% of the cases, it also means that for certain kinds of APIs, using this technique doesn’t make sense because the set of data and the boundaries between loaded sections is constantly changing (i.e. real time data). **In this case, use cursor based pagination**.

### Cursor-based pagination

> This pagination technique is is bit more complicated to implement though it **MUST BE be used when dealing with real time data.** 

In the example above, if could have specified the exact position in the list we want to begin with, and the number of items we wanted to fetch, we would not have ran into these issues. With this technique, no matter how many items were removed or added to the top of the list , we still have a constant pointer to the exact position where we left off. This pointer is called a **cursor**. A cursor is a unique object identifier that sets the starting point of the pagination until the specified limit. 

**Request**

    GET /customers/  HTTP/1.1

**Response**

```json 
{
	"data": [
		{"name": "John Doe", "..."},
		{"name": "Mark Long", "..."},
		{"name": "Helen Ping", "..."},
		{"name": "Chris Temp", "..."},
		{"name": "Simon Hillman", "..."}
	],
	"before" : "NDMyNzQyODI3OTQw1j3",
	"after" : "MTAxNTExOTQ1MjAwNzI", 
	"count": 5,
	"total" : 200,
}
```

**How to choose a cursor ?** Well, first off,  cursors MUST BE :
 - **Unique** :  if not unique, conflicts may occur while setting the starting point of the pagination 
 - **Sortable** : to insure consistency and allow sorting 
 - **Stateless** : if not stateless, a cursor might not be available by the time it is used

The ideal cursor is an **encoded timestamp** of the item, since it is satisfies all the above conditions. In the case where you do not have a timestamp on your items, you have to find the right parameter that satisfies all the above criterias : it could be a hash of an `ID`, `email` etc.

### Pagination &  Hypermedias

See [Paginating With HAL](#paginating-with-hal)

## Caching

The ability to cache and reuse previously fetched resources is a critical aspect of optimizing for performance.  Every browser comes with an implementation of an HTTP cache. Developers MUST ensure that each server response provides the correct HTTP header directives to instruct the browser on when and for how long the browser can cache the response. 

### Caching headers

This table summarizes the most used caching techniques :

|Header  | Definition |
|--|--|
| `Cache-Control : no-store` | **Disallows the browser and all intermediate caches** from storing any version of the returned response. Every time the user requests this asset, a request is sent to the server and a full response is downloaded. This is equivalent to no caching at all. |
| `Cache-Control : private` | Disallows any intermediate cache to cache the response. For instance, a user's browser can cache an HTML page with private user information, but a CDN can't cache the page. |
| `Cache-Control : max-age` | Specifies the maximum time in seconds that the fetched response is allowed to be reused from the time of the request. For instance if `Cache-Control : maxe-age=60` header is set, any call made within 60 seconds will get a cached response.  |
| `Last-Modified` | Contains the date and time at which the origin server believes the resource was last modified. It is used as a validator to determine if a resource received or stored is the same. Must be validated against `If-Modified-Since` & `If-Unmodified-Since` headers. |
| `Expires` | Tells the client when the resource is going to expire. Usually has a date value that specifies from when the resource will be considered stale (possibly out-of-date), and so it needs to be re-validated. For instance if `Expires : Wed, 31 Dec 2018 07:28:00 GMT` header is set, any request made to the server before the specified date will return a cached response.  |
| `Etag` | The ETag is a unique identifier for your response: it is used on conditional requests using `If-None-Match` header. Note: you have to make sure the server provides a validation token (ETag) service|

### Caching policies

 API developers and designers MUST define and configure the appropriate per-resource settings, as well as the overall "caching hierarchy." : try to find a balance between good and bad reactivity depending on the resource type. API designers MUST determine the best cache hierarchy for their API : the combination of resource URLs with content fingerprints (Etag) and short or no-cache lifetimes allows you to control how quickly the client picks up updates.

#### Caching dynamic data

You cannot cache for long periods of time, such as: days, hours or sometimes even minutes because data becomes stale too quickly. That doesn't mean, however, that such data shouldn't be cached at all. When caching resources for short periods of time you should be using HTTP caching instructions that do not rely on shared understanding of time, such as `Cache-Control : max-age`, `Expires` and `ETags`

> Dynamic data is data that is often changing  includes GPS location, battery voltage, tire pressure etc. 

#### Caching near-static data

If you are caching resources (API responses) for sufficiently long periods of time (hours, days, potentially: months) you usually do not have to worry about the issues related to date-time-based caching. For facilitating caching of near-static data, you could use two approaches:  

 - `ETags` that do not rely on shared agreement on time
 - `Last-Modified` : header that is date-time-centric

> Near static data includes country codes, static images like icons, localization labels etc.

#### No cache

You can choose not to cache response, in real-time systems for examples, however this comes with great responsibility. The ability to cache and reuse previously fetched resources is a critical aspect of optimizing for performance. Even for dynamic data, caching can tremendously increase performance.

> Real time data includes car speed, precise GPS location etc. 

### Caching strategy 

There's no one best cache strategy. API designers and developers MUST take several parameters into account while working on caching strategy : 
 - **Type of data served** : as we've seen before, dynamic and static data will not have the same caching policy.
 - **Traffic patterns** : caching was designed to optimize performances by limiting server calls - if your API is used be a few consumers, caching might not be as critical as if your API was used by millions of people.
 - **Application-specific requirements for data freshness** : caching is application specific, meaning that a resource might not be cached the same way across all applications.
 *example* : a GPS location might not be cached the ame way in :  
   - Real-time applications (Waze for instance) : data has ~1sec or less lifetime
   - A car dealership application to know approximately where the car is located : data has a ~1min lifetime

Developers SHALL refer to this diagram when deciding on which caching technique to choose :

<img src="https://raw.githubusercontent.com/GroupePSA/api-standards/master/examples/caching/caching-decision-tree.png" width="700">

# Version Management

This section describes how to version APIs and most specifically what are the invocation rules.

## Semantics 

A version is written as follows : 
 - **major version** : non-retro compatible updates
 - **minor version** : retro compatible updates
 - **patch version** : mostly bug fixing
\**Retro compatibility : updates that are still compatible with previous versions of the API
See full detail on versioning on slide X*

## Versioning Policies

API’s are versioned products and MUST adhere to the following versioning principles.

1. API specifications MUST follow the versioning scheme where where the `v` introduces the version, the major is an ordinal starting with `1` for the first LIVE release, and minor is an ordinal starting with `0` for the first minor release of any major release.
3. API endpoints MUST only reflect the major version. (see below)
5. A minor API version MUST maintain backward compatibility with all previous minor versions, within the same major version.
6. A major API version MAY maintain backward compatibility with a previous major version.

## Version Invocation 

There are two main ways to version an API (see next slide), either invoke the version in the :
 - **URL** : APIs MUST invoke the version in the url of the request : 
    ```
    https://api.mpsa.com/manufacturing/factory/customers/456
    ```

 	 - Cleaner approach, less error prone
	 - Consumers don’t need to handle HTTP headers (which can be tough for some clients)
	 - More explorable : easier testing with a simple browser

 - **HTTP Header** : APIs MUST NOT invoke the version in the header as it is more prone to errors and not developer friendly 
	``` 
	Accept: application/json;version=1
    https://api.mpsa.com/customers/456
    ```  


# License

<a rel="license" href="http://creativecommons.org/licenses/by/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/4.0/">Creative Commons Attribution 4.0 International License</a>.
