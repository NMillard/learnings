# Microservices fundamentals
An approach to developing a single application as a suite of small services, each running
in its own process and communicating with lighweight mechanisms.  

Better supports Deploy, Update, Replace, Scale (DURS).

## Development life cycle (DLC)
- Requirements
- Plan & design
- Development
- Testing
- Release
- Monitor
- Maintain

## Monolith issues
- New team member productivity
- Growing teams
- Code harder to understand
- No emerging technologies
- Scale for bad reasons
- Huge databases

## Domain Driven Design
Derive services based on domains.  
One way to remove dependencies between contexts is to duplicate the
depending classes into each domain.

### Data synchronization
Immediately consistency is lost.  
Move to eventual consistency by publish an event when data changes.  
Patterns: Capture data change and event sourcing.  

### Communication
Use simple protocols that only sends data, e.g. REST, gRPC, service bus/message broker.  
Communication is typically split into sync and async.  
