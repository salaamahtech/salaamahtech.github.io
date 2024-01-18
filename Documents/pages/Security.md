# Overview
background-color:: yellow
collapsed:: true
	- Application Security has 4 aspects:
		- **Authentication** - proving user identity, often called user ‘login'.
		- **Authorization** - access control
		- **Cryptography** - protecting or hiding data from prying eyes
		- **Session Management** - per-user time-sensitive state
	- ## Core Concepts
	  collapsed:: true
		- ![](../assets/shiro-basics.png)
		- 3 Main concepts in security
			- 1. Subject - "the currently executing user". It could be more than a human user. e.g., daemon apps, 3rd party apps, etc.
			- 2. SecurityManager - manages security operations for all users
			- 3. Realm - acts the bridge/connector between the security provider and your application'' security data like user accounts, access controls, etc.
- # 1. Authentication
  background-color:: yellow
	- Process of verifying a user's identity. Also called 'login' process.
	- `Authentication Token = Principal/username + Credential/password`
	- If the submitted credentials match what the system expects for that user identity (principal), the user is considered authenticated.
	- ## Basic Authentication
	  background-color:: pink
	  collapsed:: true
		- Basic authentication is the simplest protocol available for performing authentication over HTTP. It involves sending a Base64-encoded username and password within a request header to the server. With Basic authentication (with and without SSL), your name and password do get automatically Base64- encoded.
		  
		  The server checks to see if the username exists within its system and verifies the sent password. The client needs to send this Authorization header with each and every request it makes to the server.
		- ### Unauthorized request
		  collapsed:: true
			- The 401 response tells the client that it is not authorized to access the URI it tried to invoke on.
			- The **WWW-Authenticate**   header specifies which authentication protocol the client should use.
			- In this case, **Basic**   means basic authentication should be used. The realm attribute identifies a collection of secured resources on a website.
			  
			  ```http Response to an unauthorized request
			  HTTP/1.1 401 Unauthorized
			  WWW-Authenticate: Basic realm="CustomerDB Realm"
			  ```
		- ### Response with authorization details
		  collapsed:: true
			- To perform authentication, the client must send a request with the Authorization header set to a Base64-encoded string of our username and a colon character, followed by the password. If our username is bburke and our password geheim, the Base64-encoded string of bburke:geheim will be `YmJ1cmtlOmdlaGVpbQ==`.
			- The client needs to send this Authorization header with each and every request it makes to the server.
			  
			  ```http Request with authorization details
			  GET /customers/333 HTTP/1.1
			  Authorization: Basic YmJ1cmtlOmdlaGVpbQ==
			  ```
			- **Downside**: The problem with this approach is that if this request is intercepted by a hostile entity on the network, the hacker can easily obtain the username and password and use it to invoke its own requests. Using an encrypted HTTP connection, HTTPS, solves this problem. With an encrypted connection, a rogue programmer on the network will be unable to decode the transmission and get at the Authorization header. Still, security-paranoid network administrators are very squeamish about sending passwords over the network, even if they are encrypted within SSL packets.
	- ## Digest Authentication
	  background-color:: pink
	  collapsed:: true
		- Digest authentication was invented so that clients would not have to send clear text passwords over HTTP. It involves exchanging a set of secure MD5 hashes of the username, password, operation, URI, and optionally the hash of the message body itself. It is a hash value generated with the following pseudocode:
		- ### Unauthorized request
		  collapsed:: true
			- ```http 
			  GET /dir/index.html HTTP/1.0
			  Host: localhost
			  ```
			- **Response to an unauthorized request**
				- ```http
				  HTTP/1.1 401 Unauthorized
				  WWW-Authenticate: Digest realm="CustomerDB Realm",
				  qop="auth,auth-int",
				  nonce="12dcde223152321ab99cd",
				  opaque="aa9321534253bcd00121"
				  ```
				- The **nonce**   and **opaque**   attributes are special server-generated keys that will be used to build the subsequent authenticated request.
		- ### Hash calculating algorithm
		  collapsed:: true
			- ```python
			  H1 = md5("username:realm:password")
			  H2 = md5("httpmethod:uri")
			  response = md5("H1:nonce:nc:cnonce:qop:H2")
			  ```
			- The **nonce**   and **opaque**   attributes are a copy of the values sent with the earlier WWWAuthenticate header.
			- The **uri**   attribute is the base URI you are invoking on.
			- The **nc**   attribute is a request counter that should be incremented by the client with each request. This prevents hostile clients from replaying a request.
			- The **cnonce**   attribute is a unique key generated by the client and can be anything the client wants.
			- The **response**   attribute is where all the meat is. It is a hash value generated with the following pseudocode:
			- When the server receives this request, it builds its own version of the **response**   hash using its stored, secret values of the username and password. If the hashes match, the user and its credentials are valid.
			  
			  ```http Request with Authorization details
			  GET /customers/333 HTTP/1.1
			  Authorization: Digest username="bburke",
			  realm="CustomerDB Realm",
			  nonce="12dcde223152321ab99cd",
			  uri="/customer/333",
			  qop="auth",
			  nc=00000001,
			  cnonce="43fea",
			  response="11132fffdeab993421",
			  opaque="aa9321534253bcd00121"
			  ```
		- ### Pros
		  collapsed:: true
			- The password is never used directly by the protocol. For example, the server doesn't even need to store clear text passwords. It can instead initialize its authorization store with pre-hashed values.
			- Also, since request hashes are built with a nonce value, the server can expire these nonce values over time. This, combined with a request counter, can greatly reduce replay attacks.
			  When the server receives this request, it builds its own version of the response hash using its stored, secret values of the username and password. If the hashes match, the user and its credentials are valid.
		- ### Cons
		  collapsed:: true
			- This approach is that unless you use HTTPS, you are still vulnerable to man-in-the-middle attacks, where the middleman can tell a client to use Basic authentication to obtain a password.
	- ## X.509 certificates
	  background-color:: pink
		- HTTPS is not only an encryption mechanism - it can also be used for authentication.
		- How it works?
			- ![](../assets/security-certs.gif)
			- When you first interact with a secure website, your browser receives a digitally signed certificate from the server that identifies it.
			- Your browser verifies this certificate with a central authority like VeriSign. This is how you guarantee the identity of the server you are interacting with and make sure you're not dealing with some *man-in-the-middle* security breach.
		- HTTPS can also perform two-way authentication. In addition to the client receiving a signed digital certificate representing the server, the server can receive a certificate that represents and identifies the client. (See mTLS section below)
			- When a client initially connects to a server, it exchanges its certificate and the server matches it against its internal store.
			- Once this link is established, there is no further need for user authentication, since the certificate has already positively identified the user.
		- > SSL, or Secure Socket Layer, is a technology which allows web browsers and web servers to communicate over a secured connection. This means that the data being sent is encrypted by one side, transmitted, then decrypted by the other side before processing. This is a two-way process, meaning that both the server AND the browser encrypt all traffic before sending out data.
		- > Another important aspect of the SSL protocol is Authentication. This means that during your initial attempt to communicate with a web server over a secure connection, that server will present your web browser with a set of credentials, in the form of a "Certificate", as proof the site is who and what it claims to be. In certain cases, the server may also request a Certificate from your web browser, asking for proof that you are who you claim to be. This is known as "Client Authentication," although in practice this is used more for business-to-business (B2B) transactions than with individual users. Most SSL-enabled web servers do not request Client Authentication.
		- ### Pros
			- Most secure way to perform authentication on web.
		- ### Cons
			- Managing of the certificates is painful. The server must create a unique certificate for each client that wants to connect to the service. From the browser/human perspective, this can be a pain, as the user has to do some extra configuration to interact with the server.
		- ### CLI commands
			- ```bash
			  # to fetch the certificate of google.com
			  openssl s_client -connect google.com:443 2>/dev/null < /dev/null | sed -n '/BEGIN CERTIFICATE/,/END CERTIFICATE/p' > google.com.crt
			  
			  # to view the crt file content
			  cat google.com.crt
			  
			  # to view the human-readable content of the certificate
			  openssl x509 -text -noout -in google.com.crt
			  
			  ```
		- ![Illustrated X.509 Certificate. Introduction _ by Takahiko Kawasaki _ Medium.pdf](../assets/Illustrated_X.509_Certificate._Introduction_by_Takahiko_Kawasaki_Medium_1704816157924_0.pdf)
		- ![An_Overview_of_x.509_certificates.pdf](../assets/An_Overview_of_x.509_certificates_1704816238580_0.pdf)
	- ## OpenID
	  background-color:: pink
	  collapsed:: true
		- __OpenID__ is the most common open web protocol for handling federated authentication or federated identity.
		- __OpenID Connect__ is the next-generation version of OpenID based on OAuth 2.0.
		- Few other identity protocols: SAML, WS-Federation, WS-Trust, SCIM, etc.
		- ### What is Federated Authentication?
			- Although many applications have their own system of accounts (including usernames and passwords), some applications rely on other services to verify the identity of users. This is called __federated authentication__.
			- In a corporate IT environment, applications may trust on the following to authenticate users
				- an Active Directory server
				- a LDAP server, or
				- a SAML provider
			- On the Web, applications often trust OpenID providers (such as Google or Yahoo!) to handle the authentication of users.
			- eliminates the need for each web application to build its own custom authentication system, and it makes it much easier and faster for users to sign up and sign into sites around the Web.
			- **Comparison to OAuth2**
				- Passing permission to access authentication information (the user’s identity) to a site is very similar to passing along delegated access to a user’s data.
				- OpenID Connect is built on top of OAuth 2.0 and is designed as a modular specification.
		- ### How OpenID Flow works?
		  collapsed:: true
			- The basic flow for OpenID Connect is:
				- 1. The application requests OAuth 2.0 authorization for one or more of the OpenID Connect scopes (`openid`, `profile`, `email`, `address`) by redirecting the user to an identity provider.
				- 2. After the user approves the OAuth authorization request, the user’s web browser is redirected back to the application using a traditional OAuth flow.
			- The app makes a request to the __Check ID Endpoint__. This endpoint returns the user’s identity (`user_id`) as well as other bits, such as the `aud` and `state`, which must be verified by the client to ensure valid authentication.
			  3. If the client requires additional profile information about the user, such as the user’s full name, picture, and email address, the client can make requests to the __UserInfo Endpoint__.
		- ### ID Token
		  collapsed:: true
			- With OpenID Connect authentication, there is an additional type of OAuth token: __an ID token__. 
			  _ The ID token, or `id_token`, represents the identity of the user being authenticated. This is a separate token from the access token, which is used to retrieve the user’s profile information or other user data requested during the same authorization flow.
			- The ID token is a __JSON Web Token (JWT)__, which is a digitally signed and/or encrypted representation of the user’s identity asserted by the identity provider. Instead of using cryptographic operations to validate the JSON Web Token, it can be treated as an opaque string and passed to the __Check ID Endpoint__ for interpretation. This flexibility keeps with the spirit of OAuth 2.0 and OpenID Connect being significantly easier to use than their predecessors.
			- Google’s OpenID Connect implementation uses the following Endpoints:
			  collapsed:: true
				- #### CheckID Endpoint
				  collapsed:: true
					- Endpoint - https://www.googleapis.com/oauth2/v1/tokeninfo
					- Request
						- ```bash 
						  curl https://accounts.example.com/oauth2/tokeninfo?
						  id_token=eyJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJhY2NvdW50cy5nb29nbGUuY29tIiwiY...
						  ```
					- Response
						- ```json 
						  {
						  "iss" : "https://accounts.example.com",
						  "user_id" : "113487456102835830811",
						  "aud" : "753560681145-2ik2j3snsvbs80ijdi8.apps.googleusercontent.com",
						  "exp" : "1311281970",
						  "nonce" : "53f2495d7b435ac571"
						  }
						  ```
					- If this verification is completed successfully, the `user_id` is known to represent the unique identifier for the authenticated user, within the scope of the issuer (`iss`). If storing the identifier in a user database table and multiple identity providers are supported by your application, it is recommended that both values be stored upon account creation and queried upon each subsequent authentication request.
				- #### UserInfo Endpoint
				  collapsed:: true
					- Endpoint - https://www.googleapis.com/oauth2/v1/userinfo
					- Request
						- ```bash
						  GET /v1/userinfo HTTP/1.1
						  Host: accounts.example.com
						  Authorization: Bearer ya29.AHES6ZSzX
						  ```
					- Response
						- ```json
						  {
						  "user_id": "3191142839810811",
						  "name": "Example User",
						  "given_name": "Example",
						  "family_name": "User",
						  "email": "user@example.com",
						  "verified": true,
						  "profile": "http://profiles.example.com/user",
						  "picture": "https://photos.profiles.example.com/user/photo.jpg",
						  "gender": "female",
						  "birthday": "1982-02-11",
						  "locale": "en-US"
						  }
						  ```
		- ### Security Properties
		  collapsed:: true
			- Replay attacks occur when legitimate credentials are sent multiple times for malicious purposes.
			- There are 2 main types of replay attacks we wish to prevent:
				- (1) An attacker capturing a user’s OAuth credentials as they log in to a site and using them later on the same site.
					- _Solution_: The OAuth 2.0 specification requires the OAuth endpoint and APIs to be accessed over SSL/TLS to prevent man-in-the-middle attacks, such as the first case.
				- (2) A rogue application developer using the OAuth token a user was issued to log in to their malicious app in order to impersonate the user on a different legitimate app.
					- _Solution_: To solve this, use the Check ID Endpoint to verify that the credentials issued by the OAuth provider were issued to the correct application.
					- It is recommended that all developers use the Check ID Endpoint or decode the JSON Web Token to verify the asserted identity, though this is not strictly necessary in some cases when the application uses the server-side Web Application flow and the UserInfo Endpoint provides all required information.
	- ## mTLS
	  background-color:: pink
		- Mutual transport layer security (mTLS) or two-way secure socket layer is a method for mutual authentication.
		- It ensures that traffic is secure and trusted in both directions between a client and the server – it ensures that both parties sharing information are who they claim to be by verifying that they both have the correct private key.
		- In mutual includes an additional step in which the server also asks for the client's certificate and verifies it at their end. It is considered more secure than TLS, but it’s also more computationally costly.
		- ### What is TLS?
		  background-color:: blue
			- [Transport Layer Security (TLS)](https://www.cloudflare.com/learning/ssl/transport-layer-security-tls/) is an encryption protocol in wide use on the Internet. TLS, which was formerly called SSL. It serves 2 purposes
				- authenticates the server in a client-server connection (trust)
				- and encrypts communications between client and server so that external parties cannot spy on the communications. (secure)
			- Netscape Communications Corporation developed SSL in 1994 to secure web sessions.
			- TLS is normally implemented on top of TCP in order to encrypt application layer protocols such as HTTP, FTP, SMTP and IMAP, although it can also be implemented on UDP, datagram congestion control protocol (DCCP) and stream control transmission protocol (SCTP), as well.
		- ### How TLS works?
		  background-color:: blue
			- There are 3 important things to understand about how TLS works
				- Public key cryptography
				- TLS certificate
					- A TLS certificate is a data file that contains important information for verifying a server's or device's identity, including the public key, a statement of who issued the certificate (TLS certificates are issued by a certificate authority), and the certificate's expiration date.
				- TLS handshake
					- The [TLS handshake](https://www.cloudflare.com/learning/ssl/what-happens-in-a-tls-handshake/) is the process for verifying the TLS certificate and the server's possession of the private key. The TLS handshake also establishes how encryption will take place once the handshake is finished.
			- In TLS, the server has a TLS certificate and a public/private key pair, while the client does not. The typical TLS process works like this:
				- Client connects to server
				  logseq.order-list-type:: number
				- Server presents its TLS certificate
				  logseq.order-list-type:: number
				- Client verifies the server's certificate
				  logseq.order-list-type:: number
				- Client and server exchange information over encrypted TLS connection
				  logseq.order-list-type:: number
			- ![how_tls_works-what_is_mutual_tls.png](../assets/how_tls_works-what_is_mutual_tls_1704644534439_0.png){:height 308, :width 889}
		- ### How mTLS works?
		  background-color:: blue
			- In mTLS, however, both the client and server have a certificate, and both sides authenticate using their public/private key pair.
			- Compared to regular TLS, there are additional steps in mTLS to verify both parties (additional steps in **bold**):
				- Client initiates the handshake with the server
				  logseq.order-list-type:: number
					- The client initiates the handshake with the ‘Client Hello.’ It consists of several pieces of information, including: 
					  logseq.order-list-type:: number
					  > Version:      Hex code of the highest version of TLS/SSL that the client supports.
					  Session ID:    An 8-byte value, initially all 0s, is used to label the sessions.
					  Cipher Suits:  A list of cipher suites (algorithms) client supports, which is nothing but a set of cryptographic algorithms, like RSA, etc.
				- Server responds with handshake
				  logseq.order-list-type:: number
					- Once received, the server responds with ‘Server Hello,’ which consists of the same five pieces of information for the server, this time with a session ID.
					  logseq.order-list-type:: number
				- Server sends its TLS certificate along with its certificate chain and its public key
				  logseq.order-list-type:: number
				- Client verifies the server's certificate with a certificate authority
				  logseq.order-list-type:: number
					- Is the certificate valid? This done by verifying the signature on the certificate using CA’s public key (that it was indeed signed by CA using his private key) 
					  logseq.order-list-type:: number
					- Does it belong to the intended server?
					  logseq.order-list-type:: number
				- To verify whether the certificate belongs to the intended server, the client creates a random secret key and encrypts it using the server’s public key and sends it to the server.
				  logseq.order-list-type:: number
				- **Client presents its TLS certificate**
				  logseq.order-list-type:: number
				- **Server verifies the client's certificate**
				  logseq.order-list-type:: number
				- **Server grants access**
				  logseq.order-list-type:: number
				- Client and server exchange information over encrypted TLS connection
				  logseq.order-list-type:: number
			- ![image.png](../assets/image_1704670823681_0.png)
		- ### Difference between TLS and mTLS
		  background-color:: blue
		  collapsed:: true
			- | **TLS** | **mTLS** |
			  | Only server has a TLS certificate and a public/private key pair, while the client does not | Both client and server have their own TLS certificate and a key pair |
			  | TLS uses an external certificate authority | Organization implementing mTLS act as its own certificate authority. It uses a self-signed root certificate -  meaning the org creates it themselves. This won't work for one-way TLS on the public Internet because an external CA has to issue those certificates |
			  | Computationally cheap and fast | Costly and slower due to additional round-trips|
		- ### Where to use mTLS?
		  background-color:: blue
		  collapsed:: true
			- mTLS is often used in zero trust security environments to verify users, devices, and servers within an org network.
			- In B2B API interactions where the server doesn’t want to expose its services to the entire world and wants to make sure the request is coming from a known client.
			- In B2B financial transactions. For example, transactions between two bank servers.
			- For authenticating and encrypting service-to-service communication between microservices or with the API gateway.
		- ### mTLS in Service Mesh
		  background-color:: blue
		  collapsed:: true
			- For implementing mTLS, the service mesh control plane offers:
				- A Certificate Authority that issues trusted x.509 certificates for microservices authentication
				- A configuration API server to define and deliver communication security policies for authentication and authorization to the sidecar proxies.
				- Sidecar proxies that sit alongside each microservice and help manage communications based on security policies
			- ![Service-mesh-control-plane.webp](../assets/Service-mesh-control-plane_1704675286343_0.webp){:height 395, :width 901}
		- ### Pros
		  background-color:: blue
			- Prevents various kinds of attacks including
				- **On-path attacks:** [On-path attackers](https://www.cloudflare.com/learning/security/threats/on-path-attack/) place themselves between a client and a server and intercept or modify communications between the two. When mTLS is used, on-path attackers cannot authenticate to either the client or the server, making this attack almost impossible to carry out.
				- **Credential stuffing:** Attackers use leaked sets of credentials from a data breach to try to log in as a legitimate user. Without a legitimately issued TLS certificate, [credential stuffing](https://www.cloudflare.com/learning/bots/what-is-credential-stuffing/) attacks cannot be successful against organizations that use mTLS.
				- **Phishing attacks:** The goal of a phishing attack is often to steal user credentials, then use those credentials to compromise a network or an application. Even if a user falls for such an attack, the attacker still needs a TLS certificate and a corresponding private key in order to use those credentials.
				- **Malicious API requests:** When used for [API security](https://www.cloudflare.com/learning/security/api/what-is-api-security/), mTLS ensures that API requests come from legitimate, authenticated users only. This stops attackers from sending malicious API requests that aim to exploit a vulnerability or subvert the way the API is supposed to function.
		- ### Cons
		  background-color:: blue
			- Complex to implement
			- Managing of the certificates is painful.
				- The server must create a unique certificate for each client that wants to connect to the service. From the browser/human perspective, this can be a pain, as the user has to do some extra configuration to interact with the server.
			- More computationally costly and slower than TLS
			- Can only be used in an environment where you have control over the clients and can dictate what type of security each client must have in order to connect to the server.
- # 2. Authorization
  background-color:: yellow
  collapsed:: true
	- Authorization is essentially access control - controlling what your users can access in your application, such as resources, web pages, etc.
	- ## Simple Access Control List (ACL)
	  background-color:: pink
	  collapsed:: true
		- Roles/Users assigned with access to certain features. Users are assigned to roles. This model is not flexible since roles cannot be added dynamically
		- Permissions are defined for features. Users are assigned permissions. This model allows dynamic addition of users.
	- ## OAuth
	  background-color:: pink
	  collapsed:: true
		- > OAuth 2.0 = Federated Authorization
		  OpenID Connect = Federated Authentication
		- __What is OAuth?__
		  collapsed:: true
			- OAuth is a _delegated authorization_ protocol
			- Delegated authorization is granting access to another person or application to perform actions on your behalf.
			- When you drive your car to a classy hotel, they may offer valet parking. You then authorize the valet attendant to drive your car by handing him the key in order to let him perform actions on your behalf.
			- OAuth works similarly—a user grants access to an application to perform actions on the user'' behalf and the application can only perform the authorized actions.
		- __Benefits of OAuth?__
		  collapsed:: true
			- OAuth provides the ability to access a user'' data securely, without handing over the account password.
			- OAuth eliminates data silos
			- Having a common protocol for handling API authorization greatly improves the developer experience.
		- __Why not basic username and password?__
		  collapsed:: true
			- user may not trust providing their credentials to your app
			- username-password provides only authn, not authz
			- only way a user can revoke access to your app is by changing the password
			- difficult to implement stronger authentication like CAPTCHAs or multi-factor auths
		- __Diff b/w OAuth 1.0 and 2.0?__
		  collapsed:: true
			- OAuth 1.0
				- required cryptographic signatures be sent with each API request to verify the identity and authorization of the client.
				- Cons
					- Cryptography is challenging for the casual developer to grasp and also challenging for even highly skilled engineers to master.
			- OAuth 2.0
				- eliminated the complex signature requirements and introduced the use of _bearer tokens_.
				- Cons
					- removing signatures makes it easy for developers to make mistakes and accidentally send their credentials to a malicious API endpoint, which can then abuse these credentials to make additional requests because they're not protected by a signature.
		- __Mitigating concerns with OAuth 2.0 Bearer Tokens__
		  collapsed:: true
			- When implementing OAuth 2.0, calling any APIs, or using a library, you should verify that it properly handles SSL/TLS certificate chain validation by doing the following things:
				- Checking that the hostname on the certificate returned by the server matches the hostname in the URL being accessed
				- Verifying each certificate in the chain properly chains up to a valid and trusted certificate authority (CA)
				- Ensuring that the certificate authority bundle on your server is secure and not able to be modified by potential attackers
		- __Tools__
		  collapsed:: true
			- https://code.google.com/oauthplayground/
		- ### OAuth Roles
		  background-color:: green
		  collapsed:: true
			- ![](../assets/oauth-actors.png)
			- __Resource server__
				- The server hosting user-owned resources that are protected by OAuth. This is typically an API provider that holds and protects data such as photos, videos, calendars, or contacts. e.g., Google, Facebook, etc.
			- __Resource owner__
				- Typically the user of an application, the resource owner has the ability to grant access to their own data hosted on the resource server.
			- __Client__
				- An application making API requests to perform actions on protected resources on behalf of the resource owner and with its authorization. e.g., Quora
			- __Authorization server__
				- The authorization server gets consent from the resource owner and issues access tokens to clients for accessing protected resources hosted by a resource server. e.g., Okta, Auth0
				- Smaller API providers may use the same application and URL space for both the authorization server and resource server.
		- ### Client Registration
		  background-color:: green
		  collapsed:: true
			- OAuth requires that applications register with the authorization server so that API requests are able to be properly identified. While the protocol allows for registration using automated means, most API providers require manual registration via filling out a form on their developer websites.
			- After registration is complete, the developer is issued client credentials:
				- _Client ID_ - Specified as `client_id` when interacting with the resource server
				- _Client Secret_ - Specified as `client_secret` when exchanging an authorization code for an access token and refreshing access tokens using the server-side Web Application Flow.
			- _Why Register?_
				- Registration enables the application developer to obtain client credentials, which are used to authenticate requests made to the authorization server. These credentials are critical in protecting the authenticity of requests when performing operations such as exchanging authorization codes for access tokens and refreshing access tokens
			- [Google APIs Console](http://code.google.com/apis/console)
			- Microsoft Windows Live [application management site](https://manage.dev.live.com/).
			- [Facebook Developers site](https://developers.facebook.com/apps).
		- ### Access Tokens
		  background-color:: green
		  collapsed:: true
			- OAuth 2.0 authorized APIs require only bearer tokens to make authorized requests.
			- _Bearer tokens_ are a type of access token whereby simple possession of the token values provides access to protected resources. No additional information, such as a cryptographic key, is needed to make API calls.
			- The preferred method of authorizing requests is by sending the access token in a HTTP Authorization header
				- ```http
				  GET /tasks/v1/lists/@default/tasks HTTP/1.1 
				  Host: www.googleapis.com
				  Authorization: Bearer ya29.AHES6ZSzX
				  ```
		- ### OAuth Authorization Flows
		  background-color:: green
		  collapsed:: true
			- OAuth2 protocol defines 4 primary "grant types" used for obtaining authorization and also defines an extension mechanism for enabling additional grant types.
			- Every OAuth2 grant type flow has the same goal: _To obtain authorization key/access token, which represents a set of permissions, from the user, and perform something on her behalf._
			- Achieving this goal is a 2-part flow:
				- 1. __Get Access Token__: Acquire the authorization key/access token for the user from the OAuth provider, e.g., Twitter
				- 2. __Use Access Token__: Use the authorization key/access token to perform something by calling a protected API endpoint on behalf of the user, e.g., post a tweet
			- In principle, the Get Access Token flow has 5 steps (as shown in the diagram below):
				- 1. Pre-register Client (App) with OAuth Server to get Client ID/Client Secret
				- 2. OAuth Server authenticates user when she clicks on the App's social login button, which is tagged with Client ID
				- 3. OAuth Server solicits user permission to allow the App to perform something on her behalf
				- 4. OAuth Server sends secret code to App
				- 5. App acquires Key/Access Token from OAuth Server by presenting secret code and client secret
			- ![](../assets/OAuth-Flow-Comparison.png)
		- ### 1. OAuth Flow - Authorization code
		  background-color:: green
		  collapsed:: true
			- ![](../assets/oauth-serverside-flow.png){:height 716, :width 534}
			- This grant type is most appropriate for server-side web applications.
			- After the resource owner has authorized access to their data, they are redirected back to the web application with an authorization code as a query parameter in the URL.
			- This code must be exchanged for an access token by the client application.
			- This exchange is done server-to-server and requires both the `client_id` and `client_secret`, preventing even the resource owner from obtaining the access token.
			- This grant type also allows for long-lived access to an API by using refresh tokens.
			- __When to use it?__
				- Long-lived access is required.
				- The OAuth client is a web application server.
				- Accountability for API calls is very important and the OAuth token shouldn’t be leaked to the browser, where the user may have access to it.
			- __Security Properties__
				- The Authorization Code flow does not expose the access token to the resource owner’s browser.
				- The exchange process only succeeds if a correct `client_secret` is passed with the request, ensuring confidentiality of the access token as long as client security is maintained.
				- because the access token is never sent through the browser— there is less risk that the access token will be leaked to malicious code through browser history, referer headers, JavaScript, and the like
				- Although there is less chance of the access token leaking because it’s not exposed to the browser, many applications using this flow will store long-lived refresh tokens in the application’s database or key store to enable "offline" access to data. There is additional risk when an application requires long-lived offline access to data, as this creates a single point of compromise for accessing data belonging to many users.
			- __How access is revoked?__
			- Applications are not usually informed when a user revokes access, and the specification does not define any way to implement a notification—the app will simply see an error the next time it attempts to use an access token or refresh the token stored for that user.
			- While users can revoke their access manually, some OAuth 2.0 authorization servers also allow tokens to be revoked programmatically. This enables an application to clean up after itself and remove access it no longer needs if, for instance, the user uninstalls the app. Example:
			- `curl "https://accounts.google.com/o/oauth2/revoke?token=ya29.AHES6ZSzF"`
		- ### 2. OAuth Flow - Implicit grant
		  background-color:: green
		  collapsed:: true
			- ![](../assets/oauth-clientside-flow.png){:height 556, :width 355}
			- This is the most simplistic of all flows, and is optimized for client-side web applications running in a browser.
			- The resource owner grants access to the application, and a new access token is immediately minted and passed back to the application using a `#hash` fragment in the URL.
			- The application can immediately extract the access token from the hash fragment (using JavaScript) and make API requests.
			- This grant type does not require the intermediary "authorization code," but it also doesn't make available refresh tokens for long-lived access.
			- __When to use it?__
				- Only temporary access to data is required.
				- The user is regularly logged into the API provider.
				- The OAuth client is running in the browser (using JavaScript, Flash, etc.).
				- The browser is strongly trusted and there is limited concern that the access token will leak to untrusted users or applications.
			- __Limitations__
				- The Implicit Grant flow does not accommodate refresh tokens. If the Authorization server expires access tokens regularly, your application will need to run through the authorization flow whenever it needs access.
			- __Security Properties__
				- In this flow, the application does not store long-lived refresh tokens on a server, limiting the exposure if the server is compromised.
				- Because the access token is sent to the user’s web browser, this flow offers less accountability than the Authorization Code flow.
		- ### 3. OAuth Flow - Resource owner password-based grant
		  background-color:: green
		  collapsed:: true
			- ![](../assets/oauth-resource-owner-flow.png){:height 383, :width 439}
			- This grant type enables a resource owner's username and password to be exchanged for an OAuth access token.
			- It is used for only highly-trusted clients, such as a mobile application written by the API provider.
			- While the user'' password is still exposed to the client, it does not need to be stored on the device.
			- After the initial authentication, only the OAuth token needs to be stored. Because the password is not stored, the user can revoke access to the app without changing the password, and the token is scoped to a limited set of data, so this grant type still provides enhanced security over traditional username/password authentication.
			- __When to use it?__
			  collapsed:: true
				- It is recommended only for first-party "official" applications released by the API provider, and not opened up to wider third-party developer communities.
			- __Security Properties__
			  collapsed:: true
				- Although the application has access to the resource owner’s password, there are still some security benefits to using this flow versus authenticating API calls with a username and password (via HTTP Basic access authentication or similar).
				- Http Basic
					- With Basic authentication, an application needs to have continuous access to the user’s password in order to make API calls.
					- It also requires the user change their password and reenter the new password in all applications which require it, should the user no longer want an application to have access to their data.
				- Resource Owner Password
					- However, if the OAuth Resource Owner Password flow is used, the application only needs access to the user’s credentials once: on first use when the credentials are exchanged for an access token.
					- This means there’s no requirement for the app to store these credentials within the application or on the device, and revoking access is easy as well.
					- ```bash
					  curl -d "grant_type=password" \
					  -d "client_id=3MVG9QDx8IKCsXTFM0o9aE3KfEwsZLvRt" \
					  -d "client_secret=4826278391389087694" \
					  -d "username=ryan%40ryguy.com" \
					  -d "password=_userspassword__userssecuritytoken_" \
					  https://login.salesforce.com/services/oauth2/token
					  ```
					- ```json
					  {
					    "id":"https://login.salesforce.com/id/00DU0000000Io8rMAC/005U0000000hMDCIA2",
					    "issued_at":"1316990706988",
					    "instance_url":"https://na12.salesforce.com",
					    "signature":"Q2KTt8Ez5dwJ4Adu6QttAhCxbEP3HyfaTUXoNI=",
					    "access_token":"00DU0000000Io8r!AQcKbNiJPt0OCSAvxU2SBjVGP6hW0mfmKH07QiPEGIX"
					  }
					  ```
		- ### 4. OAuth Flow - Client credentials
		  background-color:: green
		  collapsed:: true
			- ![](../assets/oauth-client-credential-flow.png){:height 186, :width 530}
			- Most of the OAuth flows are for handling delegated authorization, when a resource owner grants access to an application to access her data. However, there are cases when the client itself owns the data and does not need delegated access from a resource owner, or delegated access has already been granted to the application outside of a typical OAuth flow.
			- Imagine a storage API, such as Google Storage or Amazon S3. You want to build a tool using one of these APIs to access the images or documents. The application needs to read and update these resources, but acting on behalf of the app itself rather than on behalf of any individual user. The application can ask the OAuth authorization server for an access token directly, without the involvement of any end user.
			- Here is an example using Facebook API. If the client credentials are successfully authenticated, an access token is returned to the client
				- ```bash
				  curl -d "grant_type=client_credentials\
				  &client_id=2016271111111117128396\
				  &client_secret=904b98aaaaaaac1c92381d2" \
				  https://graph.facebook.com/oauth/access_token
				  ```
			- __When the Access Token Expires__
				- The Client Credentials flow typically provides a long-lived access token. The authorization server may indicate an `expires_in` time; however, the protocol does not support issuing a refresh token in response to the Client Credentials flow. Instead, the application simply asks for a new access token if the current one expires.
	- ## JSON Web Tokens
	  background-color:: pink
		- TBD
	- ## SAML
	  background-color:: pink
		- TBD
- # 3. Cryptography
  background-color:: yellow
	- Cryptography is the process of hiding or obfuscating data so prying eyes can't understand it.
	- Cryptography is put into practice anytime information requires secure transmission from a sender to a recipient.
	- There are 3 forms of cryptography
		- Hash functions
		  logseq.order-list-type:: number
		- Symmetric cryptography
		  logseq.order-list-type:: number
		- Asymmetric cryptography
		  logseq.order-list-type:: number
	- When a client is interacting with a RESTful web service, it is possible for hostile individuals to intercept network packets and read requests and responses if your HTTP connection is not secure.
	- Sensitive data should be protected with cryptographic services like SSL. The Web defines the HTTPS protocol to leverage SSL and encryption.
	- ## Hash Functions (a.k.a Message Digest)
	  background-color:: pink
		- Hash functions are an irreversible, one-way form of cryptography in which data is protected while the original message becomes irrecoverable.
		- Hashing functions take arbitrary-sized data and output a fixed-length, unique hash value for each provided input. It can only be cracked by attempting to enter every input possible.
		- They turn any amount of data into a fixed-length "fingerprint" that cannot be reversed. If the input changes by even a tiny bit, the resulting hash is completely different.
		- File hashes are used to validate that a file is from a verified publisher while password can be hashed and stored within a database as a hashed value to secure entry.
		- __Cryptographic hash functions__
			- regular hash functions used in hash tables are designed to be fast, not secure
			- used for protecting passwords. Safe even if the password file is compromised, but at the same time, allows to verify that a user's password is correct.
			- example: SHA256, SHA512, RipeMD, and WHIRLPOOL
		- `java.security.MessageDigest` class supports MD5 and SHA digests.
		- ### Salt Hashing
		  background-color:: blue
		  collapsed:: true
			- > Source: https://crackstation.net/hashing-security.htm
			- There are several ways in which even a cryptographic hashed password can also be hacked.
			  collapsed:: true
				- __Dictionary Attacks__
				  collapsed:: true
					- uses a file containing words, phrases, common passwords, and other strings that are likely to be used as a password. Each word in the file is hashed, and its hash is compared to the password hash. If they match, that word is the password.
				- __Brute Force Attacks__
				- __Lookup Tables__
				  collapsed:: true
					- The general idea is to pre-compute the hashes of the passwords in a password dictionary and store them, and their corresponding password, in a lookup table data structure. e.g., https://crackstation.net/
					- extremely effective method for cracking many hashes of the same type very quickly
					  
					  ```bash
					  hash("hello")                    = 2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824
					  # add a random string called 'salt'
					  hash("hello" + "QxLUF1bgIAdeQX") = 9e209040c863f84a31e719795b2577523954739fe5ed3b58a75cff2127075ed1
					  hash("hello" + "bv5PehSMfV11Cd") = d1d3ec2e6f20fd420d50e2642992841d8338a314b8ea157c9e18477aaef226ab
					  hash("hello" + "YYLmfY6IehjZMQ") = a49670c3c18b9e079b9cfaf51634f563dc8ae3070db2c4a8544305df1b60f007
					  ```
					- Lookup tables work only because each password is hashed the exact same way. If two users have the same password, they'll have the same password hashes. We can prevent these attacks by randomizing each hash, so that when the same password is hashed twice, the hashes are not the same.
					- We can randomize the hashes by appending or prepending a random string, called a ***salt***, to the password before hashing.
					- To check if a password is correct, we need the salt, so it is usually stored in the user account database along with the hash, or as part of the hash string itself.
					- The salt does not need to be secret, but should be generated using Cryptographically Secure Pseudo-Random Generator (CSPRG)
					- DO NOT use the username as the salt
					- DO NOT reused the salt for all users. Use a separate random salt string for every user and every time the password is changed.
					- DO NOT use a short salt. To make it impossible for an attacker to create a lookup table for every possible salt, a good rule of thumb is to use a salt that is the same size as the output of the hash function. For example, the output of SHA256 is 256 bits (32 bytes), so the salt should be at least 32 random bytes.
					- DO NOT use double hashing or wacky combination of hashing functions.
					- **Cons**
						- creates interoperability problems
						- makes the hashes less secure, if the hacker gets hold of the open source code ([Kerckhoffs'' principle](https://en.wikipedia.org/wiki/Kerckhoffs%27s_principle) - _A cryptosystem should be secure even if everything about the system, except the key, is public knowledge_)
					- If needed, use a 'wacky' function like HMAC (Hash-based Message Authentication Code).
					  
					  ```bash
					  md5(sha1(password))
					  md5(md5(salt) + md5(password))
					  sha1(sha1(password))
					  sha1(str_rot13(password + salt))
					  md5(sha1(md5(md5(password) + sha1(password)) + md5(password)))
					  ```
				- **Key Stretching**
					- Salt ensures attackers can't use specialized attacks to crack passwords using large collection of hashes. Using high-end GPUs and custom hardware it is possible to generate billions of hashes in seconds. To make these attacks less effective, we can use a technique known as *key stretching*.
					- Key stretching is implemented using a special type of CPU-intensive hash function. e.g., PBKDF2, bcrypt
					- Key stretching may make it easier to run a DoS (Denial of Service) attack on your website. It can be eliminated by making the user solve a CAPTCHA.
	- ## Symmetric Cryptography
	  background-color:: pink
		- ![symmetric_cryptography.png](../assets/symmetric_cryptography_1704643378693_0.png)
		- Symmetric cryptography, also known as **secret key cryptography**, uses a single key for [encryption](https://builtin.com/cybersecurity/encryption-mechanisms) and decryption, relying on a cryptographic algorithm to create a single key for both the sender and recipient.
		- Secret key cryptography can be used for both in-transit data and at-rest data but is largely reserved for at-rest data. Because the secret key can become compromised during the transmission.
	- ## Asymmetric Cryptography
	  background-color:: pink
		- Asymmetric encryption, also known as **public key encryption**, uses separate keys for the encryption and decryption processes, with one key remaining private and the other being shared for public use.
		- ![assymetric_cryptography.png](../assets/assymetric_cryptography_1704643369061_0.png)
		  id:: 659acb1e-7c64-4573-9525-91b14a1c36c4
		- ![image.png](../assets/image_1704643961430_0.png)
		- | **Public key** | **Private key** |
		  | Visible to everyone | Owned only by the message owner |
		  | Using public key, message can ONLY be encrypted. Decryption is NOT possible | Decryption of the encrypted text is possible only with private key|
		  | From public key, one CANNOT derive the private key | From private key, one CAN derive the public key |
	- ## Cipher
	  background-color:: pink
		- Ciphers are cryptographic algorithms that can reversibly transform data using a key. We use them to keep data safe, especially when transferring or storing data, times when data is particularly susceptible to prying eyes.
		- **Types**
			- Public-Private key cipher
			- AES 257-bit cipher
	- ## Miscellaneous
	  background-color:: pink
		- Kerberos - how it works
		- PGP/GPG
		- How to secure passwords - http://www.youtube.com/watch?v=6bR110r-RfY&list=WL08D7E897CDBD47CB (Salt, Bcrypt, Ascrypt, Blowfish algo, AES 256)
- # 4. Session Management
  background-color:: yellow
  collapsed:: true
	- Apache Shiro enables a Session programming paradigm for any application - from small daemon standalone applications to the largest clustered web applications. This means that application developers who wish to use sessions are no longer forced to use Servlet or EJB containers if they don't need them otherwise.
- # Secure Design & Coding
  background-color:: yellow
	- > Notes from book "Pushing Left Like a Boss"
	  https://wehackpurple.com/pushing-left-like-a-boss-part-3-secure-design/
	- Design Flaw vs. Security bug
		- Design Flaw is a design problem and is expensive to fix.
		- Security bug is an implementation problem and cheap to fix.
	- Cost of discovering a flaw late
		- Bug found during requirement = $6
		- Bug found during design = $10
		- Bug found during coding = $100
		- Bug found during testing = $1000
		- Bug found during release = $10,000
	- ## Secure Design Concepts
	  background-color:: pink
	  collapsed:: true
		- **1) Defense in depth (layered protection)**
			- Web app firewall + secure code + secure data store
		- **2) Minimize attack surface**
			- Remove unused resources + code
			- Cleanup commented code
		- **3) Least Privilege**
			- RBAC (role based access control) for both the software and the people who build it
		- **4) Fail-safe or Fail-closed**
			- Always fail back to a known state, preferably an original one
			- Fail-safe = rollback transaction and start again, fail gracefully and log errors.
		- **5) Use existing security controls**
			- Do NOT write your own security controls like encryption, encoding, input sanitization, etc.
		- **6) Never hardcode secrets**
		- **7) Re-authentication for important transaction**
			- To avoid CSRF
			- Step up security
		- **8) Segregation of production data**
		- **9) Threat Modelling**
			- https://martinfowler.com/articles/agile-threat-modelling.html
			- Evil Brainstorming - brainstorming session in search of defining all threats that your application system or product will likely face
		- **10) Source code protection**
			- Security through obscurity (obfuscation)
		- **11) Error handling**
			- Never show stacktrace to users. Hackers can learn what tech stack you are using. Always catch your errors.
		- **12) Logging and alerting**
			- Don't log PII data. Log enough for debugging and auditing purposes.
	- ## Secure Coding Concepts
	  background-color:: pink
		- > Notes from https://infosecwriteups.com/pushing-left-like-a-boss-table-of-contents-42fd063a75bb
		- Guarding against accidental or unintentional misuse of the app
		- Always use the security features in your framework e.g., captcha, anti-CSRF token, session management
	- ## Security Testing Methodologies
	  background-color:: pink
		- ### SAST
		  background-color:: blue
			- **What is SAST?**
				- Static Application Security Testing
				- SAST is open box testing that scans a software application from the inside out before it is compiled or executed.
			- **How it works?**
				- SAST analyzes program source code to identify security vulnerabilities e.g., SQL injection, buffer overflows, XML external entity (XXE) attacks, and other [OWASP Top 10](https://owasp.org/www-project-top-ten/) security risks.
				- SAST testing is integrated in the CI/CD pipelines to detect security errors in the earlier stages of development.
			- e.g., Checkmarx
		- ### DAST
		  background-color:: blue
			- **What is DAST?**
				- Dynamic Application Security Testing
				- DAST is a security testing methodology that involves dynamic analyzing of an application in its running state.
				- It scans software applications in real-time against leading vulnerability sources, like the OWASP Top 10 or SANS/CWE 25, to find security flaws or open vulnerabilities.
			- **How it works?**
				- DAST testing simulates the actions of a malicious actor trying to break into your application from the outside.
				- DAST often works from the outside in, identifying security vulnerabilities by simulating attacks, inspecting HTTP responses and observing the application’s reactions.
			- **Cons**
				- This external approach means it lacks the ability to understand the complex nuances and contexts of the code being run. This, in turn, limits DAST’s effectiveness and accuracy, especially when it comes to modern, complex applications.
				- The existing DAST tools send attacks from an external vantage point and then attempt to determine whether these attacks are successful based on HTTP responses. While effective in certain circumstances, this method doesn’t offer detailed insights about what’s happening inside the code and therefore often misses true vulnerabilities.
			- | **SAST** | **DAST** |
			  | Scans app code at rest | Scans application at runtime | 
			  | open-box testing | closed-box testing |
			  | occurs during the early stages of development | scans a working application post deployment |
			  | CANNOT catch authentication and encryption issues allowing unauthorized access | CAN catch |
			  | Technology dependent | Independent |
		- ### IAST
		  background-color:: blue
			- **What is IAST?**
				- Interactive Application Security Testing
				- IAST is a form of AppSec testing that analyzes applications in a running state but analyzes them from the inside out, rather than from the outside in.
				- combines the functions of both SAST and DAST - it uses a monitoring mechanism (sensor or agent) in the application’s backend to gather information during runtime.
				- provides highly detailed vulnerability descriptions that include the HTTP request, the exact lines of code involved, and the exact flow of data through an application or an API
			- **How it works?**
				- IAST uses instrumentation to monitor an application’s internal operations, interactions with  libraries, connections with backend systems and more, all in real time while the application is being used.
				- ![image.png](../assets/image_1704824876867_0.png)
			- **Pros**
				- IAST-style DAST does not require vulnerabilities to be exploited in order to discover them. Ordinary application traffic, not fuzzing and attack exploits, can be used to find complex vulnerabilities. This opens the world of DAST to anyone, not just experienced AppSec experts. Developers can instantly find vulnerabilities in their code as they do their ordinary quality testing. All quality assurance (QA) testing, including automated test cases, can now do double-duty as both QA testing and security testing at once.
				- allows developers test their application's behaviors at runtime using DAST testing techniques while still monitoring source code execution, like SAST testing. IAST mitigates a significant limitation of the SAST methodology: its inability to follow and test all dependencies, like libraries and frameworks, that modern web applications use
- # Books and Trainings
  background-color:: yellow
  collapsed:: true
	- ## OReilly Spring Security for REST APIs
	  background-color:: pink
	  collapsed:: true
		- [Presentation Slide](/technology/springsecurityforrestapis.pdf)
		- [https://www.katacoda.com/jzheaux/scenarios/local-authentication](https://www.katacoda.com/jzheaux/scenarios/local-authentication)
		- [https://github.com/jzheaux/oreilly-spring-security-rest-apis](https://github.com/jzheaux/oreilly-spring-security-rest-apis)
		- `@SpringBootApplication(exclude = SecurityAutoConfiguration.class)`
		- doing this in a Spring Boot app disables Spring Security. DON'T DO THIS.
		- once the `exclude` is removed, every single url is now protected by Spring Security. Accessing the urls now would throw http 401 unauthorized error.
		- Accessing invalid/non-existent urls also would throw 401 instead of 404 by design. Spring Security uses this approach to avoid hackers from fingerprinting the valid url patterns by trial and error.
		- ### Username-password
		  collapsed:: true
			- Spring Security has a default user (called `user`), but not a default password (by design). However, it generates a new GUID password for every Spring Boot run.
			- `http -a user:0b44ffc6-db54-4220-8b0d-27f55806c602 :8080/goals`
			- Example of using a custom username-password hardcoded in the code. Spring supports JDBC, LDAP also.
			  
			  ```java
			  import org.springframework.context.annotation.Bean;
			  
			  import org.springframework.security.core.userdetails.UserDetails;
			  import org.springframework.security.core.userdetails.UserDetailsService;
			  import org.springframework.security.provisioning.InMemoryUserDetailsManager;
			  
			  @Configuration
			  public class SecurityConfig{
			  @Bean
			  public UserDetailsService userDetailsService(){
			  return new InMemoryUserDetailsManager(
			    User.withDefaultPasswordEncoder()
			      .username("user")
			      .password("pwd")
			      .authorities("app")
			      .build()
			  );
			  }
			  }
			  ```
		- ### Authentication
		  collapsed:: true
			- Spring Security is a set of servlet filters chained together. e.g., Authentication filter, authorization filter, defense filter (`HeaderWriterFilter`)
			  
			  | ![](../assets/spring-security-filter-chain.png) | ![](../assets/spring-security-filter-order.png) |
			  | ![](../assets/spring-security-basic-auth-filter.png) | |
			- Once the user is authenticated successfully, from anywhere in the app one can get the authenticated principal details (e.g., username) using the `SecurityContextHolder` as follows. Another way is to ask Spring to inject it for you.
			  
			  ```java Get authenticated user name - Method 1
			  Authentication auth = SecurityContextHolder.getContext().getAuthentication();
			  String username = auth.getName();
			  ```
			  
			  ```java Get authenticated user name - Method 2
			  @GetMapping("/")
			  public String myMethod(Authentication auth){
			  	String username = auth.getName();
			  }
			  ```
			  
			  ```java Get authenticated user name - Method 3
			  @GetMapping("/")
			  public String myMethod(@AuthenticationPrincipal String owner) {
			  }
			  ```
			  
			  ```java Get authenticated user name - Method 4
			  @GetMapping("/")
			  public String myMethod(@CurrentSecurityContext(expression="authentication.name") String owner) {
			  }
			  ```
		- ### CORS
		  collapsed:: true
			- When a web app running on an origin (say, http://localhost:8081) tries to connect to a web service hosted on a different origin (say http://localhost:8080), then the browser blocks the call by default throwing a CORS error.
			- To whitelist certain origins to make the call, do as follows:
			  
			  ```java
			  @Configuration
			  public class SecurityConfig{
			  @Bean
			  WebMvcConfigurer webMvc(){
			  return new WebMvcConfigurer{
			    @Override
			    public void addCorsMapping(CorsRegistry registry){
			      registry.addMapping("/**/goals")
			              .allowedOrigins("http://localhost:8081")
			              .allowedMethods("GET", "POST")
			              .allowedHeaders("Content-Type")
			              .allowCredentials(true) // to allow browser to pass the credentials entered to be passed to the webservice
			              .maxAge(0); // to disable browser from caching these mappings
			    }
			  };
			  }
			  }
			  ```
			- __Pre-flight request__: A browser sends an `OPTION` request to the server before the actual user request asking _what kind of requests are you ok with me sending, e.g., the origins, http methods, etc._
			- When authentication is enabled in an app, all requests are expected to be authentication including the pre-flight `OPTION` requests. This means the pre-flight requests will fail with 401 error. To avoid this, customize the SecurityFilterChain as follows:
			  
			  ```java
			  @Configuration
			  public class SecurityConfig{
			  @Bean
			  SecurityFilterChain web(HttpSecurity http) throws Exception{
			  return http
			        .authorizeRequests((authz) -> authz.anyRequest().authenticated()) // configuring authorization filter
			        .httpBasic(Customizer.withDefaults())
			        .csrf(Customizer.withDefaults)
			        .cors(Customizer.withDefaults()) // here we are suggesting Spring Security to put this CORS filter with our custom settings in the right place, so that the CORS pre-flight calls are not authenticated.
			        .build();
			  }
			  }
			  ```
		- ### CSRF tokens
		  collapsed:: true
			- Spring security performs CSRF checks before authentication for any non-GET HTTP calls with side-effect (`POST`, `PUT`, `DELETE`). This is to protect the app from cross site forgery even before the app tries to authenticate the request.
			- Since CSRF checks happens earlier than authentication, the app would throw HTTP 403 instead of 401.
			  
			  ```java
			  import org.springframework.security.web.csrf.CsrfToken;
			  import org.springframework.web.bind.annotation.ControllerAdvice;
			  import org.springframework.web.bind.annotation.ModelAttribute;
			  
			  @ControllerAdvice
			  public class CsrfHeaderAdvice {
			  @ModelAttribute("csrf")
			  public CsrfToken token(HttpServletRequest request, HttpServletResponse response) {
			  CsrfToken token = (CsrfToken) request.getAttribute(CsrfToken.class.getName());
			  if (token != null) {
			  response.setHeader(token.getHeaderName(), token.getToken());
			  }
			  return token;
			  }
			  }
			  ```
			  
			  https://github.com/spring-projects/spring-security/blob/main/config/src/main/java/org/springframework/security/config/annotation/web/configuration/HttpSecurityConfiguration.java#L87-L98
- # References
  background-color:: yellow
	- Books
		- RESTful Java with JAX-RS - Chapter 12 - Securing JAX-RS
		- REST in practice - Chapter 9 - Web Security
		- O'Reilly Getting Started with OAuth 2.0
	- Links
		- [Application Security With Apache Shiro](http://www.infoq.com/articles/apache-shiro)
		- Java EE Security - http://docs.oracle.com/javaee/6/tutorial/doc/gijrp.html
		- Encyclopedia of Security - http://www.microsoft.com/mspress/books/sampchap/6429.aspx
		- OAuth Roles - http://tutorials.jenkov.com/oauth2/roles.html
		- mTLS Tutorial https://builtin.com/software-engineering-perspectives/mutual-tls-tutorial