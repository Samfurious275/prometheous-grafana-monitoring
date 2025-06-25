# prometheous-grafana-monitoring

 Monitoring Nginx using Prometheus & Grafana 
Overview:
This project demonstrates the implementation of a monitoring solution for Nginx using Prometheus and Grafana . The setup enables real-time collection, visualization, and alerting of performance metrics from an Nginx service running in a Kubernetes environment. 
The entire deployment was done using Helm charts , ensuring modularity, scalability, and reusability across environments. 
A custom Grafana dashboard was created to monitor key Nginx metrics such as active connections, requests per second, response time, and more. 
Key Objectives: 
    • Deploy Prometheus and Grafana on Kubernetes using Helm.
    • Enable Nginx metrics exposure via /metrics endpoint or /status endpoint.
    • Configure Prometheus to scrape metrics from Nginx endpoints.
    • Visualize metrics using a custom Grafana dashboard .
    • Ensure smooth deployment and configuration using Helm charts . 
Architecture & Components:  
1. Kubernetes Deployment 
All components were deployed on a local Kubernetes cluster (e.g., Minikube) using Helm charts. 

Install Kube Prometheus Stack To monitor K8s Cluster
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add stable https://charts.helm.sh/stable
helm repo update
kubectl create namespace monitoring
helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --set prometheus.prometheusSpec.maximumStartupDurationSeconds=60 \
  --set prometheus.service.type=NodePort \
  --set prometheus.service.nodePort=30000 \
  --set grafana.service.type=NodePort \
  --set grafana.service.nodePort=31000 \
  --set alertmanager.service.type=NodePort \
  --set alertmanager.service.nodePort=32000 \
  --set prometheus-node-exporter.service.type=NodePort \
  --set prometheus-node-exporter.service.nodePort=32001
kubectl get svc -n monitoring
kubectl get namespace

kubectl port-forward svc/kind-prometheus-kube-prome-prometheus -n monitoring 9090:9090 --address=0.0.0.0 &
kubectl port-forward svc/kind-prometheus-grafana -n monitoring 31000:80 --address=0.0.0.0 &

2. Prometheus Queries
sum (rate (container_cpu_usage_seconds_total{namespace="default"}[1m])) / sum (machine_cpu_cores) * 100

sum (container_memory_usage_bytes{namespace="default"}) by (pod)


sum(rate(container_network_receive_bytes_total{namespace="default"}[5m])) by (pod)
sum(rate(container_network_transmit_bytes_total{namespace="default"}[5m])) by (pod)
sum(kube_pod_container_status_ready) by (pod, namespace)

# Combine with pod phase info
+ on(pod, namespace) group_left(phase)
(
  # Get current phase (using max to deduplicate)
  max(kube_pod_status_phase) by (pod, namespace, phase)
  == 1
)

2. Core Components 
Nginx: 
    •  Metrics available at /metrics endpoint.

Prometheus: 
    • Installed via Helm chart.
    • Configured to scrape metrics from Nginx endpoints.
    • Exposed via NodePort for external access.

Grafana: 
    • Installed via Helm chart. 
    • Connected to Prometheus as a data source. 
    • Used to build visual dashboards.
    • Exposed via NodePort.
Implementation Details: 
1. Nginx Configuration 
    • A service was created to expose the /metrics endpoint inside the Kubernetes cluster. 


2. Prometheus Setup 
    • Prometheus was deployed using the Helm.
    • Prometheus successfully scraped metrics from Nginx pods. 

3. Grafana Dashboard 
    • Grafana was installed via Helm and configured with Prometheus as the default data source.
    • A custom dashboard was imported or built manually to visualize:
        ◦ Active connections
        ◦ Requests per second
        ◦ Traffic (in/out)

 

    • Prometheus Targets : Confirming that Nginx metrics are being scraped.
    • Grafana Dashboard : Showing live graphs and metrics.
    • Terminal Output : Showing services running (kubectl get svc) and logs if any. 

Key Achievements: 
    • Successfully deployed and configured Prometheus and Grafana using Helm.
    • Enabled Nginx metric exposure and integrated with Prometheus.
    • Set up real-time metric scraping pipeline : Nginx → Prometheus → Grafana.
    • Created a custom Grafana dashboard for meaningful insights.
    • Validated deployment through CLI commands and UI tools.
    • Built reusable Helm chart for future deployments. 
Tools & Technologies Used: 
    • Kubernetes (Minikube)
    • Helm (for packaging and deployment)
    • Docker 
    • Prometheus 
    • Grafana 
    • Nginx 
    • Linux Terminal (Ubuntu) 
Conclusion: 
This project successfully demonstrated the integration of Prometheus and Grafana for monitoring an Nginx service in a Kubernetes environment. Highlights include: 
    • Smooth deployment of all components using Helm charts.
    • Real-time metric collection and visualization.
    • Custom Grafana dashboard for actionable insights.
    • Reusable architecture for scalable monitoring solutions. 
