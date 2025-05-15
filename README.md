# Observability and Monitoring of Microservices

- How do we debug a problem in microservices?
- How do we monitor microservice performance?
- How do we monitor microservices health & metrics?
  
**Observability & Monitoring** solve the challenge of identifying and resolving above problems in microservices architectures before they cause outages.

**What is Observability?**

Observability is the ability to understand the internal state of a system by observing its outputs. In the context of microservices, it is achieved by collecting and analyzing data
from variety of sources such as **metrics, logs and traces**

3 pillars of observability are – Metrics, Logs, Traces

| Type    | Description |
|---------|-------------|
| **Metrics** | Metrics are quantitative measurements of the health of a system. They can be used to track things like CPU usage, memory usage, and response times. |
| **Logs**    | Logs are a record of events that occur in a system. They can be used to track things like errors, exceptions, and other unexpected events. |
| **Traces**  | Traces are a record of the path that a request takes through a system. They can be used to track the performance of a request and to identify bottlenecks. |


**What is Monitoring?**
Monitoring in microservices involves checking the telemetry data available in the
application and defining alerts for known failure states. This process collects and
analyses data from a system to identify and troubleshoot problems, as well as track the
individual microservices and the overall health of the microservices network

It allows you to –
- Identify and troubleshoot problems before they cause outages or other disruptions
- Track the health of your microservice
- Optimize microservice – You can identify areas where you can optimize your microservice to improve performance and reliability

<img width="427" alt="image" src="https://github.com/user-attachments/assets/40fa28b3-a716-4431-8c28-de084f6290e5" />

In other words, Monitoring is about **collecting data** and Observability is about **understanding the data**.

| Feature   | Monitoring                                  | Observability                                          |
|-----------|---------------------------------------------|--------------------------------------------------------|
| Purpose   | Identify and troubleshoot problems          | Understand the internal state of a system              |
| Data      | Metrics, traces, and logs                   | Metrics, traces, logs, and other data sources          |
| Goal      | Identify problems                           | Understand how a system works                          |
| Approach  | Reactive                                    | Proactive                                              |


# Logging

## Best approach is centralized logging

### How to perform this Log Aggregation?

#### Managing Logs with Grafana, Loki & Promtail

##### What is Grafana?

- Grafana is an open-source data visualization and monitoring tool that allows you to analyze and visualize metrics from various sources like databases, logs, and cloud services
- It is commonly used with time-series databases like Prometheus, InfluxDB, and Elasticsearch to monitor applications, infrastructure, and services in real-time
- It can be easily installed using Docker or Docker Compose

<img width="683" alt="image" src="https://github.com/user-attachments/assets/8a85eab8-9e5d-4ef7-b7ee-b6a51d725a64" />

##### What is Loki?

- Grafana Loki is a horizontally scalable, highly available and cost-effective log aggregation system.

##### What is Promtail?

- Promtail is a lightweight log agent that ships logs from your containers to Loki.

```
Grafana provides visualization of the log lines captured within Loki.
Together Grafana, Loki, Promtail provides a powerful logging solution.
```

<img width="666" alt="image" src="https://github.com/user-attachments/assets/2b0b80e1-1f2a-4e3a-a836-9aa75f1d2b28" />

##### Note:
```
From Grafana Loki version 3.0 onwards, Promtail, which is responsible for scraping log lines,
has been replaced with a new product called Alloy.
```

To collect logs and view your log data generally involves the following steps:

<img width="602" alt="image" src="https://github.com/user-attachments/assets/7dbae75c-8387-4d8d-8c6b-0a1a67ef71f5" />

##### Promtail reads logs from files or stdout and forwards to Loki

```
- cloud-native applications generate logs as events and send them to the standard
  output, without being concerned about the processing or storage of those logs.

- One advantage of treating logs as event streams and emitting them to stdout is that it
  decouples the application from the log processing infrastructure.

- The application can focus on its core functionality without being tied to a specific logging implementation
  or storage solution.

- The infrastructure, on the other hand, can handle the collection, aggregation, and storage of logs using
  appropriate tools and services.

- 15-Factor methodology recommends the same to treat logs as events streamed to the standard output and
  not concern with how they are processed or stored.
```

<img width="688" alt="image" src="https://github.com/user-attachments/assets/a4afd04c-c255-467c-b5eb-33ac585ab988" />

```
- If you are using Docker or Kubernetes, Promtail can be configured to scrape container stdout logs. How?
- To scrape container stdout logs with Promtail in Docker or Kubernetes, you don’t need to change the
  application code—just configure Promtail to watch the right files.
```

**Below are configurations for both Docker and Kubernetes:**

###### Promtail with Docker
- Docker logs are stored as JSON files by default at:

```
/var/lib/docker/containers/<container-id>/<container-id>-json.log
```

###### Promtail Config for Docker
- Here’s a promtail-config.yaml that scrapes Docker container logs:

```yaml
server:
http_listen_port: 9080
grpc_listen_port: 0

positions:
filename: /tmp/positions.yaml

clients:
- url: http://&lt;loki-host&gt;:3100/loki/api/v1/push

scrape_configs:
- job_name: docker
static_configs:
- targets:
- localhost
labels:
job: docker
__path__: /var/lib/docker/containers/*/*.log
```

###### Docker Run Example

- Mount the Docker container logs and run Promtail:

