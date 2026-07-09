## What is REST?

REST (Representational State Transfer) is defined as a architectural design that guides the interaction through hypermedia distributed systems. REST organizes the application's components to operate as a virtual state machine where each Web interaction changes the app state transferring resources representations, i.e. data and metadata about the current machine state.

![[Pasted image 20260630202313.png]]


This app state changing goes by the HTTP methods (**GET**, **POST**, **PUT**, **DELETE**, etc.), and normally the data exchange is made by JSON due its light weighted format, however, we can do it by XML too, because REST does not define the data format.

## REST Principles

REST is based in 5 fundamentals principles:
- Server-client;
- Stateless;
- Cache;
- Uniform interface;
- Layered Architecture;

Below we can see come particularities of some principles:
### Stateless
By its nature, HTTP protocol doesn't keep state between a client and server, if its needed to keep any data about this communication we use exception and cookies, and for this matter, REST inherit this nature by using HTTP as main protocol.
Each client request MUST have ALL necessary information to get response from server, and the server MUST NOT keep any session state among requests, so this make possible server's scalability.

This is called CSS (Client Stateless Server), and that's REST allows us use any intermediary (middleware) such as load balances, proxies, caching. Bringing complexity to the application.

### Interface Uniform
Saying that the communication between Server-Client MUST be uniform means that the client MUST know how to interact with server without knowing how the server works.
E.g the response may come through a cache (locally or distributed) for a client, and for another it comes from the process itself, however the interface WILL NEVER change.

And besides this, we have the URI patterns usage, HTTP methods, status codes and media types.

### Layered Architecture
The client does not need to know if it is communicating with final server or cache or a middleware, whatever... and again this allows to scale the resources without affect the communication.

## REST VS Others
Firstly, we need to understand the image below:

![[Pasted image 20260630205755.png]]

1. We cannot compare REST with HTTP. REST is only a philosophy, because it uses HTTP protocol;
2. REST and SOAP are totally different because SOAP is a protocol and REST uses HTTP which is another protocol;
3. And REST and GraphQL are different because GraphQL is query language created to mitigate a bunch of requests sent in a row, something that happens with REST. Even GraphQL uses HTTP, it only has one way to retrieve data, besides REST have all HTTP methods to get the data.

## API Restful
First of all we need to know what is a API, so API (Application Programming Interface) is an interface that allow users to communicate with the server, this communication is made by a protocol, in this case the well-known HTTP protocol. And the REST philosophy comes as a design for this API, to show how it's going to work.

![[Pasted image 20260630211511.png]]

So, knowing all this stuff, we can say that a API Restful is an API that follows faithfully the REST principles and architecture, but in most of the time we only say REST API.

The REST's creator Roy Fielding says that a REST API must be hypertext-driven and self-describe, in other words, the API must provider ways for the user navigate through without knowing.
We can have full content in [Roy's 2008 blog](https://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven)
And that's why [Richardson's maturity model](https://martinfowler.com/articles/richardsonMaturityModel.html) was designed.


## Richardson Maturity Model


