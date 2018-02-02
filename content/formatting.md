
# JSON Format

> **APIs must use JSON formatted input and output.**

**Example** 
```json
"user" : {
    "id": 19083974,					// Number
    "name": "John Doe",				// String
    "email": "john.doe@psa.com",
    "age": 19,
    "roles": ["admin", "editor"],	// Array
    "disabled" : false,				// Boolean
    "address":{						// Object
        "city": "Paris"
	}
}
```

## Object Key

> **A key or attribute must be unique for any given level of data**

 - **Bad example**
	 ```json
	GET /customers/19083974  HTTP/1.1

    "customer" : {
	    "id": 19083974,
	    "name": "John",
	    "name": "Doe",	// Using name twice on the same level of data is forbidden
	} 
	```
 - **Good example**
	 ```json
    GET /customers/19083974  HTTP/1.1
    
    "customer" : {
	    "id": 19083974,
	    "name": "John Doe",
	    "address": {
		    "name": "Home",	// This is allowed since it belongs to address and not customer
		    "city": "Paris"
	    }
	} 
	```

> **Respect the informational context by using clear and explicit naming and use generic terms reusable in a different context than the application it  was first designed for.**

 - **Bad example**
	 ```json
	 GET /customers/19083974  HTTP/1.1
	 
    "customer" : {
	    "customeId": 19083974,	// Bad naming since a customer could be a different object in a different context
	    "home_address" : {			// Not developer friendly as other projects might call it differently
		    "city":"Paris"
	    }
	} 
	```
 - **Good example**
	 ```json
	 GET /customers/19083974  HTTP/1.1
	 
    "customer" : {
	    "id": 19083974,		// Good generic naming : can be used in any context
	    "address": {		// Good generic naming
		    "name": "Home",	// Better naming as
	    }
	} 
	```

## Object Value

> **JSON is a loosely typed format.  Nonetheless, formats must be  standardize for any JSON objects (see payload conventions)**

In JSON, values must be one of the following data types:
 - a string :  `{ "name":"John" }`
 - a number `{ "age": 19 }`
 -  an object (JSON object) `"address" : { "name": "Home" }` 
 - an array `"roles" : ["admin", "editor"]`
 - a boolean  `"disabled" : false`
 - null `"address" : null`

> **Null objects must never be included in a JSON response because it decreases payload size and facilitates API/resource evolution**

 - **Bad example**
	 ```json
	GET /customers/19083974  HTTP/1.1
	
    "customer" : {
	    "id": 19083974, 
	    "address" : null	// Do not return if null
	} 
	```
 - **Good example**
	 ```json
	 GET /customers/19083974  HTTP/1.1
	 
    "customer" : {
	    "id": 19083974,	// Good generic naming : can be used in any context
	} 
	```
## Handling Multiple Formats

> **In the case where APIs serve more than one content type, consumers must specify type they need using Accept header**

APIs may take as **input or output any content type needed** (an audio file, a specific text format etc.)  Each API endpoint **specifies what response type it outputs and what it consumes** in cases where it applies (for POST requests for instance) :

```
GET /URL  HTTP/1.1	 
Accept : text/*
```
```
HTTP/1.1 200 OK	 
Accept : text/html
```

See complete list of [MIME Types](https://developer.mozilla.org/fr/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Complete_list_of_MIME_types)
