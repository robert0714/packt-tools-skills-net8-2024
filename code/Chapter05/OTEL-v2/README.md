
The collector is configured in
[otelcol-config.yml](https://github.com/open-telemetry/opentelemetry-demo/blob/main/src/otel-collector/otelcol-config.yml),
alternative exporters can be configured here.Reference: https://opentelemetry.io/docs/demo/architecture/

```mermaid
graph TB
subgraph tdf[Telemetry Data Flow]
    subgraph subgraph_padding [ ]
        style subgraph_padding fill:none,stroke:none;
        %% padding to stop the titles clashing
        subgraph od[OpenTelemetry Demo]
        ms(Microservice)
        end

        ms -.->|"OTLP<br/>gRPC"| oc-grpc
        ms -.->|"OTLP<br/>HTTP POST"| oc-http

        subgraph oc[OTel Collector]
            style oc fill:#97aef3,color:black;
            oc-grpc[/"OTLP Receiver<br/>listening on<br/>grpc://localhost:4317"/]
            oc-http[/"OTLP Receiver<br/>listening on <br/>localhost:4318<br/>"/]
            oc-proc(Processors)
            oc-spanmetrics[/"Span Metrics Connector"/]
            oc-prom[/"OTLP HTTP Exporter"/]
            oc-otlp[/"OTLP Exporter"/]
            oc-opensearch[/"OpenSearch Exporter"/]

            oc-grpc --> oc-proc
            oc-http --> oc-proc

            oc-proc --> oc-prom
            oc-proc --> oc-otlp
            oc-proc --> oc-opensearch
            oc-proc --> oc-spanmetrics
            oc-spanmetrics --> oc-prom

        end

        oc-prom -->|"localhost:9090/api/v1/otlp"| pr-sc
        oc-otlp -->|gRPC| ja-col
        oc-opensearch -->|HTTP| os-http

        subgraph pr[Prometheus]
            style pr fill:#e75128,color:black;
            pr-sc[/"Prometheus OTLP Write Receiver"/]
            pr-tsdb[(Prometheus TSDB)]
            pr-http[/"Prometheus HTTP<br/>listening on<br/>localhost:9090"/]

            pr-sc --> pr-tsdb
            pr-tsdb --> pr-http
        end

        pr-b{{"Browser<br/>Prometheus UI"}}
        pr-http ---->|"localhost:9090/graph"| pr-b

        subgraph ja[Jaeger]
            style ja fill:#60d0e4,color:black;
            ja-col[/"Jaeger Collector<br/>listening on<br/>grpc://jaeger:4317"/]
            ja-db[(Jaeger DB)]
            ja-http[/"Jaeger HTTP<br/>listening on<br/>localhost:16686"/]

            ja-col --> ja-db
            ja-db --> ja-http
        end

        subgraph os[OpenSearch]
            style os fill:#005eb8,color:black;
            os-http[/"OpenSearch<br/>listening on<br/>localhost:9200"/]
            os-db[(OpenSearch Index)]

            os-http ---> os-db
        end

        subgraph gr[Grafana]
            style gr fill:#f8b91e,color:black;
            gr-srv["Grafana Server"]
            gr-http[/"Grafana HTTP<br/>listening on<br/>localhost:3000"/]

            gr-srv --> gr-http
        end

        pr-http --> |"localhost:9090/api"| gr-srv
        ja-http --> |"localhost:16686/api"| gr-srv
        os-http --> |"localhost:9200/api"| gr-srv

        ja-b{{"Browser<br/>Jaeger UI"}}
        ja-http ---->|"localhost:16686/search"| ja-b

        gr-b{{"Browser<br/>Grafana UI"}}
        gr-http -->|"localhost:3000/dashboard"| gr-b
    end
end
```

Find the **Protocol Buffer Definitions** in the `/pb/` directory.

```bash
docker compose up --force-recreate --remove-orphans --detach
```
* Northwind.WebApi: http://localhost:8080/
* Grafana: http://localhost:8080/grafana/
* Load Generator UI: http://localhost:8080/loadgen/
* Jaeger UI: http://localhost:8080/jaeger/ui/ 
> When Northwind.WebApi's environment variable: `ASPNETCORE_ENVIRONMENT=Development`, the swagger would appear !
> 