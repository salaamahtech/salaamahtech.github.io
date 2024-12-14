- ![Design Latency](../assets/designlatency.gif){:height 732, :width 581}
- # Basics
  background-color:: yellow
	- ## How to reduce latency
	  background-color:: red
		- ### 1. Caching
			- Temporarily storing frequently accessed data in memory to reduce access time.
			- How It Helps:
				- **Data Retrieval**: Fetching data from a cache (e.g., Redis, Memcached) is significantly faster than querying a database.
				- **Content Delivery**: Caching static assets (like images, CSS, JS) reduces the need to retrieve them from the origin server repeatedly.
		- ### 2. Load Balancing
			- Distributing incoming network traffic across multiple servers to ensure no single server becomes a bottleneck.
			- How It Helps:
				- **Resource Utilization**: Balances the load to prevent any single server from becoming overwhelmed, which can slow down response times.
				- **Redundancy**: Provides failover capabilities, ensuring requests are handled promptly even if some servers are down.
		- ### 3. Asynchronous Processing
			- Handling tasks in the background without blocking the main execution thread, allowing the system to continue processing other requests.
			- How It Helps:
				- **Non-blocking Operations**: Users don't have to wait for long-running tasks (like sending emails or processing images) to complete.
		- ### 4. Data Partitioning (Sharding)
			- Dividing a database into smaller, more manageable pieces (shards) that can be distributed across multiple servers.
			- How It Helps:
				- **Parallelism**: Queries can be executed in parallel across shards, reducing the time to retrieve data.
				- **Scalability**: Distributes the load, preventing any single database instance from becoming a bottleneck.
		- ### 5. Content Delivery Networks (CDN)
			- Distributed networks of servers that deliver web content based on the geographic location of the user.
			- How It Helps:
				- **Proximity**: Serves content from servers closest to the user, reducing the physical distance data must travel.
				- **Caching**: Caches static and dynamic content to speed up delivery.
		- ### 6. Database Optimization
			- Tuning databases to perform queries more efficiently through indexing, query optimization, and proper schema design.
			- How It Helps:
				- **Indexing**: Speeds up data retrieval by allowing the database to find records without scanning entire tables.
		- ### 7. Minimizing Network Hops
			- Reducing the number of intermediary steps data must pass through and choosing efficient communication protocols.
			- How It Helps:
				- **Fewer Hops**: Each network hop introduces additional latency; minimizing them speeds up data transmission.
		- ### 8. Parallel and Concurrent Processing
		- ### 9. Prefetching and Predictive Loading
			- Anticipating future data requests and loading them in advance.
			- How It Helps:
				- **Reduced Wait Times**: Data is already available when requested, eliminating retrieval delays.
				- **Smoother User Experience**: Especially effective in applications with predictable access patterns.
- # Caching
  background-color:: yellow
	- ## Locality of References
	  background-color:: pink
		- ### Principle of Locality
		  background-color:: blue
			- Programs tend to reuse data and instructions near those they have used recently, or that were recently referenced themselves
			- ```java Locality Example
			  int sum = 0;
			  
			  for(int i=0; i < a.length; i++){
			  sum += a[i];
			  }
			  
			  return sum;
			  ```
		- ### Temporal Locality
		  background-color:: blue
			- The concept that a resource that is referenced at one point in time will be  referenced again sometime in the near future.
			- In code above, referencing `sum` in each iteration is an example of temporal locality
		- ### Spatial Locality
		  background-color:: blue
			- The concept that likelihood of referencing a resource is higher if a resource near it was just referenced.
			- In code above, referencing array elements in succession is an example of spatial locality
	- ## Caching Solutions
	  background-color:: red
		- Ehcache
		- Guava Caching library
		- JCache (specification?)
		- Distributed Caches
		- Aerospike
		- Coherence (Oracle)
		- Gemfire
		- Gigaspaces
		- Hazelcast
		- HBase with BlockCache
		- Memcached
		- Redis
		- Riak (key-value database)
	-
	- For the remote cache layer, there are two possibilities:
		- A distributed memory caching solution, such as Memcached, to distribute the data across a cluster of nodes.
		- Setting up HBase so that all needed records can be found in the block cache. The block cache keeps data blocks in memory, where they can be quickly accessed.
	- ## Distributed memory caching
	  background-color:: red
		- A distributed memory solution like Memcached or Redis simplifies the work of developing a caching layer. In terms of performance, though, it still requires a network call, which can add a small amount of latency to requests. Request times should be in the 1- to 4-millisecond range. The advantage of this solution over the partitioning solution is that we won’t have downtime when nodes fail, since we can set up the caching solution with multiple replicas of the data.
		- The only downside of the distributed memory caching solution is you need enough memory to hold everything. If you can’t hold everything, you need an additional persistence store backed by disk, which means an additional call when data is not in memory. As we’ll see shortly, if we’re utilizing HBase, there is little reason why you need to also use a distributed caching solution.
	- ## Caching Strategies
	  background-color:: red
		- https://www.prisma.io/dataguide/managing-databases/introduction-database-caching
		- **Cache-aside pattern**
		  logseq.order-list-type:: number
			- Cache-aside is a caching architecture that positions the cache outside of the regular path between application and database. In this arrangement, the application will fetch data from the cache if it is available there. If the data is not in the cache, the application will issue a separate query to the original data source to fetch the data and then write that data to the cache for subsequent queries. The minimal crossover between the cache and backing data source allows this architecture to be resilient against unavailable caches. Cache-aside is well-suited for read-heavy workloads.
			- ![Cache-Aside](../assets/cache-aside.png)
		- **Read-through pattern**
		  logseq.order-list-type:: number
			- Read-through caching is a caching strategy where the cache is deployed in the path to the backing data source. The application sends all read queries directly to the cache. If the cache contains the requested item, it is returned immediately. It the cache request misses, the cache fetches the data from the backing database in order to return the items to the client and add it to the cache for future queries. In this architecture, the application continues to send all write queries directly to the backing database.
			- ![Read-Through](../assets/read-through.png)
		- **Write-through pattern**
		  logseq.order-list-type:: number
			- Write-around caching is a caching pattern where write queries are sent to the backing database directly rather than written to the cache first. Because any items in the cache related to the update will be now be stale, this method requires a way to invalidate the cache results for those items for subsequent reads. This technique is almost always combined with a policy for cache reads to control read behavior. This approach is best for data that is read infrequently once written or updated.
			- ![Write-Through](../assets/write-through.png)
		- **Write-back pattern**
		  logseq.order-list-type:: number
			- Write-back caching is a caching method where write queries are sent to the cache instead of the backing database. The cache then periodically bundles the write operations and sends them to the backing database for persistence. This is a modification of the write-through caching approach to reduce strain caused by high throughput write operations at the cost of less durability in the event of a crash. This ensures that all recently written data is immediately available to applications without additional operations, but can result in data loss if the cache crashes before it's able to persist writes to the database.
			- ![Write-Back](../assets/write-back.png)
		- **Writer-around pattern**
		  logseq.order-list-type:: number
			- Write-through caching is a caching pattern where the application writes changes directly to the cache instead of the backing database. The cache then immediately forwards the new data to the backing database for persistence. This strategy minimizes the risk of data loss in the event of a cache crash while ensuring that read operations have access to all new data. In high write scenarios, it may make sense to transition to write-back caching to prevent straining the backing database.
			- ![Write-Around](../assets/write-around.png)
			-