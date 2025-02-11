# System Design Interview Notes

## Basics
- You can expose your code via an API, and you will get requests for this API and you will send a response that your computer sends back.
- Setting up this computer might need a DB, endpoints, and resolience if power fails.
- You should back this up on the cloud to have a set of computers that save your data like AWS
 
## Horizontal vs Vertical Scaling
- What happens if you have too many customers polling your API with too many requests?
- Can buy a bigger machine (Vertical Scaling)
- Can buy more machines (Horizontal Scaling)

| Horizontal Scaling | Vertical Scaling |
| --- | --- | 
| Load Balancing Required | N/A |
| Resilient | Single point of failure | 
| RPCs (slow) | IPC (fast) |
| Data inconsistency | consistent data | 
| Scales well with users | Hardware Limit |

Use both in the real world as a hybrid

## System Design for Distributed Systems
- Pizza parlor where you can have 1 chef and you can make it as good as possible, but when it's sick we found a SPOF (vertical scaling)
- can horizontally scale with a team of chefs
- Keep backups and avoid SPOF
- also do preprocessing before hand. If there is an initialization step like making the bread of the pizza that can be done asynchronously, prepare this before hand at a non peak hour
- employ a microservice architecture for your pizza orders such that a team of specialized people work on garlic pizza, and another group works on cheese. This is also good for scaling individual processes that can be decoupled and also polling the status of an order doesn't involve polling everyone, but only the allocated chefs that work on the process.

| Category | 	Monolithic Architecture |	Microservice Architecture
| -- | -- | -- |
| Design |	Single unified application	| Collection of independent, loosely coupled services |
| Codebase	| Single codebase	| Multiple codebases, one for each microservice |
| Deployment |	Deployed as a single unit	| Each microservice is deployed independently |
| Scaling	| Scale the entire application	| Scale individual microservices based on demand |
| Fault | Isolation	One failure can crash the entire application	| Failures are isolated to individual services |
| Technology | Stack	Typically one stack for the whole application	| Can use different technologies for each microservice |
| Development | Speed	Slow, because the entire application needs to be redeployed |	Faster, since services can be deployed independently |
| Communication |	Internal function calls between components	| Network-based communication (e.g., HTTP, RPC, messaging) |
| Testing	| Harder to test the entire system due to tight coupling	| Easier to test individual services but integration testing is complex |
| Operational | Complexity	Simpler to deploy and manage, but becomes harder as app grows |	Complex due to multiple services to manage |

- This is actually building a distributed system so that there is fault tolerance and resilience.
- Keep the system extensible. Decouble everything to make your system extensible

## Consistent Hashing and Load Balancing
- Person sends a request to your Server, your server will send a response.
- Now let's say you have thousands and thousands of requests. The problem is your computer can't handle this load.
- If there is a second person, where should the request go. You want to balance the load on these servers.
- Each server has a load on it. And the process of taking servers and balancing load on each of them is called load balancing.
- Consistent Hashing will help us do this.
- Request ID = RID {0, 1, ..., M - 1}
- we will hash this request id, hash(rid) = m1 and then mod into the write server. m1 % n.
- What happens if we need to bring more servers. Everything gets changed up. modding from m1 % n vs m1 % (n+1) changes everything.
- This is really bad because the old hashing technique displaces everything which means caches in individual servers are rendered useless when user data gets moved to a different bucket because the hash changes.
- Consistent Hashing is a really nice technique in we minimize the amount of change, while still employing hashing for load balancing.
- Consistent Hashing is mainly determined by way of a virtual ring: In consistent hashing, both servers and keys are mapped onto a virtual ring (also known as a "hash space" or "hash ring").
The hash function is used to assign each server and key to a position on this ring. For instance, using a simple hash function, server A might be placed at position 10, server B at 40, and so on.
- Assigning Keys: Each key is hashed and placed on the same ring.
- A key is assigned to the next server in a clockwise direction on the ring. If a key is hashed to position 25, and servers are at positions 10 and 40, the key will be assigned to the server at position 40.
- Handling Server Changes:
- When a server is added, only the keys that would have been assigned to this server (based on its position in the ring) are redistributed. All other keys remain with their original servers.
- When a server is removed, only the keys assigned to this server are redistributed to the next server in the ring.
- **TODO**

