
Best Practices for Building REST Apis
---

Ive been building API for Service Oriented Applications for over a decade. Many API design opinions and guidelines I have found are more academic in nature and less real world. My goal with this article is to describe the best practices for a pragmatic API approach based on my experience. Ive found the following items to be the key to the success of my systems The Human Aspect, Security and Permissions, Implementation.

# The Human Aspect

1. Adoption
2. Standards and Consistency
3. Self Discovery
4. Intuitive

## Adoption

The number one challenge I have consistently had is, resistance. While you might think it makes sense to add a layer of abstraction, apply single responsibility principal to each layer, or create a stable elastic system to your total solution landscape, you will find that not everyone will agree. This is usually founded in either the fact that it will inherently make the over all design more complex, or the users of your system enjoy the nature of traditional single layer systems. In either case, you will be forced to nurture the idea and advocate the adoption.

### Documentation

Use tools like [Swagger.io](http://Swagger.io) to build a design first approach. The added benefit is this markdown can be used to create Client and Server stubs as well as create interactive websites. The downside is this tool is discounted from the code ( unlike Java Annotation ).

### Assurance

## Standards and Consistency

## Self Discovery

## Intuitive

If you cant explain your api in 30 seconds, it will be difficult to explain in writing. Use web standards only where they make sense. A developer should be able to use a browser and point it at the service to see the results. 

# Security and Permissions

Use SSL everywhere, no exceptions. Don't worry about debugging the payload, Charles Proxy can do that for you.  

Ue tokens, and make them expire within 15-20 minutes. Make sure the refresh tokens work once and only once. 

Use oAuth V2 if possible and never store passwords ( only the salted hash ).

Keep roles as simple as possible.


