---
layout: post
title:  "Comprehensive Monitoring of Java Microservices with Prometheus and Grafana"
date:   2024-08-02 
categories: spring boot, monitoring, microservice
---

### TL:DR 🚀 

In the previous [post](/spring/boot,/microservices/2024/04/04/spring-k8.html), we built and deployed a Java microservice using Spring Boot, Docker, and Kubernetes. Now that your microservice is running in a cloud-native
environment, it's crucial to set up monitoring to ensure its health, performance, and reliability. In this post, I'll guide you through integrating Prometheus and Grafana
into your setup to monitor your Java microservices effectively.


## Introduction to Prometheus and Grafana

When managing microservices, it's essential to differentiate between monitoring and tracing as these terms are often confused. Monitoring focuses on tracking the overall health, performance,
and resource utilization of your application through metrics like CPU usage, memory consumption, and request rates.
Tracing, on the other hand, involves following individual requests or transactions across multiple services to diagnose issues in specific workflows.

In this post, we'll concentrate on monitoring your Java microservices using Prometheus and Grafana.
These tools will help you gain real-time insights into your application's performance and stability, enabling you to maintain a healthy and efficient microservices environment.

### Why Choose Prometheus and Grafana?

As we integrate Prometheus and Grafana into our Java microservices architecture, it’s important to understand why these tools are often preferred over others.

Prometheus integrates seamlessly with Kubernetes, automatically discovering and scraping metrics from services, pods, and nodes, making it a natural choice for cloud-native environments.
Its flexibility with custom metrics through Micrometer gives you deep insights into your application’s performance.

Grafana complements Prometheus with its powerful, customizable dashboards, enabling clear visualization of your metrics.
Its ability to integrate with multiple data sources means you can centralize monitoring across your entire infrastructure.

However, it’s essential to weigh these advantages against some considerations:

- **Complexity:** Setting up Prometheus and Grafana can be more complex compared to some commercial tools like Datadog or New Relic, which offer more out-of-the-box solutions.
- **Data Retention:** Prometheus's native data retention capabilities might require external tools like Thanos for long-term storage.

Despite these considerations, Prometheus and Grafana are often the tools of choice due to their scalability,
flexibility, and extensive community support. These characteristics make them particularly well-suited for dynamic environments where deep, customizable monitoring is critical.

## Prerequisites

- A Kubernetes cluster with the Java microservice deployed (as covered in the previous post).
- Basic knowledge of Kubernetes, Docker, and Spring Boot.
- kubectl and helm installed on your local machine.

## Step 1: Setting Up Prometheus

Before diving into the installation, let’s briefly discuss Helm, the tool we’ll use to simplify this process.

## What is Helm?

Helm is a package manager for Kubernetes that simplifies the deployment and management of applications within your cluster.
It allows you to define, install, and upgrade even the most complex Kubernetes applications through reusable "charts" that describe the structure of your application.

In this guide, we use Helm to install Prometheus and Grafana. Helm charts handle the entire setup process, including pulling the necessary Docker images,
setting up configuration files, and deploying the applications. This reduces the manual work required and ensures that everything is configured correctly out of the box.

By leveraging Helm, you can easily manage updates, rollbacks, and scaling of your applications, making it an invaluable tool for Kubernetes administrators and developers alike.

### Install Prometheus Using Helm

Now that we understand the role of Helm, let’s proceed with installing Prometheus.

1. Add the Prometheus Helm Chart Repository

   ```bash
   helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
   helm repo update
   ```

2. Install Prometheus
   ```bash
   helm install prometheus prometheus-community/prometheus --namespace monitoring --create-namespace
   ```
3. Verify the Installation:
   ```bash
   kubectl get pods -n monitoring
   ```

### Prometheus should now be running in your Kubernetes cluster under the monitoring namespace.

Expose Prometheus Outside the Cluster
By default, Prometheus is only accessible within the cluster. To access Prometheus externally, you can create a Kubernetes service with a NodePort or LoadBalancer type, depending on your environment.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: prometheus
  namespace: monitoring
spec:
  type: NodePort
  ports:
    - port: 9090
      targetPort: 9090
      nodePort: 30090
  selector:
    app: prometheus
```

Apply the service configuration

```bash
kubectl apply -f prometheus-service.yaml
```

Now, Prometheus should be accessible at `http://<node-ip>:30090`.

## Step 2: Instrumenting the Java Application with Micrometer

