#### [Designing resilient systems: Circuit Breakers or Retries?](https://engineering.grab.com/designing-resilient-systems-part-1)
- A request to a service can fail for different reasons: network issues, overload, out of resource, bad deployment/config, bad request. A reliable downstream service need to be aware of them and handle things accordingly.
- A circuit breaker is a thing set up between a service and its upstream service that monitors the traffic:
  - it short circuits (opens) when failures reach a threshold
  - it provides fallback mechanisms to the service if the circuit opens
  - it tries to recover by allowing requests to the upstream service every once in a while
  - it can also include a bulwark (rate limiter to the upstream service)
- Why?
  - be a good user: stop overwhelming the upstream service
  - fail fast on our side and take alternative options, e.g. load of cache, do an estimate, reschedule request at a later time
- Some best practices on configuration:
  - track only 5xx errors to avoid DOS attack from users
  - pros and cons for setting up the same circuit per REST endpoint, per service, per service instance (e.g. host) -- depending on what kind of errors we want to detect

#### [Slack's Architecture](https://www.youtube.com/watch?v=WE9c9AZe-DY)
- Main usability: within a team, members are able to connect to slack, get updated message state since last session, and send+receive messages.
  - db setup: relational with tables like teams, users, channels, messages, emojis
  - scalability and availability: a pair of active-active DCs, sharding done by team (avoid write conflicts and please paid teams)
  - log in: a request to start connection, returns a huge state of the world (the channels and members etc), with a websocket link
  - keeping up to date: using websocket, mostly team member's online/offline status because it's O(N^2); the delay between state and websocket connection needs to be caught up by event log cursors returned by the state
  - sending a message: persisted by the message server and get broadcasted; clients store pending sends in queue
  - async jobs in job queue: url expansion, search index update, imports/exports
- Challenges
  - failure of db: both can fail because of increased load (need better scaling)
  - initial session connect request big for large teams (can do lazy queries)
  - mass reconnects: n users doing O(N^2) session connect requests at the same time (use edge cache close to the customers)

#### [Scaling Slack](https://www.youtube.com/watch?v=o4f5G9q_9O4) -- follow up of Slack's Architecture after ~2 years
- Set up: webapp (auth, API, CRUD etc) and Channel Server (real time websocket service for messages, user status, order of msgs etc)
- Scaling case 1: acking and persisting messages
  - old flow: client sends to CS, CS broadcasts to other users in channel (end of user perceived latency), then CS contacts webapp to persist msg.
    - good: good degraded performance -- users can still send&receive when webapp is down
    - bad: CS needs to maintain states -- buffer uncommitted msgs on disk (for replay after crash), and retry when webapp is down -- which increases complexity and complicates deployment/upgrades
  - new flow: client sends to webapp, webapp queues the request (defer persistency), webapp asks CS to broadcast, CS does it (end of user perceived latency) and ack to webapp, finally webapp returns HTTP 200 to client.
    - requires: stable job queue, better availability of webapp
    - good: CS can now be stateless, user perceived latency still low, crash safe, client can send without setting up a websocket session
- Scaling case 2: websocket initiation (see Challenges in previous article)
  - old flow: client asks webapp to start, webapp gathers info from db and CS, returns huge payload to client, then client connects to CS (and do some catch ups)
    - bad: payload size is huge for large teams (performance), db gets overloaded with mass # of connect requests(reliability), round trip times high (locality)
  - new flow: deploy edge cache at different regions (pre-warmed, application-aware). Client connects to cache service, cache service gathers data from db and CS (opens connection to keep things up to date), and returns a slimmed version of initialization data
- Lessons to take home
  - To increase solution space, work on the end-to-end problem instead of focusing on a single subcomponent.
  - Optimality in design changes with growth.
  
#### [Case Study: Troubleshooting a Live Service: Routing](https://www.ebayinc.com/stories/blogs/tech/sre-case-study-url-distribution-issue-caused-by-application/)
- Problem: a service is getting routed requests for other services behind a load balancer.
- A systematic approach to find the scope of the problem:
  - from the client side (see if the problem comes from direct requests, or the lb by checking source IPs)
  - from the server side (see if the lb misroutes to other services)
  - timing (after the downstream service deployed a new version)
- If you can't find where the problem is, dig wider. If you found something strange that can't be easily explained, dig deeper. 
- Get firsthand raw data to determine the root cause -- in this case take a [tcpdump](https://www.tcpdump.org/manpages/tcpdump.1.html) and view it on [wireshark](https://www.wireshark.org/) (go deeper from level 7).
- In a production environment, the top priority is to mitigate the problem -- usually just roll back to the last known good version.
