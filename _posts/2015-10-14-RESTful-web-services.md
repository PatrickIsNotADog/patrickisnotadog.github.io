In this post, I will describe RESTful web services, starting from the enterprise need for sharing data and the way World Wide Web was developed in this direction. Then, I will talk about web services and how they emerged stepping on the web and the Internet, the general constraints of the REST architecture and how you can implement a RESTful web service. Finally, I will include some best practices.

## Computers change how we share resources

The essence of computer programming is automating tasks, that were formerly assigned to people, and have computers execute them instead. For example, let’s imagine a big company, in which employees have to constantly talk to each other on phone, but do not have all the phone numbers in front of them. The company has assigned the task of using and maintaining a phone book (an example that will be used throughout this post) to an employee, let’s call him Bob, who stands at his desk, holding a big yellow book and when asked for a name, he searches for the person who corresponds to it and gives it back to the asker. If a new employee comes to the company, Bob adds his name and phone to the book and if an employee leaves, he deletes this contact.

The company buys some PCs (we are still in the 90s) and decides to help Bob with his job, so they place a PC on his desktop, install an application that emulates a phone book and Bob does not have to manually search a book any more; when someone asks for a phone, he just types the name to the application and the phone appears in his monitor.

In the meanwhile, the company buys more machines and now every employee has a desktop computer. Since the phone book application is really easy to use, the company connects every computer with Bob’s and exposes the phone book as an internal service. Bob is now free from his boring task and can be occupied somewhere more productive.

The example above is very simplistic, but shows something common place in the world of informatics: the need of creating and exposing services, that share resources and are available from computers/endpoints away from the one, where the service is actual running. In our example, the resources are the contacts in the phone book and they have to be available from every computer in the company’s network.

## World Wide Web emerges

