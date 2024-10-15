# Spring Logging

Spring is setup with Apache Commons Logging, and spring boot starters use logback as the default logging implementation.

Common logging configuration is done in the `application` settings file.

```yaml
logging:
    level:
        root: info # Default log level
    file:
        name: logs/my-logs.log # file name
    logback:
        rollingpolicy:
            max-file-size: 10KB
            file-name-pattern: ${LOG_FILE}.%d{yyyy-MM-dd}.%i.gz
```

## System.out

Generally, don't write logs to the console–always use a logger.