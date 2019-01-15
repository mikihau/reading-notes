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