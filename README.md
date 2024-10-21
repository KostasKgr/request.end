# request.end

Advanced telemetry from any application with minimal effort!

The Request End Log Spec is a simple approach to obtain advanced telemetry from any application using just a single log line.

Get 90% of the benefits with 10% of the effort! Particularly useful for legacy applications that do not have open instrumentation libraries, providing insights without other telemetry infrastructure.

## Example Log Entry

```
{
    "timestamp": "2024-10-19T12:59:47.191+00:00",
    "traceId": "4cde9fb5035c0086ac3b7d53dd0c4551",
    "spanId": "94731cc1c869a0bc",

    "type": "request.end",

    "severityText": "INFO3",
    "severityNumber": "11",

    "attributes": {
        "code.duration": 0.1631,

        "myDownstreamApi.calls.createUser": 1,
        "myDownstreamApi.calls.getUser": 2,
        "myDownstreamApi.duration.createUser": 0.0012,
        "myDownstreamApi.duration.getUser": 0.0033,
        "myDownstreamApi.total.calls": 3,
        "myDownstreamApi.total.duration": 0.0045,

        "db.calls.queries": 35,
        "db.duration.queries": 0.015,
        "db.calls": 35,
        "db.duration": 0.015,

        "myframework.route": "register",
        
        "http.method": "POST",
        "http.request.header.referrer": "https://mysite.com/",
        "http.server.request.duration": 0.182603120803833,
        "http.host": "mysite.com",

        "url.path": "/register",
        "url.query": "campaign=mysiteisgreat",

        "client.address": "153.100.100.100",

        "flag.db_logging": true
    },
    "resource": {
        "service.name": "myapp"
    }
}
```

## Key Features

- Get advanced telemetry for legacy apps with a single log line.
- Monitor which parts of your application consume the most time and drive performance improvements.
- Monitor suspected improvement areas.
- Troubleshoot calls to APIs.
- Compare your traffic today to your traffic yesterday.
- Easy composition.
- Easy transition to sending spans for each type/subtype and for the request as a whole.
- Enable incremental adoption of telemetry.

## Request End Spec Conventions

- `code.duration` is a derived field. It is calculated as the `http.server.request.duration` minus the `total.duration` of each other type (e.g., database, downstream API).
  - Types that end with `_detailed` do not factor in code duration calculations.
  - Use `_detailed` for processes that happen in parallel, else you will get incorrect calculations for the `code.duration` field.
- Avoid dynamic fields with high cardinality to prevent issues with telemetry systems (e.g., ElasticSearch has a 1k max fields soft-limit).
- All durations are recorded in seconds.
- Use OpenTelemetry semantic convention names for attributes ([Semantic Conventions for HTTP Metrics](https://opentelemetry.io/docs/specs/semconv/http/http-metrics/)).
- The convention for calls and duration is as follows:

```
        "<type>.calls.<subtype>": "value",
        "<type>.duration.<subtype>": "value",
        "<type>.total.calls": "value",
        "<type>.total.duration": "value",
```

## Breakdown of Time Spent

The total time spent on processing a request is represented by the `http.server.request.duration` attribute. To derive the `code.duration`, subtract the `total.duration` of each other type (such as downstream APIs and database calls) from the `http.server.request.duration`.

For example:

- `http.server.request.duration`: 0.1826 seconds
- `myDownstreamApi.total.duration`: 0.0045 seconds
- `db.duration`: 0.015 seconds

The `code.duration` is calculated as:

```
code.duration = http.server.request.duration - myDownstreamApi.total.duration - db.duration
               = 0.1826 - 0.0045 - 0.015
               = 0.1631 seconds
```

This value represents the time spent in the code itself, excluding external calls.



### Beta Version

This specification is currently in beta. Features and conventions are subject to change. We welcome feedback and contributions to help us reach a stable version.



## FAQ

### Q: If multiple calls are made in a single process, how is the duration calculated?

If multiple calls are made in a single process, the duration is the sum of all calls. This approach makes it difficult to calculate percentiles for individual calls.

### Q: How can I handle detailed request information in tools like Kibana?

Tools like Kibana lack easy iteration over fields. It is preferable to have a single log line for each request that includes detailed request information, such as status, size, connection time, etc.

### Q: I don't have any other telemetry tools, can this help me?

If you don't have a telemetry stack in place you can still benefit from this approach. Having structured logs means that you can go in your application server and start querying them. For example you could use duckdb to directly load the json logs and start querying them with sql.

### Q: Do I need to implement the full example?

No you dont! Start simple and focus in the areas that will give you the most benefits.
- Begin by using structured logging (the json format) and the basic http and url fields.
- Start tracking the types/subtypes that are likely to need performance optimizations, or help troubleshoot production issues.
- Start tracking app/framework specific attributes to be able to filter your data. E.g. a route like `customer/%/history` so that you can drill down into these requests.
- Track more and more of 3rd party dependencies and introduce the `code.duration`.
- Add a traceId and include it in other logs of your application so that you can see all the logs of a single request together.
- Add a spanId and start moving towards open telemetry tracing!
