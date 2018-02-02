
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
