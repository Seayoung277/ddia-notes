# Chapter 1

System design components: database, cache, search/index, stream/messaging, batch processing

Goals of system design: reliable, scalable, maintainable

## Reliability

Definition: system continues to work correctly, even when things go wrong
Or: system is resilient or fault-tolerant when system fault happens

It is impossible to reduce the probability of faults to 0, so it's best to design fault-tolerance systems than prevent faults from happening, except non-curable faults like security faults

### Hardware Faults

Solutions:
- Hardware redundancy
- Software techniques (e.g. ?)

### Software Errors

Systematic software errors are hard to anticipate, more likely to cause large scale failures, and cascading failures

Things to help reduce software errors:
- Be careful about assumptions
- Thorough testing
- Process isolation
- Allowing processes to crash and restart (self healing)
- Measuring and monitoring

### Human Errors

How to prevent human errors:
- Minimize opportunities for error (abstractions, APIs, admin interfaces)
- Decouple environments that are most likely to make mistakes (sandbox)
- Thorough testing at all levels
- Quick and easy recovery (fast rollback, gradual rollout)
- Detailed and clear monitoring
- Good management and training

## Scalability

Definition: system's ability to cope with increased load and maintain performance
- When load parameter is increased, how is the performance affected
- When load parameter is increased, how much resource to increase to maintain the same performance

### Load

How to describe load: load parameters, e.g.:
- Requests per second of a web service
- Ration of reads to writes in database
- Number of simultaneously active users
- Cache hit rate

### Performance

How to describe performance:
- Throughput (batch processing system like Hadoop)
- Response time (online systems)
    - Average
    - Percentile (median/p95/p99)
        - Commonly used in Service Level Objectives(SLO) and Service Level Agreement(SLA)
        - Queueing delays often accounts for long response time (head-of-line blocking)
        - Load generator should keep sending requests independently of response time

### Coping with Load

2 ways of scaling:
- Scale up: vertical scaling, using more powerful machine
- Scale out: horizontal scaling, distributing the load across multiple smaller machines
    - Elastic systems can automatically add computing resources
    - Stateless services are easier to scale out than stateful services

## Maintainability

3 principles of designing maintainable systems:
- Operability: Easy to keep the system running smoothly
- Simplicity: Easy for new engineers to understand
- Extensibility: Easy to make changes in the future, adapting for unanticipated use cases

### Operability

Approaches to improve Operability:
- Good monitoring to provide visibility of runtime
- Good support for automation with standard tools
- Avoid dependency on individual machines
- Good documentation and easy to understand instructions/run books
- Good default behavior, and freedom to override default when needed
- Self-healing where appropriate, and manual control over the system state
- Exhibiting predictable behavior, minimizing surprises

### Simplicity

A simple system should not be simple in functionality. A simple system should reduce its accidental complexity, which arises only from the implementation, not the problem that it solves

The best tool for removing accidental complexity is abstraction, which hide the implementation details.