## Caching in distrubuted systems
- let's say you have a user on Instagram, who's asking for their feed.
- The request reaches the server. the server queries the db: for this user, select all posts from all following.
- the response comes back from db to server to client. Overall time is 220 ms (100 ms client to server, 10 ms server to db, 10 ms db to sever, 100 ms server to client)
- Now one important thing, is that the server may deliver the same content as a response to more than 1 user so long as their interests are similar.
- In this way you can group users into a single cohort and give them similar news feeds. Now when 1 user from this cohort, asks for a news feed, cache the response in the server so that when another user comes, the server can pull from cache and it takes 202 ms assuming cache takes 1 ms to hit.
- And while it doesn't look like a lot now, this can be extended to the mobile device you have so that you can cache results on the client's device itself.
- When you fetch your news feed, you can reuse the news feed.
- be smart with cache eviction policies. (LRU, LFU, etc)
- **Thrasing** occurs when you are evicting elements from your cache that are being polled and if you have a cyclic request ABCD when your cache can only fit 3 elements, then its cooked
- Another problem of caches is eventual consistency. If you have a copy of the data, then the copy has to be updated along with the original source of truth. In most cases, the DB is the source of truth. Cache has a stale copy or dirty copy.
- Let's say you are seeing the number of likes on a YouTube video. For every added Like, there is a query to the db, but maybe the cache is updated every minute or hour, it doesnt need to have the latest value in cache, That helps reduce the work on the cache. But the drawback here is that the data is just not true.
- For financial systems, you might see stale entries which could cause problems. This is known as eventual consistency as determined by your policy.
- You can place cache honestly anywhere, global cache, server cache, db cache, etc.
- In a large scale production system you do all 3. DBs have a super small cache.
- But what you want is a distributed cache for a large scale distributed system, and the reason for this is, the cache can scale independently like redis.

## SPOF in Distributed Systems
- if a db crashes, the entire system crashes
- the easiest way to mitigate this is to add another node. For instance, let's say you had a profile server on your app. Just duplicate and make another profile server. One way to set this up is to have the second node as a backup. For a server, this isn't so useful bc it's empty if no one is connected to it. On the other hand if this was a database, and every change is mirrored onto the next database, then the db is truly a backup of the data. backup services don't make much sense. When there is a copy of the data, you can say the database is more resilient than before.
- If a server fails, that's bad, so make a backup server, for dbs, have a master slave architecture.
- back to the server, add a load balancer to direct traffic accross servers
- the load balancer itself is a SPOF, so add more load balancers.
- Because of this the cleint may not know which load balancer to connect to.
- In this way, we add the load balancer onto a DNS, and the client is going to connect to the DNS
- We need to have multiple IP addresses being resolved under the same host.
- For example, if we want to go to www.facebook.com, then the corresponding IP address should be one of the many load balancers
- To call it a load balancer now is misleading which is why we call it a gateway,
- are we done? hell no!
- if the entire system is located in 1 location, it can all fail with 1 power outage
- use many regions

## Content Delivery Networks
- let's say some users want to connect to your website
- they first resolve the DNS address, and ask what the domain name's IP is. Now you try to connect to that IP address
- In a simple system, this IP address belongs to a web server which has web pages (HTML, CSS, JS)
- you can keep a cache on this server to return static pages that can be returned quickly.
- The problem is when you have multiple clients, especially all over the world
- there is no single server that's quick to connect to for the average person.
- soln: Take the cache and distribute it across the globe
- This entire solution is called a CDN or content delivery network
- CDNs are made by large companies, boxes are close to users, follow regulations, content updated by server.
- Amazon CloudFront is an example of a CDN and has good integration with S3.



