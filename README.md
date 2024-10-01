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
- 
