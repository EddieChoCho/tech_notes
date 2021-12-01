# Zipkin

## Architecture

### Overview

* Tracers live in your applications and record timing and metadata about operations that took place.
* The trace data collected is called a Span.
* Instrumentation is written to be safe in production and have little overhead.
  * They only propagate IDs in-band, to tell the receiver there’s a trace in progress.
  * Completed spans are reported to Zipkin out-of-band, similar to how applications report metrics asynchronously.

* The component in an instrumented app that sends data to Zipkin is called a Reporter.
  * Reporters send trace data via one of several transports to Zipkin collectors, which persist trace data to storage.

### Example flow

* Trace instrumentation is responsible for creating valid traces and rendering them properly. For example, a tracer
  ensures parity between the data it sends in-band (downstream) and out-of-band (async to Zipkin).

```
┌─────────────┐ ┌───────────────────────┐  ┌─────────────┐  ┌──────────────────┐
│ User Code   │ │ Trace Instrumentation │  │ Http Client │  │ Zipkin Collector │
└─────────────┘ └───────────────────────┘  └─────────────┘  └──────────────────┘
       │                 │                         │                 │
           ┌─────────┐
       │ ──┤GET /foo ├─▶ │ ────┐                   │                 │
           └─────────┘         │ record tags
       │                 │ ◀───┘                   │                 │
                           ────┐
       │                 │     │ add trace headers │                 │
                           ◀───┘
       │                 │ ────┐                   │                 │
                               │ record timestamp
       │                 │ ◀───┘                   │                 │
                             ┌─────────────────┐
       │                 │ ──┤GET /foo         ├─▶ │                 │
                             │X-B3-TraceId: aa │     ────┐
       │                 │   │X-B3-SpanId: 6b  │   │     │           │
                             └─────────────────┘         │ invoke
       │                 │                         │     │ request   │
                                                         │
       │                 │                         │     │           │
                                 ┌────────┐          ◀───┘
       │                 │ ◀─────┤200 OK  ├─────── │                 │
                           ────┐ └────────┘
       │                 │     │ record duration   │                 │
            ┌────────┐     ◀───┘
       │ ◀──┤200 OK  ├── │                         │                 │
            └────────┘       ┌────────────────────────────────┐
       │                 │ ──┤ asynchronously report span     ├────▶ │
                             │                                │
                             │{                               │
                             │  "traceId": "aa",              │
                             │  "id": "6b",                   │
                             │  "name": "get",                │
                             │  "timestamp": 1483945573944000,│
                             │  "duration": 386000,           │
                             │  "annotations": [              │
                             │--snip--                        │
                             └────────────────────────────────┘
```

### Transport

* Spans sent by the instrumented library must be transported from the services being traced to Zipkin collectors. There
  are three primary transports: HTTP, Kafka and Scribe.

### Components

* There are 4 components that make up Zipkin:
  * collector: Once the trace data arrives at the Zipkin collector daemon, it is validated, stored, and indexed for
    lookups by the Zipkin collector.
  * storage:
    * Zipkin was initially built to store data on Cassandra since Cassandra is scalable, has a flexible schema, and is
      heavily used within Twitter.
    * However, this component pluggable. In addition to Cassandra, it natively supports ElasticSearch and MySQL.
  * search(Zipkin Query Service)
    * The query daemon provides a simple JSON API for finding and retrieving traces. The primary consumer of this API is
      the Web UI.
  * web UI
    * The web UI provides a method for viewing traces based on service, time, and annotations.
    * Note: there is no built-in authentication in the UI!
