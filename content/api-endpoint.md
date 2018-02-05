
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

## URI Rules

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


### Pagination

#### Offset/Limit pagination

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
 <img src="https://raw.githubusercontent.com/GroupePSA/api-standards/master/examples/images/pagination-example-addition.png" width="600">
 
 2. **Skipping an item**. Similarly to the example above, if we **remove** an item from the list we'll skip an item. 
 
Though offset/limit can be used in 90% of the cases, it also means that for certain kinds of APIs, using this technique doesn’t make sense because the set of data and the boundaries between loaded sections is constantly changing (i.e. real time data). **In this case, use cursor based pagination**.

#### Cursor-based pagination

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

#### Pagination and  hypermedias

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
