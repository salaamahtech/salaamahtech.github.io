- [Basics of Security](/technology/security.html)
-
- **[https://www.pingidentity.com/en/resources/blog/post/complete-guide-to-api-security.html](https://www.pingidentity.com/en/resources/blog/post/complete-guide-to-api-security.html)**
-
-
- ![image.png](../assets/image_1702666991991_0.png)
-
- ![image.png](../assets/image_1702667005437_0.png)
-
- ![image.png](../assets/image_1702667017929_0.png)
-
- ![image.png](../assets/image_1702667029422_0.png)
- ![image.png](../assets/image_1702667041576_0.png)
- ![image.png](../assets/image_1702667056685_0.png)
- # Overview
  background-color:: yellow
	- ## 3 phases of API Security
	  background-color:: pink
		- **Secure by design**
			- At the design time of the app, define the security strategies
			- e.g., who are the consumers of the API? Is it going to be an internal or external API? Is PII data involved? What are the access patterns - B2B, B2C, B2B2C?
			- Typically owned by architecture team and development team
		- **Secure during build/development**
			- Use security libraries or utilities to develop a secure code.
			- Incorporate shift-left strategy in the CI pipeline (SAST)
			- Owned by app development team
		- **Secure at Runtime**
			- DAST and XAST tools are used to monitor and secure apps at runtime.
			- Includes the security setup in platforms like API Gateway, CDN, Service Mesh, etc.
			- Owned by platform teams.
- # API Security Methods
  background-color:: yellow
	- ## API Authentication Methods
	  background-color:: pink
		- Web APIs connected over the public Internet can make use of 3 authentication mechanisms: HMAC, Digital Signature, OAuth
		- ## HMAC
		  background-color:: blue
		  collapsed:: true
			- What?
				- HMAC (hash-based message authentication code) is used to verify that a request is coming from an expected source and that the request has not been tampered with in transit.
			- How it works
				- Server generates a public key (or API key that is unique for each client) and a private key (or secret key) combination.
				- Server sends both the keys to the client
				- Before passing the payload message to the server, client creates a signature or HMAC (hashed code) using the private key as follows
					- `signature = Base64( HMAC-SHA-256( PrivateKey, PayloadMessage ) )`
				- Client sends the payload (*API key + HMAC Signature + Actual Message* ) to the server
				- Server gets the public/API key from the request payload and gets the associated private key from the server.
				- Server then generates a signature of the message using the same formula as the client. If the server generated signature matches the one in the payload, then the server trusts the client and processes the request.
				- Server responses are signed in a similar way
				- ![image.png](../assets/image_1704636907407_0.png)
			- When to use?
				- When you have a public API and want to control who can access your API
			- Pros
				- Easy to implement
			- Cons
				- Server generates both the private and public key.
		- ## Digital Signature
		  background-color:: blue
			- This is nothing but the TLS mechanism
			- What?
				- This method relies on private-public key pairs (*Public Key Cryptography*) - which is useful for securing server-to-server communications.
				- For 2 way communication, both client and server will hold their own key pairs and exchange public keys with each other. (Similar to mTLS)
			- How it works?
				- It works by signing the message content with a private key to produce a security signature that can be verified using the corresponding public key (certificate). A key pair is usually provided by a certificate authority.
				- The client will provide its public key to the server. Each request is then made to the server by signing the message content using the private key. This allows the server to verify the message authenticity without knowing the private key of the client.
		- ## mTLS
		  background-color:: blue
			- mTLS can be used only if the clients are known and are willing to use this security method.
		- ## OAuth
		  background-color:: blue
			- Check OAuth under [[Security]]
- # Authentication
  background-color:: yellow
	- To enable authentication, modify the */WEB-INF/web.xml* file in the war file deployed.
	- Value of `<auth-method>` could be `BASIC`, 'DIGEST' or `CLIENT_CERT`
	- The `<login-config>` element doesn’t turn on authentication. To enforce authentication, you must specify a URL pattern you want to secure. e.g., `<url-pattern>/services/customers</url-pattern>`
	- The `<http-method>` element says that we only want to secure POST requests to this URL. If we leave out this element, all HTTP methods are secured.
	- `<auth-constraint>` specifies which roles are allowed to POST to `/services/customers`
	- `<role-name>` - If you set this to `*` then it means anybody who is able to log in can access the constrained URL. Authentication with a valid user would still be required, though.
	- *Limitation* - When declaring `<security-constraints>` for JAXRS resources, the `<url-pattern>` element does not have as rich an expression syntax as JAX-RS `@Path` annotation values. It supports only simple wildcard matches via the `*` character. No regular expressions like `/*`, `/foo/*` or `*.txt` are supported.
	- ```xml
	  <?xml version="1.0"?>
	  <web-app>
	    <security-constraint>
	      <web-resource-collection>
	        <web-resource-name>customer creation</web-resource-name>
	        <url-pattern>/services/customers</url-pattern>
	        <http-method>POST</http-method>
	      </web-resource-collection>
	      <auth-constraint>
	        <role-name>admin</role-name>
	      </auth-constraint>
	    </security-constraint>
	    <login-config>
	      <auth-method>BASIC</auth-method>
	      <realm-name>jaxrs</realm-name>
	    </login-config>
	    <security-role>
	      <role-name>admin</role-name>
	    </security-role>
	  </web-app>
	  ```
	- **Enforcing Encryptiong (HTTPS)** - To enforce HTTPS access for these constraints, specify a `<user-data-constraint>` as shown here. If a user tries to access the URL pattern with HTTP, she will be redirected to an HTTPS-based URL
	    
	  ```xml
	  <security-constraint>
	  ...
	  <user-data-constraint>
	    <transport-guarantee>CONFIDENTIAL</transport-guarantee>
	  </user-data-constraint>
	  </security-constraint>
	  ```
- # Authorization
  background-color:: yellow
	- The server and application know the permissions for each user and do not need to share this information over a communication protocol. This is why authorization is the domain of the server and application
	- JAX-RS relies on the servlet and Java EE specifications to define how authorization works. Authorization is performed in Java EE by associating one or more roles with a given user and then assigning permissions based on that role.
	- You do not assign access control on a per-user basis, but rather on a per-role basis.
	- Authorization can be enabled through XML or by applying annotations to JAX-RS resource classes
	- ### Authorization Annotations
		- Java EE defines a set of following annotations in package `javax.annotation.security` :
		- `@RolesAllowed` - defines the roles permitted to execute a specific method or defines the default ACL (Access Control List) for all HTTP operations.
		- `@DenyAll`
		- `@PermitAll`
		- `@RunAs`
		- **Pros**
			- Annotations allow to express fine-grained constraints that are not possible in *web.xml* because of the limited expression capabilities of `<url-pattern>`
			- Allows to apply constraints at method-level using these annotations.
		- **Cons**
			- Specify security related annotations within business logic code is a leakage of concern. To add or remove role constraints periodically, one has to recompile the whole application.
		- ```java
		  @Path("/customers")
		  @RolesAllowed({"ADMIN", "CUSTOMER"})
		  public class CustomerResource {
		    @GET
		    @Path("{id}")
		    @Produces("application/xml")
		    public Customer getCustomer(@PathParam("id") int id) {...}
		  
		    @RolesAllowed("ADMIN")
		    @POST
		    @Consumes("application/xml")
		    public void createCustomer(Customer cust) {...}
		  
		    @PermitAll
		    @GET
		    @Produces("application/xml")
		    public Customer[] getCustomers() {}
		  }
		  ```