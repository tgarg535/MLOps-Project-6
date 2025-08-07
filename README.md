# MLOps-Project-6
Learning Prometheus And Grafana

# **Flask Application with Metrics**  
🚀 A complete guide to deploying a Flask application on Minikube, setting up Prometheus for metrics collection, and visualizing data on Grafana. This step-by-step tutorial is beginner-friendly and packed with practical instructions for a seamless hands-on experience.  

---

## **Project Overview**  
This repository demonstrates:  
1. How to deploy a Flask application locally and on Minikube.  
2. Setting up Prometheus to scrape metrics from the Flask app.  
3. Integrating Prometheus with Grafana for monitoring and visualization.  

---

## **Table of Contents**  
1. [Getting Started with Flask App](#getting-started-with-flask-app)  
2. [Deploying Flask App on Minikube](#deploying-flask-app-on-minikube)  
3. [Setting Up Prometheus](#setting-up-prometheus)  
4. [Configuring Prometheus to Scrape Flask Metrics](#configuring-prometheus-to-scrape-flask-metrics)  
5. [Installing Grafana](#installing-grafana)  
6. [Visualizing Metrics with Grafana Dashboards](#visualizing-metrics-with-grafana-dashboards)  

---

## **Getting Started with Flask App**  
1. Prepare the Flask app locally. Make sure it includes metrics endpoints for Prometheus to scrape.  

---

## **Deploying Flask App on Minikube**  
1. **Build Docker Image:**  
   ```bash  
   docker build -t flask-metrics-app:latest .  
   ```  
2. **Start Minikube:**  
   ```bash  
   minikube start  
   ```  
3. **Load Docker Image into Minikube Cluster:**  
   ```bash  
   minikube image load flask-metrics-app:latest  
   ```  
4. **Deploy Flask App:**  
   Create a `flask-app.yaml` Kubernetes manifest file and apply it:  
   ```bash  
   kubectl apply -f flask-app.yaml  
   ```  
5. **Access the Flask App:**  
   Get the service URL:  
   ```bash  
   minikube service flask-metrics-app --url  
   ```  

---

## **Setting Up Prometheus**  
1. **Install Helm:**  
   Install Helm using your package manager:  
   - Windows:  
     ```bash  
     winget install Helm.Helm  
     ```  
   - macOS:  
     ```bash  
     brew install helm  
     ```  
2. **Add Prometheus Helm Chart:**  
   ```bash  
   helm repo add prometheus-community https://prometheus-community.github.io/helm-charts  
   helm repo update  
   ```  
3. **Install Prometheus:**  
   ```bash  
   helm install prometheus prometheus-community/prometheus --namespace monitoring --create-namespace  
   ```  

---

## **Configuring Prometheus to Scrape Flask Metrics**  
1. **Retrieve Flask App Service IP:**  
   ```bash  
   kubectl get svc  
   ```  
   Note the ClusterIP and port for `flask-metrics-app`.  
2. **Edit Prometheus ConfigMap:**  
   Add your Flask app as a scrape target:  
   ```yaml  
   scrape_configs:  
     - job_name: 'flask-app'  
       static_configs:  
         - targets: ['<FLASK-APP-IP>:5000']  
   ```  
3. **Restart Prometheus:**  
   ```bash  
   kubectl rollout restart deployment prometheus-server -n monitoring  
   ```  
4. **Verify Metrics Collection:**  
   Access the Prometheus dashboard and search for Flask metrics:  
   ```bash  
   kubectl port-forward -n monitoring svc/prometheus-server 9090:80  
   ```  

---

## **Installing Grafana**  
1. **Add Grafana Helm Chart:**  
   ```bash  
   helm repo add grafana https://grafana.github.io/helm-charts  
   helm repo update  
   ```  
2. **Install Grafana:**  
   ```bash  
   helm install grafana grafana/grafana -n monitoring --create-namespace  
   ```  
3. **Access Grafana Locally:**  
   Port-forward Grafana to your local machine:  
   ```bash  
   kubectl port-forward svc/grafana -n monitoring 3000:80  
   ```  
   Log in with the default credentials:  
   - Username: `admin`  
   - Password: Obtain using:  
     ```bash  
     kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode  
     ```  

---

## **Visualizing Metrics with Grafana Dashboards**  
1. **Add Prometheus as a Data Source:**  
   - Navigate to: `Connections > Data Sources > Add Data Source > Prometheus`  
   - Use Prometheus URL:  
     ```
     http://prometheus-server.monitoring.svc.cluster.local:80  
     ```  
   - Click `Save & Test`.  
2. **Create a Dashboard:**  
   - Navigate to: `Dashboards > New Dashboard > Add New Panel`.  
   - Visualize metrics such as `total_api_requests_total`.  

---

## **Highlights**  
- **End-to-End Observability:** Full setup for monitoring a Flask app with Prometheus and Grafana.  
- **Production-Ready Notes:** Includes steps for Minikube and considerations for scaling to production.  
- **Beginner-Friendly Guide:** Clear instructions to help even first-time users.  

---

Feel free to use this as a foundation for your observability stack. Contributions are welcome! 🛠️  

---  
---
---

### **Observability in Microservices**
- **Why Observability Matters**:
  - Microservices are inherently distributed, making it difficult to track system behavior.
  - Observability provides insights into the state and health of the system through logs, metrics, and traces.
  - It helps in identifying bottlenecks, detecting failures, and ensuring reliability and performance.

---

### **Monitoring Basics**
- **Definition**: Monitoring is the process of collecting and visualizing data about a system’s health and performance over time.
- **Key Questions Monitoring Answers**:
  - Is the service running?
  - Is the service functioning as expected?
  - Is the service performing within acceptable thresholds?

---

### **Monitoring vs. Observability**
| **Monitoring**                           | **Observability**                        |
|------------------------------------------|------------------------------------------|
| Tracks predefined metrics.               | Offers deeper insights into unknown issues. |
| Answers "What is wrong?"                 | Answers "Why is it wrong?"               |
| Focused on system health.                | Focused on understanding internal state. |

---

### **Telemetry Data**
- Telemetry data includes:
  - **Metrics**: Quantitative data, e.g., CPU usage, response time.
  - **Logs**: Detailed event records.
  - **Traces**: Sequence of events showing request flow.

---

### **Methods of Metrics Collection**
1. **Push Method**:
   - Clients send data to a centralized system periodically.
   - Example: Using Prometheus **Pushgateway** for short-lived jobs.
   - Pros: Works well for dynamic, short-lived services.
   - Cons: Higher client responsibility for sending data.
   
2. **Scrape Method**:
   - Centralized system (e.g., Prometheus) pulls data from endpoints.
   - Example: Prometheus scrapes metrics from `/metrics` exposed by an exporter.
   - Pros: Decouples data collection from clients.
   - Cons: Services must expose data at specific endpoints.

---

### **Exporter and Pushgateway**
- **Exporter**:
  - A tool that exposes application or system metrics in a Prometheus-compatible format.
  - Example: Node Exporter collects system-level metrics like CPU and memory usage.

- **Pushgateway**:
  - Allows ephemeral jobs to push metrics directly to Prometheus.
  - Used when a service doesn't persist long enough for Prometheus to scrape.

---

### **Prometheus Data Model**
- Prometheus stores **time series data**.
- Each time series is identified by:
  - **Metric name**: Represents what is being measured (e.g., `http_requests_total`).
  - **Labels**: Key-value pairs for filtering and grouping (e.g., `status="200"`).
  - Example: `predict_api_hit{count="1", time_taken="600"}`.

---

### **PromQL Basics**
- PromQL is the query language for Prometheus.
- **Examples**:
  - `up`: Shows whether targets are up (1) or down (0).
  - `rate(http_requests_total[5m])`: Computes request rate over the last 5 minutes.
  - `sum by (status)(http_requests_total)`: Groups and sums requests by HTTP status.

---

### **Grafana Cloud**
- **Advantages**:
  - No need for setup or maintenance.
  - Automatically updated with the latest features.
  - Handles complex architectures out-of-the-box.
- **Disadvantages**:
  - Higher costs for scaling.
  - Dependency on the vendor.
  - Reduced control over security and data management.

---

### **On-Premise Grafana Setup**
- **Advantages**:
  - Complete control over data and security.
  - Tailored to specific organizational needs.
  - No recurring license costs for basic features.
- **Disadvantages**:
  - Requires infrastructure and maintenance.
  - Scalability can be challenging.
  - Manual updates and configurations.

---

### **Alerting in Grafana**
1. **How Alerts Work**:
   - Alerts are raised when specific conditions (rules) are met.
   - Rules are defined using queries (e.g., CPU usage > 80%).

2. **Components**:
   - **Alert Manager**: Evaluates rules and triggers alerts.
   - **Notification Policies**: Define how alerts are routed.
   - **Contact Points**: Email, Slack, PagerDuty, or other mediums for notifications.

3. **Setup Example**:
   - Define a query: `avg_over_time(cpu_usage[5m]) > 80`.
   - Configure a notification policy to send alerts to an email address.

---

Changes to be made on default.ini file to add your email into Grafana alert notification system:


[smtp]
enabled = true
host = smtp.gmail.com:587
user = your-email@gmail.com
password = your-generated-app-password
from_address = your-email@gmail.com
from_name = Grafana Alerts
skip_verify = true