```bash
docker run -d \
--name=promtail \
-v /var/log:/var/log \
-v /var/lib/docker/containers:/var/lib/docker/containers:ro \
-v /etc/promtail-config.yaml:/etc/promtail/promtail-config.yaml \
grafana/promtail:latest \
-config.file=/etc/promtail/promtail-config.yaml
```










## Grafana in a Spring Boot Microservices Example
Let's assume we have a Spring Boot microservices architecture with the following services:

- Order Service - Handles customer orders.
- Payment Service - Manages payments.
- Inventory Service - Manages stock and inventory.

We want to monitor the health, performance, and metrics of these microservices using Grafana + Prometheus.

**1. Setting up Actuator for Metrics in Spring Boot**
<br>
- Spring Boot provides the Actuator library, which exposes built-in metrics and health information.
- Add Actuator and Micrometer Dependencies
  
In pom.xml, add:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>

<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

**2. Enable Metrics Endpoint**
<br>
Modify application.yml to expose Prometheus metrics:

```
management:
  endpoints:
    web:
      exposure:
        include: "prometheus"
  metrics:
    export:
      prometheus:
        enabled: true
  endpoint:
    health:
      show-details: always
```

Now, the Prometheus metrics will be available at: <br>
👉 http://localhost:8080/actuator/prometheus

**3. Configure Prometheus**
<br>
Prometheus scrapes metrics from Spring Boot services and makes them available for Grafana.
<br>
Create prometheus.yml configuration:

```
global:
  scrape_interval: 5s

scrape_configs:
  - job_name: 'spring-boot-microservice'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['host.docker.internal:8080']
```


**Run Prometheus using Docker:**

```
docker run -p 9090:9090 -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
```

Now, Prometheus is running at: <br>
👉 http://localhost:9090

<br>

**4. Connect Grafana to Prometheus**
<br>
Install and run Grafana using Docker:
```
docker run -d -p 3000:3000 grafana/grafana
```

- Open Grafana at http://localhost:3000 and log in (default: admin/admin).
- Go to Configuration → Data Sources → Add Data Source.
- Select Prometheus and set:
  URL: http://host.docker.internal:9090
- Save & Test.

**5. Create Grafana Dashboards**

# Metrics Visualization

## Key Metrics for Monitoring

Below are the key metrics to visualize and monitor system performance:

### 1. Response Time
   - **Metric:** `http_server_requests_seconds`
   - **Description:** Measures the time taken to handle HTTP requests.
   - **Visualization:** Line chart with time on X-axis and response time on Y-axis.

### 2. Memory Usage
   - **Metric:** `jvm_memory_used_bytes`
   - **Description:** Tracks JVM memory usage in bytes.
   - **Visualization:** Area chart showing memory usage over time.

### 3. CPU Load
   - **Metric:** `system_cpu_usage`
   - **Description:** Represents CPU load as a fraction (0 to 1).
   - **Visualization:** Gauge chart indicating real-time CPU load.

### 4. Request Count
   - **Metric:** `http_requests_total`
   - **Description:** Counts total HTTP requests received by the server.
   - **Visualization:** Bar chart showing request count per endpoint.

<br>

In the context of Prometheus and Grafana, **"scrape" means collecting or pulling metrics from a target system at a specified interval**.

<br>

**Scraping in Prometheus**
- Prometheus scrapes (fetches / pulls) metrics from configured endpoints (like a Spring Boot microservice).
- It is a pull model, as opposed to a push model
- It makes an HTTP request to the endpoint (e.g., /actuator/prometheus) and stores the collected data.
- The frequency of scraping is defined in scrape_interval in prometheus.yml.
- Metrics are stored in Prometheus and can be visualized using tools like Grafana.

<img width="379" alt="image" src="https://github.com/user-attachments/assets/1ab38e75-66ab-4051-bc0d-9ff32a24a57f" />

## Summary - Spring Boot Actuator, Micrometer, Prometheus, and Grafana – Monitoring Workflow

### **Spring Boot Actuator**
- Provides built-in endpoints to monitor and manage a Spring Boot application.
- Exposes application metrics, health status, and other operational information.
- Requires adding the `spring-boot-starter-actuator` dependency to the project.

### **Micrometer**
- A library that provides a **vendor-neutral API** for collecting application metrics.
- Integrates with various monitoring systems, including **Prometheus**.
- Allows developers to instrument their code with metrics without vendor lock-in.
- Requires adding the `micrometer-registry-prometheus` dependency.

### **Prometheus**
- An open-source **monitoring and alerting system**.
- **Scrapes metrics** from applications at regular intervals.
- Stores **time-series data** for analysis and visualization.
- Configured to pull metrics from the actuator's `/actuator/prometheus` endpoint.

### **Grafana**
- An open-source platform for **data visualization and monitoring**.
- Connects to various data sources, including **Prometheus**.
- Creates dashboards to display metrics and visualize application performance.
- Can import **pre-built dashboards** designed for Spring Boot applications.

### **Workflow**
- **Spring Boot Actuator** exposes metrics through its `/actuator/prometheus` endpoint.
- **Micrometer** collects metrics from the application and formats them for **Prometheus**.
- **Prometheus** scrapes the `/actuator/prometheus` endpoint to collect metrics data.
- **Grafana** queries **Prometheus** to display metrics on customizable dashboards.

<img width="607" alt="image" src="https://github.com/user-attachments/assets/1de336d6-ede6-426a-bf0d-7cad53c567b1" />

<br><br>
**Happy Monitoring,**
<br>
Pavan Uppuluri

