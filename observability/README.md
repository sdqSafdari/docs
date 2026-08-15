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

### OTeL Collector
> The OpenTelemetry Collector is a vendor-agnostic proxy that can receive, process, and export telemetry data.     
> It supports receiving telemetry data in multiple formats (for example, OTLP, Jaeger, Prometheus, as well as many commercial/proprietary tools) and sending data to one or more backends.     
> It also supports processing and filtering telemetry data before it gets exported.

below is a sample of otel-collector configuration yaml:    
```yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318

processors:
  batch:
    timeout: 1s
    send_batch_size: 1024

exporters:
  zipkin:
    endpoint: "http://zipkin:9411/api/v2/spans"
    tls:
      insecure: true
  elasticsearch:
    endpoints: ["http://elasticsearch:9200"]
  debug:
    verbosity: detailed

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [batch]
      exporters: [zipkin, debug]
    logs:
      receivers: [otlp]
      processors: [batch]
      exporters: [elasticsearch, debug]
```
and here is the decoded span from otel-collector debug log:    
```
Span #0
    Trace ID       : 7aa9df8e40b2d331265f4986903224df
    Parent ID      : 488f2fda93eb7517
    ID             : dbf1ac29cc32d87c
    Name           : HTTP POST
    Kind           : Client
    Start time     : 2026-08-15 09:57:17.686374946 +0000 UTC
    End time       : 2026-08-15 09:57:22.697885929 +0000 UTC
    Status code    : Unset
    Status message : 
    DroppedAttributesCount: 0
    DroppedEventsCount: 0
    DroppedLinksCount: 0
Attributes:
     -> http.method: Str(POST)
     -> http.status_code: Str(400)
     -> http.uri: Str(http://localhost:7979/create-m)
     -> spring.cloud.gateway.route.id: Str(first-wiremock)
     -> spring.cloud.gateway.route.uri: Str(http://localhost:7878/)
```

### OTLP protocol
OTLP is a general-purpose telemetry data delivery protocol designed in the scope of the OpenTelemetry project.    
OTLP is implemented over gRPC and HTTP transports and specifies Protocol Buffers schema that is used for the payloads.    

### Sampling
> if the large majority of your requests are successful and finish with acceptable latency and no errors,    
> you do not need 100% of your traces to meaningfully observe your applications and systems.     
> You just need the right sampling.

A trace or span is considered “sampled” or “not sampled”:    
- Sampled: A trace or span is processed and exported. Because it is chosen by the sampler as a representative of the population, it is considered “sampled”.
- Not sampled: A trace or span is not processed or exported. Because it is not chosen by the sampler, it is considered “not sampled”.

Sampling is one of the most effective ways to reduce the costs of observability without losing visibility.    
Kinds of Sampling:    
- Head Sampling 
  - Head sampling is a sampling technique used to make a sampling decision as early as possible. A decision to sample or drop a span or trace is not made by inspecting the trace as a whole.
- Tail Sampling
  - Tail Sampling gives you the option to sample your traces based on specific criteria derived from different parts of a trace, which isn’t an option with Head Sampling.
  - For example to Always sample traces that contain an error

Head sampling (what `management.tracing.sampling.probability` controls) decides at span creation time, before the request has even finished —     
the app has no idea yet whether this trace will end in an error, run slow, or hit an interesting code path.      
Tail sampling needs the complete trace — every span, from every service the request touched   
That requires a component that sees all spans centrally and can group them by trace ID,   
which only the Collector (not any single app instance) is positioned to do.     

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
