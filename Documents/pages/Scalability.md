# Basics
	- ## Scale Up / Vertical Scaling
		- When volume of data increases, add computing power to a single server or move to a bigger server.
		- Pros: no change in architecture needed.
		- Cons: there are limitation on how big a single host can be.
	- ## Scale Out / Horizontal Scaling
		- Processing is handled by more than 1 server. When data volume increases, add more servers to the farm.
		- Pros: Cheaper purchase costs than scale up, High availability
		- Cons: complex data processing stratagies involved.
		- Features: smart-software-dumb-hardware, move-processing-not-data.
		- Challenges: Bottlenecks, increased risk of failure
	- > [Horizontal and Vertical Scaling Strategies for Batch applications](http://www.ontheserverside.com/blog/2014/07/23/horizontal-and-vertical-scaling-strategies-for-batch-applications)
- # Options for increasing the DB performance
  collapsed:: true
	- ## 1. Vertical Scaling
		- **CPU Upgrades**
			- typically this is due to slow read queries. Optimizing those offensive queries can most of the times solve the issue.
			- If the CPU is high due to too many users, then typically scaling out is the only option.
			- If the memory is low, CPU usage can be high due to *disk swapping*.
		- **Memory Upgrades**
			- Memory is heavily used for caching results, index caching, sorting, aggregation, etc.
			- If the memory runs low, CPU automatically swaps the contents to disk which is typically 100 times slower. Adding more memory generally solves the problem, but there is a limit.
			- Highly inefficient queries also sometimes contribute to high memory usage.
		- **Disk Upgrades**
			- Full table scan queries or high user/transaction volume lead to high disk usage.
			- Using RAID subsystems or Solid State Drives (SSD) sometimes solve the issue. But upgrading disks rarely solve performance issues.
	- ## 2. Read Scaling
		- Read scaling is a simple technique of creating read-only replicas of the monolithic DB server to reduce the read-only queries on a single DB.
	- ## 3. Horizontal Scaling
		- When your application involves heavy data reads/writes and heavy transactional volumes, and if none of the above techniques work, then scaling horizontally is the only option.
		- Partitioning/Sharding data across multiple nodes/servers in a cluster also introduces multiple failure points. DB cluster must be *highly available- to ensure the interim server failures do not interrupt the live operations.
- # Sharding
  background-color:: gray
	- ## What is Sharding?
	  background-color:: pink
		- Sharding is a form of scaling known as **horizontal scaling** or **scale-out**, as additional nodes are brought on to share the load.
		- **Sharding** is a method for distributing a single large dataset across multiple databases, which can then be stored on multiple machines. This allows for larger datasets to be split into smaller chunks and stored in multiple data nodes, increasing the total storage capacity of the system.
		- by distributing the data across multiple machines, a sharded database can handle more requests than a single machine can.
	- ## Sharding Strategy
	  background-color:: pink
		- ### Range-based sharding
		  background-color:: blue
			- **Ranged sharding** divides data into ranges based on the shard key values. Each chunk is then assigned a range based on the shard key values.
			- ![image.png](../assets/image_1703261265746_0.png)
		- ### Simple hashed sharding
		  background-color:: blue
			- **Hashed Sharding** involves computing a hash of the shard key field’s value. Each chunk is then assigned a range based on the hashed shard key values.
			- ![image.png](../assets/image_1703261280417_0.png)
		-
		- ### Consistent hashing or Hash Ring
		  background-color:: blue
			- is a special kind of hashing such that when a hash table is resized and consistent hashing is used, only `K/n` keys need to be remapped on average, where `K` is the number of keys, and `n` is the number of slots. In contrast, in most traditional hash tables, a change in the number of array slots causes nearly all keys to be remapped.
			- The advantage of Consistent Hashing with sharding is to reduce the number of rows affected (i.e., that need to be moved) as new physical shard servers are added or removed.
			- http://highscalability.com/blog/2023/2/22/consistent-hashing-algorithm.html
			- [Consistent Hashing Explained](http://michaelnielsen.org/blog/consistent-hashing/)
	- ## Sharding Types
	  background-color:: pink
		- ### Black-Box Sharding
		  background-color:: blue
			- The most common sharding technique in existence is black-box sharding, meaning that the shard distribution logic is controlled internally by the toolkit or product, and not exposed to the application.
			- The primary drawback for the black-box sharding approach is when you need to obtain related data, such as lists of items that have to do with a particular data element. This is often referred to as a *scatter-gather approach*: the data is scattered by key across the cluster, and must be gathered into meaningful lists of related data
		- ### Relational Sharding
		  background-color:: blue
			- With relational sharding, the application or database architect defines the sharding schema along the natural data relationships in the data model. The advantage is that related data is *co-located* in the same physical server, allowing more application queries to be resolved with a single invocation to a given shard server.
			- The sharding strategy for the relational approach also uses a hash function to partition the data, again typically using a modulus or consistent hash approach.
			- Allows easy join of related data unlike in black-box sharding.
	- ## Replication vs. Sharding
	  background-color:: pink
		- Replication is taking copies of a given DB as-is. However, sharding is splitting the large dataset into multiple smaller chunks stored across multiple nodes.
		- Replication is much simpler to setup than sharding.
		- **Read-focused vs. Write-focused load**:
			- Replication works well if your data workload is primarily read-focused. It increases availability and read performance by simply spinning up additional copies of the DB and through load balancing.
			- Sharding works well if your DB contains large amounts of data, requires high read and high write volume, and have specific availability requirements.
- # Caching
  collapsed:: true
	- ## Locality of References
		- ### Principle of Locality
			- Programs tend to reuse data and instructions near those they have used recently, or that were recently referenced themselves
			- ```java Locality Example
			  int sum = 0;
			  
			  for(int i=0; i < a.length; i++){
			  sum += a[i];
			  }
			  
			  return sum;
			  ```
		- ### Temporal Locality
			- The concept that a resource that is referenced at one point in time will be  referenced again sometime in the near future.
			- In code above, referencing `sum` in each iteration is an example of temporal locality
		- ### Spatial Locality
			- The concept that likelihood of referencing a resource is higher if a resource near it was just referenced.
			- In code above, referencing array elements in succession is an example of spatial locality
	- ## Caching Solutions
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
		- A distributed memory solution like Memcached or Redis simplifies the work of developing a caching layer. In terms of performance, though, it still requires a network call, which can add a small amount of latency to requests. Request times should be in the 1- to 4-millisecond range. The advantage of this solution over the partitioning solution is that we won’t have downtime when nodes fail, since we can set up the caching solution with multiple replicas of the data.
		- The only downside of the distributed memory caching solution is you need enough memory to hold everything. If you can’t hold everything, you need an additional persistence store backed by disk, which means an additional call when data is not in memory. As we’ll see shortly, if we’re utilizing HBase, there is little reason why you need to also use a distributed caching solution.
- # References
	- Understanding Big Data Scalability - Cory Isaacson