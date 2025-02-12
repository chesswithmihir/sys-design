# Design Tinder
## By gkcs

## Tinder Architecture
1. Store profiles (images, 5 images per user)
2. Recommend matches (who got you smiling bro) (number of active users, don't go into too much details tho lol)
3. Note matches (best feature, you match with someone). (0.1% of number of users actually result in matches)
4. Direct Messaging (feature of Tinder once y'all match)

## Storing images
- File vs Blob (binary large object)
- Images typically are large in size
- databases are good for mutability and transaction guarantees (ACID) and indexing (search) and access control ()
- Are you every going to change the image? no, mutability
- transactions are not required for atomic properties on images. required? no
- are you going to sort through images stored as 1s and 0s BLOB? no
- Files are cheaper and faster storing large objects seperately with vertical partitioning
- images are static so you can build a CDN over this. CDNs allow for faster access.
- in the db, we can have a profileID, imageID, fileURL
- distributed file system is going to handle files.

## client server archtiecture
- We have a client app on the mobile where a user clicks a button to send as a request
- Profile service is attached to a DB for username and password verification
- Assume the profile service can handle all the stuff.
- dont send passowrd btw over the network, send a token which is cryptographically safer.
- but this is all tightly coupled with user information.
- Instead use a gateway service which takes a request and asks the profile service whether this is authenticated or not. and the profile service says yes or no.
- if so direct to correct service, then gets a response, forward response to client
- now we have decoupled systems. If you are going to use message protocols then you have separated protocols everywhere.
- Now that we have a profile service, update description, name maybe, etc
- is it good to store images in same service as profile service?
- i'd say decouple as max as possible. So have a seperate service for other features
- maybe have details, description, age for profile
- image service has a distributed file system as we discussed
- also has database with userID, imageID, imageURL

## DMs
- slideeeeeee
- let's say we have two clients and we just matched with someone, let's have them chat
- instead of the update, have a message to userID 1 to userID 2.
- how do we send a message to this user?
- if you know about HTTP, it's mainly a protocol to talk between two machines and in this way there is always a client and a server.
- This is bad for chat tho, the only way a user can get the message is to poll the server ("hey are there any messages for me")
- you want messages to be pushed to you (peer-to-peer protocol)
- XMPP is a good one for this.
- This message is probably also going to be sent via XMPP.
- Internally, one of the things that can happen is you know the connections, web socket connection or TCP.
- and both clients are doing TCP with the gateway service.
- but this is again too coupled
- have a sessions service
- if you have direct messages, this is good so that it can have userID to connectionID mapping in a db and said messages to a socket.
- If you do match things are looking good with XMPP and TCP and web sockets.

## Matching algorithm
- One of the cons of storing a lot of info on the client's device is that the server should be the source of truth. 
- if you match with someone, send that to the server, and that will probably be sent to the profile service or the matcher service, which keeps a track of userID to userID.
- indexes are put on the userID, and you can duplicate the records A--> B == B-->A
- There will be some communication between the matcher and sessions.
- In general, this looks fine.

## Recommendation Engine
- the big one is finding out is users who i will be interested in
- age, gender, location, etc
- profile service could have in its db, the age, gender, location
- why not put indexes on all 3?
- can't have multiple indexes and have it sorted in multiple ways
- if it's sorted by all 3, when you make a query its only going to use 1 of those
- bc it depends on a query optimizer its out of your control p much, but what we're getting to is optimizing on multiple parameters
- and you can only do it on 1
- either use a nosql db like cassandra and replicate the data in multiple places or amazon dynamo
- second solution is if person is not comfortable moving onto distributed db, you can use something called sharding known as horizontal partitioning
- all users name A - J go to DB 36 for example
- all K - P go to 79 for example.
- When you query the data, must exist in db number so and so.
- partiitoning as a concept is useful.
- one of the ways you can partition is via sharding
- consistent hashing is critical to keeping servers functional.
- what about SPOF (should users k-p fail what happens? have a master slave architecture)
- this is how you think about horizontal partitioning
- convince your interviewer as to why you are choosing sharding which is a bit different from using cassandra or dynamo
- shard on location, could be chunks of a city or a city
- based on that chunk you can pull out data all users within the chunk and search within that share of age and gender

## Final points