Micrometer is a metrics collection and instrumentation library that integrates seamlessly with Spring Boot and supports exporting metrics to various monitoring systems, including Prometheus.

### Add Micrometer Prometheus Dependency

Update your pom.xml to include the Micrometer Prometheus dependency:

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

### Configure Micrometer in Your Application

In your application.properties, enable Prometheus as the metrics export target:

```properties
management.endpoints.web.exposure.include=*
management.metrics.export.prometheus.enabled=true
management.endpoint.prometheus.enabled=true
```

This configuration exposes the metrics endpoint at `/actuator/prometheus`, which Prometheus will scrape.

## Step 3: Setting Up Grafana

### Install Grafana Using Helm

We'll use Helm to install Grafana in the Kubernetes cluster.

1. Add the Grafana Helm Chart Repository
   ```bash
   helm repo add grafana https://grafana.github.io/helm-charts
   helm repo update
   ```
2. Install Grafana

   ```bash
   helm install grafana grafana/grafana --namespace monitoring
   ```

3. Verify the Installation

   ```bash
   kubectl get pods -n monitoring
   ```

### Expose Grafana Outside the Cluster

Like Prometheus, Grafana can be exposed using a Kubernetes service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: grafana
  namespace: monitoring
spec:
  type: NodePort
  ports:
    - port: 3000
      targetPort: 3000
      nodePort: 30001
  selector:
    app.kubernetes.io/name: grafana
```

Apply the service configuration

```bash
kubectl apply -f grafana-service.yaml
```

Grafana should now be accessible at `http://<node-ip>:30001`

### Access Grafana Dashboard

- Open your browser and navigate to `http://<node-ip>:30001`.
- Log in using the default credentials **(admin/admin)**.
- Upon the first login, you will be prompted to change the password.

## Step 4: Integrating Prometheus with Grafana

### Add Prometheus as a Data Source

1. In the Grafana dashboard, go to `Configuration > Data Sources`.
2. Click `Add data source` and select `Prometheus`.
3. Set the URL to `http://prometheus.monitoring.svc.cluster.local:9090` if you're accessing it within the cluster, or use the external URL if you're accessing Prometheus outside the cluster.
4. Click `Save & Test` to ensure Grafana can connect to Prometheus.

### Create a Dashboard

1. Go to `Create > Dashboard`.
2. Click `Add new panel`.
3. In the `Metrics` section, enter a Prometheus query, such as `http_server_requests_seconds_count` to monitor the number of HTTP requests.
4. Customize the visualization and save the panel.

### Import Pre-built Dashboards

Grafana has a library of pre-built dashboards you can import:

1. Go to `Create > Import`.
2. Enter the dashboard ID from the Grafana Dashboard library, such as **1860** for a general JVM dashboard.
3. Select the Prometheus data source and click `Import`.

## Step 5: Setting Up Alerts

### Define Alerting Rules in Prometheus

To trigger alerts based on specific conditions, define alerting rules in Prometheus. Create a `prometheus-alerts.yaml` file:

```yaml
groups:
  - name: example
    rules:
      - alert: HighCpuUsage
        expr: sum(rate(container_cpu_usage_seconds_total[1m])) by (pod) > 0.8
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: 'High CPU usage detected'
          description: 'CPU usage is above 80% for more than 2 minutes.'
```

Apply the alerting rules:

```bash
kubectl apply -f prometheus-alerts.yaml
```

### Configure Alerting in Grafana

You can also configure alerts in Grafana that trigger when a certain threshold is met on any of your panels.

1. Open the panel you want to monitor.
2. Click the bell icon to create an alert.
3. Set the conditions under which the alert should be triggered.
4. Configure notifications to be sent to email, Slack, or other supported channels.

## Conclusion

By integrating Prometheus and Grafana into your Java microservices architecture, you gain the ability to monitor your application's health, performance,
and reliability in real-time. This setup enables you to identify and troubleshoot issues quickly, optimize resource usage, and ensure your services are running smoothly.

While Prometheus and Grafana offer powerful monitoring capabilities, they also introduce some complexity to your infrastructure.
For teams looking for a quicker, more managed solution, tools like Datadog or New Relic can be appealing due to their ease of setup and comprehensive feature sets,
though they might come with higher costs or less flexibility for custom metrics.

Nevertheless, if your project demands deep insights and flexibility, Prometheus and Grafana are exceptional choices that provide scalability,
customizability, and extensive community support, making them particularly well-suited for dynamic, cloud-native environments.