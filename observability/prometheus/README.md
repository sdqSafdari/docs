### Prometheus, Time Series Database(TSDB)
Prometheus is a metric oriented, multidimensional, pull-based, Time Series DataBase.       
It stores metrics(which might have multiple labels) in a storage efficient manner.   
> An open-source monitoring system with a dimensional data model, 
> flexible query language, 
> efficient time series database and modern alerting approach.

### Prometheus, is not an observability backend
### Prometheus is a monitoring system
Prometheus "scrapes" (pulls) numerical data (metrics) from your services.    
True observability requires "The Three Pillars" (Metrics, Logs, and Traces) to be linked together.    
If you see a spike in Prometheus,    
you can't "click" on that spike to see the specific logs or traces associated with it within Prometheus itself.   
You usually need Grafana to bridge that gap.

### OpenMetrics format
*OpenMetric* It specifies the de-facto standard for transmitting cloud-native metrics at scale,     
with support for both text representation and Protocol Buffers.    
```
float64 value;
EpochMillis timestamp;
metric_name{label_name="label_value", ...} value timestamp
prometheus_build_info{version="3.8.0", goversion="go1.25.4"} 1 1711050000
```

Here is the sample text-based metrics, exposed by Spring Actuator at `/actuator/prometheus`:   
```
# HELP http_server_requests_active_seconds  
# TYPE http_server_requests_active_seconds summary 
http_server_requests_active_seconds_count{application="gateway",exception="none",method="GET",outcom
e="SUCCESS",status="200",uri="UNKNOWN"} 1
http_server_requests_active_seconds_sum{application="gateway",exception="none",method="GET",outcome=
"SUCCESS",status="200",uri="UNKNOWN"} 0.012867914
# HELP http_server_requests_active_seconds_max  
# TYPE http_server_requests_active_seconds_max gauge
http_server_requests_active_seconds_max{application="gateway",exception="none",method="GET",outcome=
"SUCCESS",status="200",uri="UNKNOWN"} 0.012878955
```

**Label (Tag, Dimension)**:    
a Label is a key-value pair attached to a metric.    
Think of it as metadata that turns a single flat number into a multidimensional data point.   
##### Why call it a "Dimension"?
Imagine you are tracking `http_requests_total`.
- Without labels, you only know the total requests for the whole system.
- With labels like `{method="GET", status="200", service="api"}`, you can suddenly "slice" the data:
  *Slice 1: Show me only `GET` requests.
  *Slice 2: Show me only `404` errors across all methods.
  *Slice 3: Compare the performance of `service="api"` vs `service="payment"`.


**Series (Identity)**:   
A Series is the unique identity of a metric. It is defined by the Metric Name + all its Labels.    
In the TSDB, each unique combination is assigned a unique Series ID.   

**Sample**:   
timestamp + value

### Metric Types
The Prometheus server does not yet make use of the type information and    
flattens all types except native histograms into untyped time series of floating point values.   
- Counter: Monotonically Increasing counter
- Gauge: single numerical value that can arbitrarily go up and down.
- Histogram: bucketed counter
- Summary: like Histogram While it also provides a total count of observations and a sum of all observed values

### Counter: A Single Series
A Counter is a scalar value. It generates exactly one time series per unique combination of labels.
- Format: `metric_name{labels} value`
- Goal: To track a single, ever-increasing total.
- Analogy: A simple click-counter hand tool. Every time you click, the single number goes up.

Building dashboards and alerts of the rate of a counter per some interval of time     
disregards the longevity of the app, letting you see aberrant(not conforming to standard) behavior long after the application has started.    

### Histogram: bucketed counter
In Mathematics To construct a histogram, the first step is to "bucket" the range of values—    
divide the entire range of values into a series of intervals—    
and then count how many values fall into each interval.    
![Histogram sample](./Black_cherry_tree_histogram.svg)

Histograms exists in two fundamentally different versions: The more recent `native histograms` and the older `classic histograms`.    
**If you can, use native histograms and prefer them over both classic histograms and summaries.**    
A native histogram is exposed and ingested as composite samples, where each sample represents the count and sum of observations together with a dynamic set of buckets.   

**First of all, check the instrumentation library support for histograms and summaries.**
Summaries are usually supported by all libraries, Classic histogram support is also widespread, but native histogram support is still rare.    
Currently, the latter requires exposition via the protobuf format, limiting the support to protobuf-enabled libraries, like the Java and the Go library.   
support in a text-based format is underway as part of OpenMetrics v2.     
Things should be moving very soon, so definitely check what your library has to offer.   

> Histograms and summaries both record observations, typically request durations or response sizes.    
> In all variants (even quantile-less summaries), they track the number of observations and the sum of the observed values,    
> allowing you to calculate the average of the observed values.

> In the case of a summary or a classic histogram,    
> you have separate time series for the sum and count of observations,    
> marked by the magic suffixes _sum and _count, respectively.

For example, for observing request duration with three buckets  duration≤ 0.1s, ≤ 0.5s, ≤ 1s
You would have following time series with Classic Histograms
```
# Three cumulative bucket time series
http_requests_duration_seconds_bucket{le="0.1"} 42      # 42 requests ≤ 0.1s
http_requests_duration_seconds_bucket{le="0.5"} 95      # 95 requests ≤ 0.5s (includes the 42)
http_requests_duration_seconds_bucket{le="1.0"} 100     # 100 requests ≤ 1.0s

# The _count series (counts all observation)
http_requests_duration_seconds_count 100                # Total requests = 100

# The _sum series (sum of all request durations in seconds)
http_requests_duration_seconds_sum 37.5                 # Total duration of all requests
```

### Summary: A Composite Metric (Multiple Series)
A Summary is a complex object. It is designed to track two things at once:     
How many events happened AND the total magnitude of those events. Because of this, even in its simplest form,     
a Summary generates at least two linked time series with base-metric-name of *<basename>*:

1.  `<basename>_count`: Tracks how many times the event occurred (behaves like a Counter).
2.  `<basename>_sum`: Tracks the total sum of all observed values (e.g., total seconds spent).
3.  `<basename>{quantile="0.95"}` (Optional): If you enable quantiles (percentiles), it generates even more series (one for each quantile you define).

!WARNING: quantile time series have performance cost, because they are calculated at Instrumented App

### Cumulative Counter
metric types: summary, counter
Prometheus uses "Cumulative Counters" because they are resilient to network issues.   
- If Prometheus fails to scrape your app at 12:00:15 but succeeds at 12:00:30, it hasn't "lost" the data.
- The `_sum` at 12:00:30 will still include everything that happened during the missed 15 seconds.
- The `rate()` function will simply see a bigger jump between the two points and calculate the average correctly.


### References
- [Prometheus Metric Types](https://prometheus.io/docs/concepts/metric_types/)
- [Prometheus Histogram and Summaries](https://prometheus.io/docs/practices/histograms/)
- [open metric](https://openmetrics.io/)
- [WIKI Histogram](https://en.wikipedia.org/wiki/Histogram)
