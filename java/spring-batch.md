# Spring Batch

The spring batch framework supports processing of large amounts of structured data. Jobs run periodically to process the input data. The output gets written to a store or provided as input for another job.  

Batching is commonly done for data transformations and system integrations.


The Spring framework also supports streaming processing, but needs a different library dependency for that: spring cloud data flow.  
This allows you to read a stream of data change events. Runs when an event is received. Stream processing is different in the sense that it relies on additional setup to not lose a message, whereas batch processing always can go back to the original source.

### Installation
Add these dependencies to the maven `pom.xml`.

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-batch</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.batch</groupId>
        <artifactId>spring-batch-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

## Basics
- JobRepository: persists meta-data about batch job in either in-memory or a database.
- JobExporer: find and retrieve meta-data from the job repository.
- JobLauncher: runs a job with a given set of parameters.


Jobs have instances, executions, contexts, and parameters. A job has one or more steps, that defines a job's phase, and may be simple or complex.

Additionally, a step has an item reader, item processor, and an item writer. The reader and processor are single item-based and the writer is "chunk" based to optimize inserts/writes.