Pretty similar is [how the World Wide Web (or simply web) emerged](http://webfoundation.org/about/vision/history-of-the-web/). It all started with [Tim Berners-Lee](https://www.w3.org/People/Berners-Lee/), who was at that time working as a software engineer at CERN, and noticed that scientists, that came from all over the world to use organization’s equipment, had difficulty sharing information. Information was saved in different computers and in order to be retrieved, one had to login to the specific machine, that holded the specific information, he was concerned about. Tim saw a way to solve this problem, using the fast-developing internet (a network of computer networks, with machines connecting to each other using the TCP/IP protocol) and the hypertext technology. He imagined millions of devices, connected to each other through the internet, that would share resources using some common protocols and languages. So, he wrote three fundamental technologies that remain the foundation of today’s Web:

* HTML:    the web’s markup language.
* URI:    a unique “address” that identifies each resource in the web.
* HTTP:    the transfer protocol to share resources across the web.

He also wrote the first web browser and the first web server. CERN started sharing resources as web pages inside and outside the organization and web soon became famous and started growing. Then, Tim made a big decision: he unleashed the technology, in order to be freely adopted and developed by the world community, leading web to be the gigantic pool of information it is today.

## Web Services

Coming back to our company’s example, the company could decide to keep its employees’ phones private, but share the enterprise and organizations’ contact information to the public, to make it easier to look up for that kind of – already public – information. In that case, a web page could be created and get uploaded on the web. But what if other enterprises or organizations wanted to use this phone book, through some application? How would the two applications communicate with each other and which technology should be used for this case? Such needs lead to the development of web services.

There are many definitions of web services, more or less “strict” but they all agree to one thing: a web service is a software system, that is designed to be produced and consumed by machines, rather than humans, and should be available over a network. I will use a stricter definition for this post though: web services are software systems designed to be produced and consumed by machines, that make use of web technologies to be available over the internet. This definition is pretty similar with [the one Java uses](http://docs.oracle.com/javaee/6/tutorial/doc/gijvh.html). So, for this post the common ground for all web services is that they use HTTP to transfer requests and responses.

A frequently used implementation of web services, that can also be found in W3C’s [Web Services Architecture](https://www.w3.org/TR/ws-arch/) documentation, involves messages being exchanged using the SOAP protocol over HTTP and XML serialization for the body of the SOAP messages. This way, the server, that exposes the resources, and the client, that requests for them, can communicate independently of the platform or the language each of them uses. A high degree of interoperability is achieved. from one machine to another.

The con of this implementation though, is that a heavy payload is added on the data exchanged in order to comply with the SOAP protocol. [Here you can read a hilarious, yet descriptive, post](http://stackoverflow.com/questions/209905/representational-state-transfer-rest-and-simple-object-access-protocol-soap/8983122#8983122) describing this “payload tax”. So, it would be great if we could use a more lightweight way to exchange data with our web services.

Another aspect of this SOAP/XML implementation that deserves a deeper look, is how the requests are actually made. As said, SOAP is used over HTTP and it will usually target an endpoint, that exposes some services. Let’s say that the endpoint is `http://www.travelling-with-code.com/phonebook` and we want to invoke the “delete” service. We will create an XML message, that will contain the information about the contact we want to delete in an XML style:

```xml
<sns:deleteContact>
  <id>46</id>
</sns:deleteContact>
```

wrap the message with a SOAP envelope and send it to the endpoint using a POST method. The contradiction here is that we use a POST method to delete a contact, when the [formally documented DELETE](http://www.ietf.org/rfc/rfc2616.txt) could be used for this action.

## REST

To handle these issues, another implementation of web services was proposed, based on REST. These services are commonly known as RESTful web services. But let’s first take a look on REST.

REST, which stands for Representational State Transfer, is an architectural style for distributed hypermedia systems, initially proposed by Roy Thomas Fielding in his 2000 PhD dissertation [Architectural Styles and the Design of Network-based Software Architectures](https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm). As any architectural style, it declares some constraints that must be followed in order for a system to be described as “RESTful”. These constraints are:

* Client – Server design: There must be two clearly separated roles: the client, that makes requests, and the server that serves these requests. It is essential that the client and server interfaces are separated and independent from each other. Thus, the components are able to evolve independently and the same user interfaces can be used across multiple platforms, improving portability. Also, server components are simpler, improving server’s scalability.
* Stateless: This is a client-server interaction constraint and states that each request, from the client to the server, must contain all the information necessary to understand the request and no stored context on the server should be used. Session state is kept entirely on the client. This way, the server is free from keeping resources for each client and thus, can recover quickly from failures and also scale better. The downside is that network performance may decrease, because of possible repetitive data.
* Cache: Data within a response must be labeled as cacheable or non-cacheable. If the response is cacheable the client can reuse that data for later equivalent requests. This way performance is improved, but there is the danger of stale data within cache to differ significantly from the data that would have been obtained if the requests had been sent directly to the server.
* Uniform interface: Components must have a uniform interface, simplifying the system architecture and improving the visibility of interactions. Implementations are decoupled from the services they provide, which encourages independent evolvability. The trade-off is that the standardized form may be not the ideal for some specific implementation, thus decreasing performance.
* Layered system: The architecture allows the existence of multiple hierarchical layers, with which the components interact, but each component is capable to see only the layer it is interacting with and not deeper.
* Code on demand: Client can be extended by downloading and executing code in the form of applets or scripts, reducing the number of features required to be pre-implemented on the client.

Roy also declares some architectural elements:


* Data elements: REST components communicate by transferring data, that are a key aspect in REST. These data are exchanged in some representation/format, that can be different from the one they are stored in the client or the server. The format that will be used during a session gets mutually selected by the sender and the consumer, among a set of available formats.
* Resources and resource identifiers: Resource is a term that can be used for any information that can be named. For example the “Java current release” is a resource, but that does not mean that the resource is Java 8 (at the time of writing). Resource is the mapping to a set of entities and not the entity itself, something like a mathematical function. In the previous example, resource is the mapping to the current Java at any time and Java 8 is the resource value at the time of writing. Resource identifier is the way we can describe a resource and refer to it.
* Representations: A representation is a set of data, in the form of byte sequence, and metadata that describe the data. It represents the current or desired state of a resource.
* Connectors: a connector is an interface for component communication, that encapsulates the activities of accessing resources and transfering resource representations. As in-parameters, it uses request control data, a resource identifier indicating the target of the request and an optional representation. The out-parameters consist of response control data, optional resource metadata and an optional representation. There are 5 connector types: client, server, cache, resolver and tunnel.
* Components: REST components are the origin server, gateway, proxy and user agent.

You can see that the constraints described above are pretty vague, and in no way binded with specific technologies. For example, it is obvious that the client will communicate somehow with the server, but there is no constraint that the HTTP protocol must be used. Also, resource identifiers are not forced to be URIs.

## RESTful Web Services

In order for a web service to be called RESTful it must implement all REST’s constraints. Also, according to this post’s definition of a “web service”, a RESTful web service should also use web’s technologies, which are HTTP for transfer and URIs as resource identifiers.

We will now design a RESTful web service, implementing the phone book example. For simplicity, we will assume that contacts consist of a name, surname, the phone and an id uniquely identifying the contact. The main entity in a REST architecture are the resources, that represent the data, that the server holds and shares with its clients. In a phone book the main entity that contains useful data, we want to store, fetch and delete, are contacts. So, the service’s resources will be contacts and a reasonable URI that can be used to address them is `http://www.travelling-with-code.com/phonebook/contacts`. In order to address a specific contact, rather than all of them, we will use `http://www.travelling-with-code.com/phonebook/contacts/{id}`, where `{id}` is a variable that will contain the specific contact’s id.

Now that we decided about the service’s resources and identifiers we will have to choose what operations the service will expose to the outer world. Let’s support all CRUD operations and also see how a REST client would send requests to the service, using the HTTP protocol. First of all, in order to read (get) all the phone book contacts, a client will use the HTTP GET method, and the contacts’ URI:

    GET www.travelling-with-code.com/phonebook/contacts

and an HTTP response will arrive with all the phone book contacts in a specific format (let’s say JSON). You can see that the request is much smaller and more elegant than the corresponding SOAP/XML request. Also, the HTTP method denotes what we want to do: we want to get information from the server. The URI denotes the target of the request: we want to get the phone book contacts. The client should also be able to get a specific contact from the server:

    GET www.travelling-with-code.com/phonebook/contacts/3

or search for the contact(s) that match some criteria, using URI request attributes:

    GET www.travelling-with-code.com/phonebook/contacts?surname="Simpson"

Respectively, the other operations will be implemented as following:

    POST www.travelling-with-code.com/phonebook/contacts

with a message body describing a new contact in some form (e.g. JSON)

```
DELETE www.travelling-with-code.com/phonebook/contacts/3

PUT www.travelling-with-code.com/phonebook/contacts/3
```

with a message body containing the contact with the new attributes

## Best Practices

Some best practices I collected mainly from the following pages:

http://blog.mwaysolutions.com/2014/06/05/10-best-practices-for-better-restful-api/
http://www.restapitutorial.com/resources.html


* URIs should be descriptive (for example /phonebook/contacts declares that the resource are the contacts of a phonebook)
* Use HTTP verbs to mean something: GET should just get and not change, DELETE should be used for delete. Post is usually used to create and PUT to update a resource
* Use nouns, not verbs in URIs (GET /contacts and not /getAllContacts)
* Use plural nouns (/contacts instead of /contact, so that /contacts/11 has a better meaning)
* Use sub-resources for relations (/cars/11/drivers/3)
* Use query-string parameters for filtering, not as resource names (/search?item=cars&color=red should be avoided and /search/cars?color=red should be used instead)
* Provide filtering, sorting, field selection and paging for collections (GET /cars?color=red, GET /cars?sort=-manufactorer,+model, GET /cars?fields=manufacturer,model,id,color, GET /cars?offset=10&limit=5)
* Version your API (/blog/api/v1)
* Handle errors with HTTP status codes
* PUT and DELETE should be indepondent (2 deletes should just delete an entity)
* Reduce the payload: prefer JSON to XML
* Documentation: API should be properly documented. Swagger is a solution.

