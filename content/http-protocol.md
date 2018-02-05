# HTTP Protocol

REST APIs are designed around the rich HTTP protocol.

## Verbs

> **Normalize HTTP verbs usage to keep the API intuitive. Only the 5 verbs below SHALL BE used : use the right verb for the right operation to perform** 

|  Method | Action |
|--|--|
|  `GET` | Method **requests data** from the resource and should not produce any side effect |
|  `POST` | Method requests the server to **create** a resource in the database |
|  `PUT` | Method requests the server to **update** resource or **create** the resource, if it doesn’t exist |
|  `PATCH` | Method requests the server to **partially update** resource |
|  `DELETE` | Method requests that the resources, or its instance, **should be removed** from the database|


## Idempotent vs. Safe 

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
