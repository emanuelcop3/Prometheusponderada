# Monitoramento do coletor OpenTelemetry (OTEL)

## Métricas

O coletor pode expor métricas do Prometheus localmente na porta 8888 e no caminho `/metrics`. Para ambientes containerizados, pode ser desejável expor essa porta em uma interface pública em vez de apenas localmente.

```yaml
service:
  telemetry:
    metrics:
      address: 127.0.0.1:8888
      level: detailed   
```

O coletor pode coletar suas próprias métricas por meio de sua própria linha de pipeline de métricas. Portanto, a configuração real pode parecer com isso:

```yaml
extensions:
  sigv4auth/aws:

receivers:
  prometheus:
    config:
      scrape_configs:
      - job_name: otel-collector-metrics
        scrape_interval: 10s
        static_configs:
          - targets: ['127.0.0.1:8888']

exporters:
  prometheusremotewrite/aws:
    endpoint: ${PROMETHEUS_ENDPOINT}
    auth:
      authenticator: sigv4auth/aws
    retry_on_failure:
      enabled: true
      initial_interval: 1s
      max_interval: 10s
      max_elapsed_time: 30s

service:
  pipelines:
    metrics:
      receivers: [prometheus]
      processors: []
      exporters: [awsprometheusremotewrite]
  telemetry:
    metrics:
      address: 127.0.0.1:8888
      level: detailed
```

## Painel do Grafana para métricas do coletor OpenTelemetry

[![Painel do coletor OpenTelemetry](dashboard/opentelemetry-collector-dashboard.png)](https://github.com/monitoringartist/opentelemetry-collector-monitoring/tree/main/dashboard)

## Alertas do Prometheus

Alertas do Prometheus recomendadas para métricas do coletor OpenTelemetry:

```yaml
groups:
  - name: opentelemetry-collector
    rules:
      - alert: processor-dropped-spans
        expr: sum(rate(otelcol_processor_dropped_spans{}[1m])) > 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: Algumas spans foram descartadas pelo processador
          description: Talvez o coletor tenha recebido spans não padrão ou tenha atingido alguns limites
      - alert: processor-dropped-metrics
        expr: sum(rate(otelcol_processor_dropped_metric_points{}[1m])) > 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: Alguns pontos de métricas foram descartados pelo processador
          description: Talvez o coletor tenha recebido pontos de métricas não padrão ou tenha atingido alguns limites
      - alert: receiver-refused-spans
        expr: sum(rate(otelcol_receiver_refused_spans{}[1m])) > 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: Algumas spans foram recusadas pelo receptor
          description: Talvez o coletor tenha recebido spans não padrão ou tenha atingido alguns limites
      - alert: receiver-refused-metrics
        expr: sum(rate(otelcol_receiver_refused_metric_points{}[1m])) > 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: Alguns pontos de métricas foram recusados pelo receptor
          description: Talvez o coletor tenha recebido pontos de métricas não padrão ou tenha atingido alguns limites
      - alert: exporter-enqueued-spans
        expr: sum(rate(otelcol_exporter_enqueue_failed_spans{}[1m])) > 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: Algumas spans foram enfileiradas pelo exportador
          description: Talvez o destino utilizado tenha um problema ou o payload utilizado não esteja correto
      - alert: exporter-enqueued-metrics
        expr: sum(rate(otelcol_exporter_enqueue_failed_metric_points{}[1m])) > 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: Alguns pontos de métricas foram enfileirados pelo exportador
          description: Talvez o destino utilizado tenha um problema ou o payload utilizado não esteja correto
      - alert: exporter-failed-requests
        expr: sum(rate(otelcol_exporter_send_failed_requests{}[1m])) > 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: Algumas solicitações de exportador falharam
          description: Talvez o destino utilizado tenha um problema ou o payload utilizado não esteja correto
      - alert: high-cpu-usage
        expr: max(rate(otelcol_process_cpu_seconds{}[1m])*100) > 90
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: Alto uso máximo de CPU
          description: O coletor precisa aumentar a escala
```


## Conclusão
Este projeto proporcionou uma compreensão prática do monitoramento do coletor OpenTelemetry usando métricas do Prometheus. Aprendemos a configurar e monitorar o coletor OTEL, bem como a criar dashboards informativos no Grafana para visualizar as métricas coletadas. Este conhecimento será valioso para implementar monitoramento em outros projetos futuros.

## Documentação

- https://github.com/open-telemetry/opentelemetry-collector/blob/main/docs/monitoring.md
- https://github.com/open-telemetry/opentelemetry-collector/blob/main/docs/troubleshooting.md#metrics
