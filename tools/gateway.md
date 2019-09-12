# gateway是什么
1. Access control:
	1) Routing
	Logical distribution of traffic based on different routing criteria such as, url path or domain headers.
	2) Authentication and authorization
		a. Plugins
	3) Rate limiting
		a. Rate limiting for specific API of specific user
		b. Quota - Request throttling
	
2. Traffic management
	1) Application level load balancing
		a. Applications, such as a content delivery network, that requires multiple HTTP requests on the same long-running TCP connection to be routed or load balanced to different back-end servers.
	2) Transformation
		a. Inject headers
		b. Transform inbound and outbound data
	3) Traffic control
	Canary Releases
	monolithic transition
	4) Cookie-based session affinity
	Applications that require requests from the same user/client session to reach the same back-end virtual machine. -- Shopping cart
	5) Request collapsing
	Similar request collapsed to one
	Decrease the connection time
	Less chance to break existing clients
	6) Cache
		a. Caching custom entities from custom data store, loading them once, and caching them in-memory.
		b. Cache requests - throttling
		c. Cache responses - performance acceleration
3. Off loading
	1) SSL termination
	Removing SSL termination overhead for web server farms.
	2) Authentication
	3) IP whitelisting
	4) Client rate limiting (throttling)
	5) Logging and monitoring
	6) Response caching
	7) Web application firewall
	8) GZIP compression
	9) Servicing static content

4. Reporting analytics
	1) Metrics
	2) Monitoring
	3) Logging/tracing
5. Security
	a. Protecting web applications from common web-based attacks like SQL injection, cross-site scripting attacks, and session hijacks.
	b. API gateway products provides access tokens, HMAC request signing, JSON Web tokens, OpenID Connect, basic auth, LDAP, Social OAuth (e.g. GPlus, Twitter, Github) and legacy Basic Authentication providers.
6. Others
	a. Mock API
	b. Notifications
	c. Release new API on the fly
	d. Multiple data center support
	e. API documentation
	f. Charge

• API gateway variations
Traditional API gateway

1. Access control:
	1) Routing
	Logical distribution of traffic based on different routing criteria such as, url path or domain headers.
	2) Authentication and authorization
		a. Plugins
	3) Rate limiting
		a. Rate limiting for specific API of specific user
		b. Quota - Request throttling
	
2. Traffic management
	1) Application level load balancing
		a. Applications, such as a content delivery network, that requires multiple HTTP requests on the same long-running TCP connection to be routed or load balanced to different back-end servers.
	2) Transformation
		a. Inject headers
		b. Transform inbound and outbound data
	3) Traffic control
	Canary Releases
	monolithic transition
	4) Cookie-based session affinity
	Applications that require requests from the same user/client session to reach the same back-end virtual machine. -- Shopping cart
	5) Request collapsing
	Similar request collapsed to one
	Decrease the connection time
	Less chance to break existing clients
	6) Cache
		a. Caching custom entities from custom data store, loading them once, and caching them in-memory.
		b. Cache requests - throttling
		c. Cache responses - performance acceleration
3. Off loading
	1) SSL termination
	Removing SSL termination overhead for web server farms.
	2) Authentication
	3) IP whitelisting
	4) Client rate limiting (throttling)
	5) Logging and monitoring
	6) Response caching
	7) Web application firewall
	8) GZIP compression
	9) Servicing static content

4. Reporting analytics
	1) Metrics
	2) Monitoring
	3) Logging/tracing
5. Security
	a. Protecting web applications from common web-based attacks like SQL injection, cross-site scripting attacks, and session hijacks.
	b. API gateway products provides access tokens, HMAC request signing, JSON Web tokens, OpenID Connect, basic auth, LDAP, Social OAuth (e.g. GPlus, Twitter, Github) and legacy Basic Authentication providers.
6. Others
	a. Mock API
	b. Notifications
	c. Release new API on the fly
	d. Multiple data center support
	e. API documentation
	f. Charge

• API gateway variations
Traditional API gateway
• Considerations of choosing which gateway
Features. The options listed above all support layer 7 routing, but support for other features will vary. Depending on the features that you need, you might deploy more than one gateway.
Deployment. Azure Application Gateway and API Management are managed services. Nginx and HAProxy will typically run in containers inside the cluster, but can also be deployed to dedicated VMs outside of the cluster. This isolates the gateway from the rest of the workload, but incurs higher management overhead.
Management. When services are updated or new services are added, the gateway routing rules may need to be updated. Consider how this process will be managed. Similar considerations apply to managing SSL certificates, IP whitelists, and other aspects of configuration.

