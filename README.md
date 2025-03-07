# Observability-and-Monitoring-of-Microservices

Grafana is an open-source data visualization and monitoring tool that allows you to analyze and visualize metrics from various sources like databases, logs, and cloud services. It is commonly used with time-series databases like Prometheus, InfluxDB, and Elasticsearch to monitor applications, infrastructure, and services in real-time.

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


