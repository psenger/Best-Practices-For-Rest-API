
# Best Practices for Building REST Apis

Written by Philip A Senger

[philip.a.senger@cngrgroup.com](mailto:philip.a.senger@cngrgroup.com) | mobile: 0404466846 | [CV/Resume](http://www.visualcv.com/philipsenger) | [blog](http://www.apachecommonstipsandtricks.blogspot.com/) | [LinkedIn](http://au.linkedin.com/in/philipsenger) | [twitter](http://twitter.com/PSengerDownUndr) | [keybase](https://keybase.io/psenger)

Ive been building API for Service Oriented Applications for over a decade. Many API design opinions and guidelines I have found are academic in nature and less real world or practical. My goal with this article is to describe the best practices for a pragmatic API approach based on my experience and as a framework for my thoughts. Ive found the following items to be the key to the success of my systems The Human Aspect, Security and Permissions, Implementation.
	
## The Human Aspect

Service Oriented Architecture is a relatively new concept. "New" meaning with in the last 25 years. Before that we had Remote Procedures Calls which gained popularity with the advent of Common Object Request Broker Architecture (CORBA) and the father of electronic data interchange, EDI. After these technologies there was SOAP and a variety of other technologies. Given the dynamic nature of this landscape and added complexity, it is easy to understand that the biggest barrier to adoption and therefore implementation would be resistance by people. This resistance can come in the form of laziness, being overloaded with work, and pure anti social and stubbornness. 

1. Adoption
2. Standards and Consistency
3. Self Discovery
4. Intuitive

### Adoption

The number one challenge I have consistently had is, adoption. While you might think it makes sense to add a layer of abstraction, apply single responsibility principal to each layer, or create a stable elastic system to your total solution landscape, you will find that not everyone will agree. 

This is usually founded in either the fact that it will inherently become more complex, or the users of your system enjoy the nature of traditional single layer systems. In either case, you will be forced to nurture the idea and advocate the adoption.

Although I can't guarantee adoption, there are several pillars needed for adoption. Easy of use, Documentation, Stability, and Assurance.

#### Easy of use

Adoption is always blocked when you can't sell the concept. Besides having a elevator pitch ready, you need to remove all barriers preventing adoption. These tend to be.

1. Lack of documentation
2. Lack of standards
3. Lack of convention
4. Difficulty in gaining access to resources.

Now, how can you do these things in a faster way ? I use Swagger


#### Documentation

Documentation is one of the pillars of success. Lack there of implies you haven't thought the system through or its ramification. What kind of documentation should you create? API specifications, user stories, and examples.

Managing relevance of the documentation is difficult and requires discipline. Use tools like [Swagger.io](http://Swagger.io) to build a design first approach. The added benefit is this markdown can be used to create Client and Server stubs as well as create interactive websites. The downside is this tool is discounted from the code ( unlike Java Annotation ). Because Swagger can create documentation and examples this tool can be very useful.

#### Stability

#### Assurance

Creating tests should be part of every developers daily activity. It provides assurance that the goal was accomplished and meets the needs of the specification. 

### Standards and Consistency

Keep your services designed to serve **Resources** otherwise you risk the chances that your Services will become a remote procedure call. REST is Representational State Transfer, not RPC or Remote Procedure Call. 

#### Naming convention

The naming convention is very important because it implies consistency. Convention. The naming convention should not leak implementation details. It should relate to resources.

##### Nouns

End points should be nouns, such as **Books** or **Users**. Names that are verbs or adjectives are a bad idea such as **DoPayRoll** or **PostFin**

##### Versioning

Resources should be versioned. There are two good ways to do this. both have advantages and disadvantges. 

In the base of the url works best for the API team. This works in the base of the url because it is easy to stand up a server to represent that endpoint behind a firewall or Load Balancer, and not convolute your code with cross concerns of versioning. Unfortunately, the consumer of the API will have to be flexible enough to redeploy if the need to change in accordance. Refer to [http://semver.org/](http://semver.org/) for the versioning technique. 

URL based endpoints, for example:

```
/v2/books
```

Keep in mind that semantic versioning works, but the major number implies incompatibility. So as a convention, use the major number for endpoints. Avoid names for the versions such as "2.14.2". As you can see this will ultimately become a nightmare to manage.

```
/v2.14.2/books
```

One problem with putting the version in the url is applications will need to release in tandem. Avoid this with a header ``Accept-Version`` and the version number as the value. This will cause complexity in the API. 

Restify has a good technique of mapping versions of the APIs to functions [Versioned Routes](http://restify.com/#versioned-routes) 

##### Plural

Resources ( endpoints ) should always have a plural name in the end point. for example

```
/books
/users
```
Avoid the following singular names 

```
/user
/book
```

Making the name singular such as **/book** sounds as if you are going to create a single endpoint to perform operations on a `book`. Rather, you should stick with the plural name **/books** and make use of the unique id of the book for singular operations on the endpoint. for example **/books/1234**

###### CRUD

Create Read Update Delete should always be represented through the HTTP Verbs.

### Self Discovery

Self discovery implies that links within the model coupled with meta data will make discovery of other endpoints and additional supporting data easy, helpful, and data driven. 

Links to the details of a entity should be also provided here.   

This feature will aid in promoting the adoption of the system.

When you have a hrefs in the model, always include a **rel** value. This is one of the few meta data values that is widely adopted. I have found it better to actually include a directory or listing of all the **rel** types in the system. This allows the consumer developers to create global decorators, controllers, views, an modesl as opposed to "one off designs". I will expand on this idea a little later.

```
...
"links": [
   { "rel": "detail", "href": "http://foo.com" },
   { "rel": "next", "href": "http://foo.com" },
   { "rel": "prev", "href": "http://foo.com" },
]
...
```

### Intuitive

If you can not explain your api in 30 seconds ( the elevator pitch ), it will be difficult to explain in writing let alone to others in documentation. Use web standards only where they make sense but **use the standards**. For example, a developer should be able to use a browser and point it at the service to see the results. Additionally, JSON is the new standard for the format of the data. This is a schema-less format, so you will need to make the schema relevant to the domain and relevant to the users.

## Security and Permissions

For private APIs I suggest Tokens, specifically JWT. For public facing APIs use oAuth V2. Avoid Basic Auth. While this is the standard way to authenticate a user, it is not appropriate for an Application.  

If you have to store credentials, never store the password, use a salted hash. preferable with a App Salt and a User Salt. 

### Principal and Subjects

5. Use Role Based Permissions. 
6. Keep roles as simple as possible. They always become more complex as time evolves.
7. Use a **Grant** based permissions model and NEVER restriction based permissions model.
8. Rate limiting should always be added to an endpoint. I have found it to be helpful if the rate limit countdown was in the headers. See GIT for a h

### Passwords

If you have to store passwords, don't. Create a App Salt and a User Salt. Add the two together to do a hmac5 digest verification of the password. 

### Encrypted Transmission

Use SSL everywhere, no exceptions. Don't worry about debugging the payload, Charles Proxy has the ability to be the man-in-the-middle and grant you visibility into the payload. 

### Tokens

If you can, use JWT it has been around for a while, easy to explain, and the internet is rich with examples.

JWT provides access to the claim. You can create a version number in the claim. Use the version number to notify your client that the version of the app is outdated and needs to be updated.

Claims can be decoded, as they are base64. this can provide meta data to the client on how to behave. For example, the principal's roles can be encased in the claims and the app can then use the roles to dictate the presentation. Furthermore, the subjects name can also be encased in the payload. 

Never put information in the JWT token that could compromise the user account. 

Renweal is simple, make a service that will generate a token when new is requested, and invalidate the current token. The client should track when the token is expired and initiate the new request. 

Expiry within 15-20 minutes is a good rule of thumb. Make sure the refresh tokens work once and only once.

### Rate Limiting

Rate limiting prevents users from _sucking_ all the data out of your system and prevents potentially dangerous dos attacks.

It is a good idea to reveal the rate usage as response headers.

### Service Unavailable 504

Dynamic horizontal scaling services may experience unavailability as they come on line. This really is not a good idea, but it does happen. Clients need to implement a retry attempt.

### Payload Content Restrictions

In MogoDB and some BASS systems you can use a pattern called _proejction_ this is the act of sending what members you want to include or exlcude in the payload. 

For example, if you want to exclude everything except fname, lname and ssn. The uri would have 
```
&projection=+fname,+lname,+ssn
```

Alternatively you could use _-_ to indicate remove. Generlly, avoid negative and use additive _+_. It is also possible to use a HTTP Header value and avoid the query parameter all together.

```
GET / HTTP/1.1
Host: erbosoft .com
Connection: keep-alive
Cache-Control: max-age=0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/38.0.2125.104 Safari/537.36
Accept-Encoding: gzip,deflate,sdch
Accept-Language: en-US,en;q=0.8
If-Modified-Since: Tue, 07 Feb 2012 04:44:06 GMT
Projection: +fname,+lname,+ssn
```

### Payload Wrappers / Envelopes and Pagination

I have implemented on several cases a Payload Wrapper or Envelope for the purpose of pagination through large volumes of data. They basically looked like the following. 

In this case, it included a page, page size, and toal. 
```JS
{ 
	data: [],
	page: 0,
	total: 2340,
	pageSize: 50,
}
```

In this cases, I built a set of pages like google
```JS
{ 
	data: [],
	pages : [
		{ rel: "_prev", href : "http://foo" },
		{ rel: "_page_1", href : "http://foo" },
		{ rel: "_page_2", href : "http://foo" },
		{ rel: "_page_3", href : "http://foo" },
		{ rel: "_page_4", href : "http://foo" },
		{ rel: "_page_5", href : "http://foo" },
		{ rel: "_next", href : "http://foo" },
	]
}
```

In this case, implemented the following which is foundly refered to as the endless list pattern.
```JS
{ 
	data: [],
	nextPage : { rel: "_next", href : "http://foo" },
}
```

Recently I packed the pagination data in the response header and passed the data back as an array of object literals.
```
x-page: 10
x-page-size: 50
x-page-total: 2340
```

### Errors - Problem Details for HTTP APIs ( [RFC 7807](https://datatracker.ietf.org/doc/html/rfc7807) )

Ive built many different type of error objects, I think this technique is favorable.

* ***type*** (string) - A URI reference [RFC3986] that identifies the problem type.  This specification encourages that, when dereferenced, it provide human-readable documentation for the problem type (e.g., using HTML [W3C.REC-html5-20141028]).  When this member is not present, its value is assumed to be "about:blank". This value should ( a URI ) should never change, making it a constant that systems can key on.

* ***title*** (string) - A short, human-readable summary of the problem type.  It SHOULD NOT change from occurrence to occurrence of the problem, except for purposes of localization (e.g., using proactive content negotiation; see [RFC7231], Section 3.4).

* ***status*** (number) - The HTTP status code ([RFC7231], Section 6) generated by the origin server for this occurrence of the problem.

* ***detail*** (string) - A human-readable explanation specific to this occurrence of the problem, this can change based on the details of the problem. EG the error could be because of an invalid parameter, this could call it out.

* ***instance*** (string) - A URI reference that identifies the specific occurrence of the problem.  It may or may not yield further information if dereferenced.

**Example:** Here, the out-of-credit problem (identified by its type URI) indicates the reason for the 403 in "title", gives a reference for the specific problem occurrence with "instance", gives occurrence- specific details in "detail", and adds two extensions; "balance"conveys the account's balance, and "accounts" gives links where the account can be topped up.

```HTTP
HTTP/1.1 403 Forbidden
   Content-Type: application/problem+json
   Content-Language: en

   {
    "type": "https://example.com/probs/out-of-credit",
    "title": "You do not have enough credit.",
    "status": 403,
    "detail": "Your current balance is 30, but that costs 50.",
    "instance": "/account/12345/msgs/abc",
    "balance": 30,
    "accounts": ["/account/12345",
                 "/account/67890"]
   }
```

**Example:** The ability to convey problem-specific extensions allows more than one problem to be conveyed.

```HTTP
HTTP/1.1 400 Bad Request
   Content-Type: application/problem+json
   Content-Language: en

   {
   "type": "https://example.net/validation-error",
   "title": "Your request parameters didn't validate.",
   "status": 400,
   "invalid-params": [ {
                         "name": "age",
                         "reason": "must be a positive integer"
                       },
                       {
                         "name": "color",
                         "reason": "must be 'green', 'red' or 'blue'"}
                     ]
   }
```
