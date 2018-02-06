# Groupe PSA API standards

Standards and guidelines for Groupe PSA REST APIs.

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
   - [JSON Formatting](#json-formatting)
     - [Object Key](#object-key)
     - [Object Value](#object-value)
   - [Handling Multiple Formats](#handling-multiple-formats) 
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

## JSON Formatting

> **APIs MUST use JSON formatted input and output.**

**Example** 
```json
"user" : {
    "id": 19083974, 
    "name": "John Doe",
    "email": "john.doe@psa.com",
    "age": 19,
    "roles": ["admin", "editor"],
    "disabled" : false,
    "address":{
        "city": "Paris"
	}
}
```

### Object Key

> **A key or attribute MUST BE unique for any given level of data**

 - **Bad example**
	 ```json
	GET /customers/19083974  HTTP/1.1

    "customer" : {
	    "id": 19083974,
	    "name": "John",
	    "name": "Doe",	"Bad : using name twice on the same level of data is forbidden"
	} 
	```
 - **Good example**
	 ```json
    GET /customers/19083974  HTTP/1.1
    
    "customer" : {
	    "id": 19083974,
	    "name": "John Doe",
	    "address": {
		    "name": "Home",	"Good:  this is allowed since it belongs to address and not customer"
		    "city": "Paris"
	    }
	} 
	```

> **You MUST respect the informational context by using clear and explicit naming and use generic terms reusable in a different context than the application it  was first designed for.**

 - **Bad example**
	 ```json
	 GET /customers/19083974  HTTP/1.1
	 
    "customer" : {
	    "customeId": 19083974,	"Bad : since a customer could be a different object in a different context"
	    "home_address" : {		"Bad : not developer friendly as other projects might call it differently"
		    "city":"Paris"
	    }
	} 
	```
 - **Good example**
	 ```json
	 GET /customers/19083974  HTTP/1.1
	 
    "customer" : {
	    "id": 19083974,		"Good: generic naming, can be used in any context"
	    "address": {		"Good : generic naming"
		    "name": "Home",	"Good : better naming strategy"
	    }
	} 
	```

### Object Value

> **JSON is a loosely typed format.  Nonetheless, formats MUST BE standardize for any JSON objects (see payload conventions)**

In JSON, values must be one of the following data types:
 - a string :  `{ "name":"John" }`
 - a number `{ "age": 19 }`
 -  an object (JSON object) `"address" : { "name": "Home" }` 
 - an array `"roles" : ["admin", "editor"]`
 - a boolean  `"disabled" : false`
 - null `"address" : null`

> **Null objects MUST NOT be included in a JSON response because it decreases payload size and facilitates API/resource evolution**

 - **Bad example**
	 ```json
	GET /customers/19083974  HTTP/1.1
	
    "customer" : {
	    "id": 19083974, 
	    "address" : null	"Bad : do not return if null"
	} 
	```
 - **Good example**
	 ```json
	 GET /customers/19083974  HTTP/1.1
	 
    "customer" : {
	    "id": 19083974,
	} 
	```
## Handling Multiple Formats

> **In the case where APIs serve more than one content type, consumers MUST specify type they need using Accept header**

APIs MAY take as **input or output any content type needed** (an audio file, a specific text format etc.)  Each API endpoint **specifies what response type it outputs and what it consumes** in cases where it applies (for POST requests for instance) :

```
GET /URL  HTTP/1.1	 
Accept : text/*
```
```
HTTP/1.1 200 OK	 
Accept : text/html
```

See complete list of [MIME Types](https://developer.mozilla.org/fr/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Complete_list_of_MIME_types)



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

    https:// api.mpsa.com/manufacturing/factory/v1/customers?name=john

## URI Naming Rules

> **The URI or API endpoint must not contain actions or verbs. URIs must contain the plural form of resources and the HTTP method should define the kind of action to be performed on the resource.**

Let’s consider a **Customer** resource on which we would like to perform several actions such as **list**, **update**, **delete** and **promote**. We could define our API in a way that each URI corresponds to an operation as follows :

 - `GET /`**`getCustomers`** : **return** all customers 
 - `GET /`**`deleteCustomer`** : **delete** a customer (using `GET` here because we are not sending any data)
 - `GET /`**`promoteCustomer`** : **promote** a customer (using `GET` here because we are not sending any data)
  - `POST /`**`createCustomer`** : **create** customer according to the request **body**
  - `POST /`**`updateCustomer`** : **update** customer according to the request **body**

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

If your API uses [hypermedias](tobecompleted) (HAL), pagination can be made easier for the consumer by returning preformatted URIs to fetch the **next** and **previous** pages like so : 

 **Using offset/limit**

```json 
GET /customers/  HTTP/1.1

{
	"data": [
		{"name": "John Doe", "..."},
		{"name": "Mark Long", "..."},
		{"name": "Helen Ping", "..."},
		{"name": "Chris Temp", "..."},
		{"name": "Simon Hillman", "..."}
	],
	"count": 25,
	"offset" : 0,
	"total" : 200,
	"_links": {
		"previous" : "http://api.mpsa.com/customers?offset=0&limit=25",
		"next" : "http://api.mpsa.com/customers?offset=25&limit=25"
	}
}
```
 **Using after/before cursor**

```json 
GET /customers/  HTTP/1.1

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
	"count": 25,
	"total" : 200,
	"_links": {
		"previous" : "http://api.mpsa.com/customers?before=NDMyNzQyODI3OTQw1j3&limit=25",
		"next" : "http://api.mpsa.com/customers?after=MTAxNTExOTQ1MjAwNzI&limit=25"
	}
}
```

## Caching


### Caching headers

The ability to cache and reuse previously fetched resources is a critical aspect of optimizing for performance.  Every browser comes with an implementation of an HTTP cache. Developers only need to ensure that each server response provides the correct HTTP header directives to instruct the browser on when and for how long the browser can cache the response. The following table provides information regarding caching techniques :

|Header  | Definition |
|--|--|
| `Cache-Control : no-store` | **Disallows the browser and all intermediate caches** from storing any version of the returned response. Every time the user requests this asset, a request is sent to the server and a full response is downloaded. This is equivalent to no caching at all. |
| `Cache-Control : private` | Disallows any intermediate cache to cache the response. For instance, a user's browser can cache an HTML page with private user information, but a CDN can't cache the page. |
| `Cache-Control : max-age` | Specifies the maximum time in seconds that the fetched response is allowed to be reused from the time of the request. For instance if `Cache-Control : maxe-age=60` header is set, any call made within 60 seconds will get a cached response.  |
| `Last-Modified` | Contains the date and time at which the origin server believes the resource was last modified. It is used as a validator to determine if a resource received or stored is the same. Must be validated against `If-Modified-Since` & `If-Unmodified-Since` headers. |
| `Expires` | Tells the client when the resource is going to expire. Usually has a date value that specifies from when the resource will be considered stale (possibly out-of-date), and so it needs to be re-validated. For instance if `Expires : Wed, 31 Dec 2018 07:28:00 GMT` header is set, any request made to the server before the specified date will return a cached response.  |
| `Etag` | The ETag is a unique identifier for your response: it is used on conditional requests using `If-None-Match` header. Note: you have to make sure the server provides a validation token (ETag) service|

### Caching policies

There's no one best cache policy. You must take several parameters into account while working on caching strategy : traffic patterns, type of data served and application-specific requirements for data freshness. API developers and designers MUST define and configure the appropriate per-resource settings, as well as the overall "caching hierarchy.“ : try to find a balance between good and bad reactivity depending on the resource type. API designers MUST determine the best cache hierarchy for their API : the combination of resource URLs with content fingerprints (Etag) and short or no-cache lifetimes allows you to control how quickly the client picks up updates.

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

Graph


# License

<a rel="license" href="http://creativecommons.org/licenses/by/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/4.0/">Creative Commons Attribution 4.0 International License</a>.