# Gateway 的好处
Using an API gateway has the following benefits:
	• Insulates the clients from how the application is partitioned into microservices
	• Insulates the clients from the problem of determining the locations of service instances
	• Provides the optimal API for each client
	• Reduces the number of requests/roundtrips. For example, the API gateway enables clients to retrieve data from multiple services with a single round-trip. Fewer requests also means less overhead and improves the user experience. An API gateway is essential for mobile applications.
	• Simplifies the client by moving logic for calling multiple services from the client to API gateway
	• Translates from a “standard” public web-friendly API protocol to whatever protocols are used internally
The API gateway pattern has some drawbacks:
	• Increased complexity - the API gateway is yet another moving part that must be developed, deployed and managed
	• Increased response time due to the additional network hop through the API gateway - however, for most applications the cost of an extra roundtrip is insignificant.
Issues:
	• How implement the API gateway? An event-driven/reactive approach is best if it must scale to scale to handle high loads. On the JVM, NIO-based libraries such as Netty, Spring Reactor, etc. make sense. NodeJS is another option.

https://tyk.io/blog/microservices-api-gateway/
Prevents exposing internal concerns to external clients. An API gateway separates external public APIs From internal microservice APIs, allowing for microservices to be added and boundaries changed. The result is the ability to refactor and right-size microservices over time, without negatively impacting externally-bound clients. It also hides service discovery and versioning details from the client by providing a single point of entry for all of your microservices.
Adds an additional layer of security to your microservices. API gateways help to prevent malicious attacks by providing an additional layer of protection from attack vectors such as SQL Injection, XML Parser exploits, and denial-of-service (DoS) attacks.
Enables support for mixing communication protocols. While external-facing APIs commonly offer an HTTP or REST-based API, internal microservices may benefit from using different communication protocols. Protocols may include ProtoBuf, AMQP, or perhaps system integration with SOAP, JSON-RPC, or XML-RPC. An API gateway can provide an external, unified REST-based API across these various protocols, allowing teams to choose what best fits the internal architecture.
Decreased microservice complexity. Microservices have common concerns, such as: authorization using API tokens, access control enforcement, and rate limiting. Each of these concerns can add more time to the development of microservices by requiring that each service implement them. An API gateway will remove these concerns from your code, allowing your microservices to focus on the task at hand.
Microservice Mocking and Virtualization. By separating microservice APIs from the external API, you can mock or virtualize your services to validate design requirements or assist in integration testing.

# 为什么需要gateway
1. Why gateway?

The Microservice architecture pattern creates the need for this pattern.
A microservice-based architecture may have from 10 to 100 or more services. An API gateway can help provide a unified entry point for external consumers, independent of the number and composition of internal microservices.

From: https://www.nginx.com/blog/building-microservices-using-an-api-gateway/
• One problem is the mismatch between the needs of the client and the fine‑grained APIs exposed by each of the microservices. The client in this example has to make seven separate requests. In more complex applications it might have to make many more.
• Another problem with the client directly calling the microservices is that some might use protocols that are not web‑friendly. One service might use Thrift binary RPC while another service might use the AMQP messaging protocol. Neither protocol is particularly browser‑ or firewall‑friendly and is best used internally. An application should use protocols such as HTTP and WebSocket outside of the firewall.
• Another drawback with this approach is that it makes it difficult to refactor the micro services. Over time we might want to change how the system is partitioned into services. For example, we might merge two services or split a service into two or more services. If, however, clients communicate directly with the services, then performing this kind of refactoring can be extremely difficult.

# 缺点
Why Do Microservices Need an API Gateway?
https://tyk.io/blog/microservices-api-gateway/
Benefits and drawbacks in there

Drawbacks:
• Your deployment architecture will require more orchestration and management with the addition of an API gateway
• Configuration of the routing logic must be managed during deployment, to ensure proper routing from the external API to the proper microservice
• Unless properly architected for high availability and scale, an API gateway can become a limiting factor and even a single point of failure

a. Zuul 2 支持websocket，zuul 2是async的non blocking编程模型基于netty，not released yet
b. The API Gateway also has some drawbacks. It is yet another highly available component that must be developed, deployed, and managed. There is also a risk that the API Gateway becomes a development bottleneck. Developers must update the API Gateway in order to expose each microservice’s endpoints. It is important that the process for updating the API Gateway be as lightweight as possible. Otherwise, developers will be forced to wait in line in order to update the gateway.



c. Considerations
From https://www.nginx.com/blog/building-microservices-using-an-api-gateway/
	a. Performance and Scalability
	b. Using a Reactive Programming Model
	c. Service Invocation
	d. Service Discovery
	e. Handling Partial Failures

