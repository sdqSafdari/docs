### Cloud-native Observability 
Observability refers to the ability to monitor, measure, and understand the state of a system or application    
by examining its outputs, logs, and performance metrics.        
In modern software systems and cloud computing, observability plays an increasingly crucial role in ensuring the     
`reliability(fault-tolerance patterns, all functionalities work correctly)`, `performance(response-time)`, `security(influx of requests)` of applications and infrastructure.     
Observability absorbs and extends classic monitoring systems and helps teams identify the root cause of issues.    

Why Observability Matters?
- to proactively address problems and anomalies before they impact end-users. 
- the rise of `platform engineering(query metrics)` as a discipline    
- the widespread adoption of microservices, and the growing reliance on distributed architectures.    

Benefits of observability?    
- Improved reliability 
- Efficient troubleshooting
- Optimized performance
- Data-Driven decision-making

In a nutshell, cloud-native observability is a practice of monitoring, analyzing, and troubleshooting modern,    
cloud-native applications built using microservices architecture and deployed in containers or serverless environments.    

### Observability in DevOps
As observability becomes increasingly important for ensuring the reliability and performance of cloud-native applications,     
there is a greater focus on observability in the DevOps process.    
This includes the integration of observability tools into the DevOps toolchain, such as:    
- Prometheus/Grafana
- Jaeger
- Kafka
- OpenTelemetry

### Metrics, Logs, Traces
Metrics, Logs, and Traces , plus events are types of `Telemetry data`, and are pillars of Observability.   
- Log: timestamped text record
- Trace: the breadcrumb trail a request leaves as it moves through a system
  - the journey of a user transaction flow across different services
  - helps with root cause analysis, and identifying bottlenecks
- Metric: numerical measurement of a system, software, or application captured at runtime.
  - aggregations over a period of time of numeric data.
  - quantity analysis of performance over time, and include things like CPU usage, request rate, error rate, response time, and memory utilization.
  - helps with anomaly detection to things like capacity planning and SLA compliance monitoring

### OpenTelemetry
The OpenTelemetry project, sometimes abbreviated as OTel, provides a vendor-neutral,    
open-source framework to collect, process, and export telemetry data.     
Backed by the Cloud Native Computing Foundation,     
it offers an API, an SDK, a standard wire protocol called OTLP for exporting data,     
and a pluggable architecture (including the OpenTelemetry Collector) for handling ingestion, processing, and export to backends.    
OTLP wire-protocol is used to transmit Log/Metric/Trace (signals),    
OTLP can be used with HTTP or gRPC

The open source `observability framework` that unifies data collection and standardizes telemetry data formats.    
To send your telemetry data. you'll need a transmission protocol.     
This could be HTTPS (or HTTP) for web apps,     
MQTT for IoT devices,     
or even specialized protocols such as OpenTelemetry (OTLP).    

To make a system observable, it must be **instrumented**. That is, the code must emit traces, metrics, or logs.      
The instrumented data must then be sent to an observability backend(storage).    

Using OpenTelemetry, you can instrument your code in two primary ways:    
- Code-based solutions via official APIs and SDKs for most languages    
- Zero-code solutions: e.g. opentelemetry-javaagent.jar   

Typically, zero-code instrumentation adds instrumentation for the libraries you’re using:   
> Zero-code instrumentation adds the OpenTelemetry API and SDK capabilities to your application typically as an agent or agent-like installation.      
> The specific mechanisms involved may differ by language, ranging from bytecode manipulation, monkey patching, or eBPF to inject calls to the OpenTelemetry API and SDK into your application.

### Distributed tracing
Distributed tracing lets you observe requests as they propagate through complex, distributed systems.
Distributed tracing components(signal):   
- Log
- Metric
- Trace

Logs aren’t enough for tracking code execution, as they usually lack contextual information, such as where they were called from.   
They become far more useful when they are included as part of a span, or when they are correlated with a trace and a span.    

**Span**:    
Represents a unit of work or operation. Spans track specific operations that a request makes,     
painting a picture of what happened during the time in which that operation was executed.    
A span contains name, time-related data, structured log messages, and other metadata (that is, Attributes) to provide information about the operation it tracks.    
What data, A span includes?    
- *start_time* and *end_time* denoting the beginning and end of the entire operation.    
- *trace_id* and *span_id* and *parent_id* denoting the context.
- *attributes* object carrying context info e.g. *user.id* key-value 

**trace(correlation)**:
A distributed trace, more commonly known as a trace,    
records the paths taken by requests (made by an application or end-user) as they propagate through multi-service architectures, like microservice and serverless applications.    
A trace is made of one or more spans. spans are building blocks of trace.     
Without tracing, finding the root cause of performance problems in a distributed system can be challenging.   
Many Observability backends visualize traces as waterfall diagrams that look like this:     
![waterfall trace](./waterfall-trace.svg)

**context propagation**:    
With context propagation, signals (traces, metrics, and logs) can be correlated with each other.     
Propagation is the mechanism that moves context between services and processes.    
Propagation is usually handled by *instrumentation libraries* and is transparent to the user.    
**Example**:    
The context (here: Trace ID and Span ID as “Parent ID”) is propagated using     
the `traceparent` header as it is defined in the W3C TraceContext specification.

```
traceparent: <version>-<trace-id>-<parent-id>-<trace-flags>
#e.g.
traceparent: 00-a0892f3577b34da6a3ce929d0e0e4736-f03067aa0ba902b7-01
```

### References
- [redhat observability](https://www.redhat.com/en/topics/devops/what-is-observability)
- [elasticsearch telemetry data](https://www.elastic.co/what-is/telemetry-data)
- [OTel doc](https://opentelemetry.io/docs/)
- [OTel basic concepts](https://opentelemetry.io/docs/concepts/observability-primer/)
- [Spring blog opentelemetry](https://spring.io/blog/2025/11/18/opentelemetry-with-spring-boot)
