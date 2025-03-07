# Grafana Loki and Promtail – Log Monitoring & Aggregation
**Grafana Loki** is a log aggregation system designed for storing and querying logs efficiently. 
Unlike traditional log systems like ELK (Elasticsearch, Logstash, Kibana), Loki doesn’t index log content 
but instead indexes labels (metadata), making it lightweight and cost-effective.

**Promtail** is an agent that collects logs from applications and forwards them to Loki.

## How Grafana Loki and Promtail Work Together
- Promtail collects logs from files, applications, or system logs (e.g., /var/logs).
- Promtail attaches labels (metadata) to logs (e.g., service name, instance).
- Promtail sends the logs to Grafana Loki.
- Loki stores logs in a scalable and efficient manner.
- Grafana queries logs from Loki and visualizes them.

