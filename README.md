
Best Practices for Building REST Apis
---

Ive been building API for Service Oriented Applications for over a decade. Many API design opinions and guidelines I have found are more academic in nature and less real world. My goal with this article is to describe the best practices for a pragmatic API approach based on my experience. Ive found the following items to be the key to the success of my systems The Human Aspect, Security and Permissions, Implementation.

# The Human Aspect or the Biggest Barriers to Implementation

Service Oriented Architecture is a relatively new concept. "New" meaning with in the last 25 years. Before that we had Remote Procedures Calls which gained wide popularity with the advent of Common Object Request Broker Architecture (CORBA) and the father of electronic data interchange, EDI. After these technologies there was SOAP and a variety of other technologies. Given the dynamic nature of this landscape and added complexity. it is easy to understand that the biggest barrier to adoption and therefore implementation would be resistance by people. 

1. Adoption
2. Standards and Consistency
3. Self Discovery
4. Intuitive

## Adoption

The number one challenge I have consistently had is, resistance. While you might think it makes sense to add a layer of abstraction, apply single responsibility principal to each layer, or create a stable elastic system to your total solution landscape, you will find that not everyone will agree. This is usually founded in either the fact that it will inherently make the over all design more complex, or the users of your system enjoy the nature of traditional single layer systems. In either case, you will be forced to nurture the idea and advocate the adoption.

### Documentation

Use tools like [Swagger.io](http://Swagger.io) to build a design first approach. The added benefit is this markdown can be used to create Client and Server stubs as well as create interactive websites. The downside is this tool is discounted from the code ( unlike Java Annotation ).

### Assurance

Creating tests should be part of every developers daily activity. It provides assurance that the goal was accomplished and meets the needs of the specification.

## Standards and Consistency

## Self Discovery

Self discovery implies that links within the model coupled with meta data will make discovery of other endpoints and additional supporting data easy and helpful. 

This activity will aid in promoting the adoption of the system

When you have a href in the model, always include a **rel** value. This is one of the few meta data values that is widely adopted. I have found it better to actually include a directory of all the **rel** types in the system. This allows the developers to create decorators and easily create modules that can consume the values. 

## Intuitive

If you can not explain your api in 30 seconds, it will be difficult to explain in writing. Use web standards only where they make sense. A developer should be able to use a browser and point it at the service to see the results. 

# Security and Permissions

1. Use SSL everywhere, no exceptions. Don't worry about debugging the payload, Charles Proxy has the ability to be the man-in-the-middle and grant you visibility into the payload. 
2. Ue tokens, and make them expire within 15-20 minutes. Make sure the refresh tokens work once and only once. 
3. Use oAuth V2 if possible
4. never store passwords ( only the salted hash ).
5. Use Role Based Permissions. 
6. Keep roles as simple as possible. They always become more complex as time evolves.
7. Use a **Grant** based permissions model and NEVER restriction based permissions model.
8. Rate limiting should always be added to an endpoint. I have found it to be helpful if the rate limit countdown was in the headers. See GIT for a h


## Rate Limiting

Rate limiting prevents users from _sucking_ all the data out of your system and prevents potentially dangerous si


 
