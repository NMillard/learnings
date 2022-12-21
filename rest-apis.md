# Representational State Transfer
REST is not a prescription on how to build APIs, it's an architectural style, i.e. a named set of constraints.
REST was coined back in 2000, where it made more sense to transfer large data objects, rather than many small ones, due to the internet was mostly a high latency network.  

Fielding's disseration was build around hypermedia, and was written prior to Web APIs becoming a standard.

## Constraints
- Client-server relationship
    REST doesn't specify what the interaction should look like, or how and where data get stored.
- Stateless
    Servers must not store session state. Each request must contain all information needed to process it.
    State is the client's responsibility and must be transferred on each request.
- Cachable
    Resources can declare themselves cachable or not - but this is a highly debated field.
- Uniform interface
    e.g. each resource is locatable and each endpoint takes the same type of data (e.g. json) with a commonly defined set of naming conventions and formats.
- Layered system
    A system's architecture can be composed of many hierarchical layers and components, but each component cannot see beyond its immediate layer with which it's interacting.


## REST elements
**Resources and representation**
A resource is a "named thing" with a URI and conveys a concept. How the concept is stored can be entirely different as to how it's represented.  
You essentially have a few important aspects, a **concept** that is decoupled from its **content**, and the **representation** is further decoupled from how it's **stored**. The underlying content of a concept can change while the concept remains the same, just like one resource may have multiple representations.

In HTTP content negotiation is typically used for multiple representations, e.g. json, xml, csv.  

## Cookies
Cookies represent site-wide state information and fails to match REST's model of application state. To be "RESTful", each request must contain all information needed to process it.