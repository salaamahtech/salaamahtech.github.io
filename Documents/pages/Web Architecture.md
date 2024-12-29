# Architecture Styles
background-color:: yellow
	- ## Micro Frontends
	  background-color:: pink
	  collapsed:: true
		- [[Micro Frontend]]
	- ## Single Page App (SPA)
	  background-color:: pink
	  collapsed:: true
		- Single-page applications (SPAs) consist of a single or a few JavaScript files that encapsulate the entire frontend application, usually downloaded up front. When the web servers or the content delivery network (CDN) serves the HTML index page, the SPA loads the JavaScript, CSS, and any additional files needed for displaying any part of our application.
		- Pros
			- client downloads the application code just once, at the beginning of its life cycle, and the entire application logic is then available up front for the entire user’s session.
		- Cons
			- Downloading all the app logic in one-go
				- First time load time could be longer
				- potential memory leaks when not implemented properly
				- failures in low end devices like smart TV or set-top-box
			- SEO - web crawlers won’t have an easy job indexing all the contents served by an SPA unless we prepare alternative ways for fetching it. Usually, when we want to provide better indexing for an SPA, we tend to create a custom experience strictly for the crawler. For instance, Netflix lowers its geofencing mechanism when the user-agent requesting its web application is identified as a crawler rather than serving content similar to what a user would watch based on the country specified in the URL.
	- ## Progressive Web App (PWA)
	  background-color:: pink
	  collapsed:: true
		- **Service workers**
			- ==Cache First/Offline First Pattern==
				- Using service workers, we can now create our caching strategy for a web application, with native APIs available inside the browsers. This pattern is called **offline first**, or **cache first**,
				- If a resource is cached and available offline, return it first before trying to download it from the server.
				- If it isn’t in the cache already, download it and cache it for future usage.
	- ## Isomorphic applications
	  background-color:: pink
	  collapsed:: true
		- Isomorphic or universal applications are web applications where the code between server and client is shared and can run in both contexts. It is particularly beneficial for the time to interaction, A/B testing, and SEO.
		- These web applications share code between server and client, allowing the server, for instance, to render the page requested by the browser, retrieve the data to display from the database or from a microservice, aggregate it, and then prerender it with the template system used for generating the view in order to serve to the client a page that doesn’t need additional round trips for requesting data to display.
		- Because the page requested is prerendered on the server side and is partially or fully interpreted on the backend, the time to interaction is enhanced. This avoids a lot of round trips on the frontend, so we won’t need to load additional resources (vendors, application code, etc.), and the browser can interpret a static page with almost everything inside.
		- Pros
			- improved SEO since the page is rendered in the server
			- A/B testing - because you can prerender on the server the specific experiment you want to serve to a specific user.
		- Cons
			- for high traffic apps, scalability could be an issue. If the responses are highly cacheable, then using CDN could help avoid hitting the server for each request.
	- ## Jamstack (JS, APIs, Markup)
	  background-color:: pink
		- Jamstack is intended to be a modern architecture to help create fast and secure sites and dynamic apps with JavaScript/APIs and prerendered markup, served without web servers.
		- The artifact can be served directly by a CDN since the application doesn’t require any server-side technology to work.
		- One of the simplest ways for serving a Jamstack application is using [GitHub pages](https://pages.github.com/) for hosting the final result.
		- In this category, we can find popular solutions like [Gatsby.js](https://www.gatsbyjs.org/), [Next.js](https://nextjs.org/), or [Nuxt.js](https://nuxtjs.org/).
	- ## Backend For Frontend (BFF)
	  background-color:: pink
		- TBD
- # Web Design
  background-color:: yellow
	- ## Angular vs. React
	  background-color:: red
	  collapsed:: true
		- | **Factors** | Angular | React | 
		  | --- | ---| --- |
		  | Type | Framework | Library |
		  | MVC | Supports M, V, C | Supports only V |
		  | State Management | Built-in | Redux library needed. Global state for all components |
		  | Routing | Built-in | React Router library |
		  | API calls | Built-in | Helmet library |
		  | Form validation | Built-in | Not supported |
		  | Dependency Injection | Built-in | Not supported |
		  | Data Binding | Bi-directional. Many watchers are created automatically by Ng | Unidirectional. No watchers |
		  | Language | TS or JS | JSX |
		  | Change Rendering | Real DOM | Virtual DOM |
		  | Design System | Built-in Material Design toolset | Material-UI library needed. |
		  | Server side rendering | Angular Universal | Next.js |
		  | Mobile development | React Native / Cordova / Flutter | Ionic / NativeScript |
	- ## Responsive Web Design
	  background-color:: red
	  collapsed:: true
		- https://www.smashingmagazine.com/2011/01/guidelines-for-responsive-web-design/
		- **Why?**
			- Avoid web designing for each device separately. More devices are coming out every year in different sizes and resolutions. It is not practical to design for each one of them separately. The same device can switch from portrait to landscape mode.
		- **What?**
			- Achieving adjustable screen resolutions, adjustable screen sizes by using practices like
				- flexible grids and layouts,
				- flexible images
					- The **maximum width** of the image is set to 100% of the screen or browser width, so when that 100% becomes narrower, so does the image.
					- ```css
					  img { max-width: 100%; }
					  ```
				- intelligent use of CSS media queries, etc.
			- | ![rwd1.jpg](../assets/rwd1_1687440702490_0.jpg){:height 308, :width 550} | ![rwd2.jpg](../assets/rwd2_1687440763761_0.jpg) |
		- Viewport
			- The viewport is the user's visible area of a web page.
			- HTML 5 introduced `<meta>` tag to control the viewport
				- ```html
				  <meta name="viewport" content="width=device-width, initial-scale=1.0">
				  ```
- # Load Balancers
  background-color:: yellow
  collapsed:: true
	- ![](../assets//reverse_proxy.png)
	- ## Overview
	  background-color:: red
		- From a physical point of view, it can be plugged anywhere in the architecture:
			- in a DMZ
			- in the server LAN
			- as front of the servers, acting as the default gateway
			- far away in an other separated datacenter
	- ## Why load balancers?
	  background-color:: red
	  collapsed:: true
		- HTTP is not a connected protocol and is stateless
		- When there are multiple app servers, the user may potentially hit a different server for every request and a different session on every server.
		- Possible solutions are
			- 1. Use a ***clustered web application server*** where the session are available for all the servers
				- Pros
				- Cons
					- Only certain products like Weblogic, Tomcat, JBoss, allow to create a cluster
					- very complex to set up and maintain
			- 2. ***Sharing user’s session information*** in a database or a file system on application servers
				- Pros
					- simple to share session via DB or shared file system
				- Cons
			- 3. Use IP level information to maintain ***affinity*** between a user and a server
				- An easy way to maintain affinity between a user and a server is to use user’s IP address: this is called ***Source IP affinity***.
				- Pros
				- Cons
					- works only if a user use a single IP address or never change his IP address during the session.
					- wouldn't work for multiple users who are behind a single proxy
			- 4. Use application layer information to maintain ***persistance*** between a user and a server
				- Store the user information details in a _Session Cookie_, either set by the load-balancer itself or using one set up by the application server.
				- Pros
				- Cons
	- ## Affinity vs. Persistence
	  background-color:: red
	  collapsed:: true
		- __Affinity__: this is when we use an information from a layer below the application layer to maintain a client request to a single server
		- __Persistence__: this is when we use Application layer information to stick a client to a single server
		- sticky session: a ___sticky session___ is a session maintained by persistence
		- The main advantage of the persistence over affinity is that it’s much more accurate, but sometimes, persistence is not doable, so we must rely on affinity.
		- Using persistence, we mean that we’re 100% sure that a user will get redirected to a single server.
		- Using affinity, we mean that the user may be redirected to the same server
	- ## Load Balancer Types
	  background-color:: red
	  collapsed:: true
		- ### 1. Hardware load balancer (Level 4 load-balancers)
			- Layer 4 (transport level). e.g.,: TCP and UDP protocols are transport level.
			- A layer 4 load-balancer takes routing decision based on IPs and TCP or UDP ports.
			- It has a packet view of the traffic exchanged between the client and a server which means it takes decisions packet by packet.
			- The layer 4 connection is established between the client and the server.
			- It is really fast but can’t perform any action on the protocol above layer 4.
			- The fastest layer4 load-balancers uses an ASIC to take routing decision.
			- Example: F5 BIG-IP, Citrix Netscaler, etc.
			- Expensive
		- ### 2. Software load balancer (Level 7 load-balancers)
			- Layer 7 (application level) e.g.,: HTTP, FTP, SMTP, DNS protocols are application level.
			- A layer 7 load-balancer takes routing decision based on IPs, TCP or UDP ports or any information it can get from the application protocol (mainly HTTP).
			- The layer 7 load-balancer acts as a proxy, which means it maintains two TCP connections: one with the client and one with the server.
			- The packets are re-assembled then the load-balancer can take a routing decision based on information it can find in the application requests or responses.
			- Even if this kind of processing seems slow, it is not that much: less than the millisecond.
			- Cheaper
	- ## Hardware load balancer architectures
	  background-color:: red
		- ### 1. NAT or routed mode
		- ### 2. Direct server return or Gateway mode
		- ### 3. IP Tunnel mode
	- ## Software load balancer architectures
	  background-color:: red
		- ### 1. Proxy mode
		- ### 2. Transport proxy mode