# Observability and Monitoring of Microservices

- How do we debug a problem in microservices?
- How do we monitor microservice performance?
- How do we monitor microservices health & metrics?
  
**Observability & Monitoring** solve the challenge of identifying and resolving above problems in microservices architectures before they cause outages.

**What is Observability?**

Observability is the ability to understand the internal state of a system by observing its outputs. In the context of microservices, it is achieved by collecting and analyzing data
from variety of sources such as **metrics, logs and traces**

3 pillars of observability are – Metrics, Logs, Traces

<img width="610" alt="image" src="https://github.com/user-attachments/assets/569d8b78-716f-4e1e-b485-d449eb99e755" />

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

<img width="605" alt="image" src="https://github.com/user-attachments/assets/d1821747-28ba-4ac3-b8d2-d88c370418b4" />


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
- Prometheus scrapes (fetches) metrics from configured endpoints (like a Spring Boot microservice).
- It makes an HTTP request to the endpoint (e.g., /actuator/prometheus) and stores the collected data.
- The frequency of scraping is defined in scrape_interval in prometheus.yml.


