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
