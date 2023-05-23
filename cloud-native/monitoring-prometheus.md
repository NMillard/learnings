# Prometheus
Prometheus is an open source technology for monitoring and alerting in cloud-native environments, including Kubernetes. Metrics are collected and stored as time-series data.

Prometheus uses a flexible language called PromQL for queying logs.

## High-level overview of how it works
Prometheus requires applications to have an exposed HTTP endpoint that prometheus can scrape data from. An "exporter" is used to provide prometheus with data. Typically, you'd use a client library that includes an exporter.