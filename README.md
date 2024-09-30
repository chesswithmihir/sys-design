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
Design |	Single unified application	| Collection of independent, loosely coupled services
Codebase	| Single codebase	| Multiple codebases, one for each microservice
Deployment |	Deployed as a single unit	| Each microservice is deployed independently
Scaling	| Scale the entire application	| Scale individual microservices based on demand
Fault | Isolation	One failure can crash the entire application	| Failures are isolated to individual services
Technology | Stack	Typically one stack for the whole application	| Can use different technologies for each microservice
Development | Speed	Slow, because the entire application needs to be redeployed |	Faster, since services can be deployed independently
Communication |	Internal function calls between components	| Network-based communication (e.g., HTTP, RPC, messaging)
Testing	| Harder to test the entire system due to tight coupling	| Easier to test individual services but integration testing is complex
Operational | Complexity	Simpler to deploy and manage, but becomes harder as app grows |	Complex due to multiple services to manage
