# Prometheus

- About Prometheus
  - Its a pull based monitor tool created for monitoring highly dynamic container environments.
  - To Montior complex infra and services which are interlinked to each other where 100 servers and services are performing tasks.
- Prometheus Architecture

  - Prometheus Server
    - Data Retreival Worker
    - Time Series DB
    - HTTP Server accept queries
  - Format - Human Readable Text Based
  - Metrics Entries - Type and Help attribute
  - Help Entry - Description of what the metrics is
  - Type Entry - There are 3 type of metrics entry
    1. Counter - How Many Times X Happend
    2. Gauge - What is the current value of X now ?
    3. Histogram - How long or How big ?

- How Does Prometheus Collect Metrics Data from target ?
  Example - Monitor a linux server ?

  - Download a Node Exporter (Exporter available as docker images)
  - Untar and Execute the Exporter
  - Converts metrics of the server
  - Expose /metrics endpoint
  - Configure Prometheus to scrape this endpoint

  Example - Monitor Own Application - like How many requestes, exception and server resources are used ?

  - Using Client Libraries written in Programming Lang, we can expose /metrics endpoint

- Configure Prometheus
  To Configure Prometheus we need to create a prometheus.yaml file which has the data like

  - global => How often Prometheus will scrape its target
  - rules_files => Rule for aggregating metric values or creating Alert when condition met.
  - scrape_config => What resources to monitor
    - here you can also define your own jobs.
    - default value for each jobs will be `metric_path: "/metrics", scheme: "http"`

- Alert Manager is responsible to fire Alert to diff ways like email, slack, etc. when alert rule condition met through configured channel.
- Time Series DB is responsible to store data in custom time series format. so later you can query to this data using promql query lang.
- PromQL query Lang is responsible to query on the data stored in tsd.
  example -
  http_requests_total{status!~"4.."} - Query all HTTP status codes except 4xx ones.
  rate(http_requests_total[5m])[30m:] - Return the 5 min rate of the http_requests_total metric for the past 30 mins.

- Prometheus Advantages

  - Reliable
  - Stand Alone and Self Containing
  - Works, even if other parts of infra broken
  - no extensive set-up needed
  - less complex

- Prometheus Disadvantages

  - Difficult to scale
  - limits monitoring

- Prometheus Federation - It allows a prometheus server to scrape data from other prometheus servers

Monitoring Workflow For Kubernetes and Microservices Application
=================================================================
1. Create a K8s Cluster with EKS
2. Deploy Microservices App
3. Deploy Prometheus Monitoring Stack
   - Understand the Created Component like
     - Alert Manager
     - Time Series DB
     - Service Monitor
     - Grafana
     - Prometheus Rule
4. Monitor Cluster Nodes like usage of resources
5. Monitor K8s Component like service, workloads, etc.
6. Monitor Third-Party Application like redis.
   - Deploy Redis Exporter to Understand the redis data
7. Monitor Own Application
8. Visualize the data using Grafana
9. Configure Alert in Prometheus to send Alert.

- Here we will monitor on diff levels like -

1. Infra Level - CPU, RAM, Network etc
2. Platform Level - K8s Component
3. Application Level - Own App or Third Party app

- After pulling these data we will visualy analyze the application using Prometheus but in more advance use Grafana.

Monitoring Kubernetes Cluster and Microservices Application
=============================================================
- We need to deploy prometheus different parts with k8s cluter
  1. Using Manual Yaml File Creation - Not Efficient
  2. Using "Prometheus Operator" - Efficient
     - Manager of all Prometheus Components
     - It Manages the combination of all components as one unit.
  3. Using Helm Chart to Deploy Operator - Most Efficient
     - Helm will do initial setup
     - Operator manages the prometheus setup

1. Create EKS Cluster on AWS
   ```
       Run this command - eksctl create cluster --name <cluster-name> --region <region-name>
           - This command will take default aws creds and defalut region to deploy eks cluster with 2 worker nodes.
           - Run this command to check cluster is up with worker node - kubectl get node
       Run this command - eksctl delete cluster --name <cluster-name> --region <region-name>
           - To delete the cluster
   ```
2. Deploy Microservices Application on EKS
   ```
       Using Helm Chart we will deploy the microservice application
           - Ref - Prometheus-Learning/microservices-helm-charts
           - Inside microservices-helm-charts Run "./install.sh" file it will deploy the microservice application
   ```
3. Deploy Prometheus Stack using Helm
   ```
       helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
       helm repo update
       kubectl create namespace monitoring
       helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring
       helm ls
   ```
4. Access Prometheus UI
   ```
       Prometheus UI is an Internal service, let do port forwarding and access it to localhost
       - kubectl port-forward svc/monitoring-kube-prometheus-prometheus 9090:9090 -n monitoring &
   ```
5. Access Grafana UI
   ```
       Grafana UI is an Internal service, let do port forwarding and access it to localhost
       - kubectl port-forward svc/monitoring-grafana 8080:80 -n monitoring &
       - user: admin
       - pwd: prom-operator
   ```
6. Alerting in Prometheus is seprated into 2 parts
   - Define what we wanna be notified about (Alert Rules in Prometheus Server) like send notification, when cpu usage is above 50%
   - Send Notification (Configure Alert Manager)
7. Create a alert rule yaml file and apply it -
   ```
    - Ref - alert-rules.yaml
    - After Creating rule apply the file = kubectl apply -f <alert-rule file>
   ```
8. Now, You can check your manual creation rule in prometheus ui
   ```
    - Goto Prometheus UI
    - Click on Alert
    - Search for main-rules
   ```
9. Test Our Alert Rules
   ```
     Using CPUStress docker image we will run the container inside the k8 pod for testing our rule.
     - `kubectl run cpu-test --image=containerstack/cpustress -- --cpu 4 --timeout 30s --metrics-brief`
     You can check the cpu load over grafana under dashboard 'kubernetes-cluster'
     Also, Can check on prometheus alert.
   ```
10. Create Alert Manger yaml file to send alert Notification at firing state of alert rules
    ```
      To Check Prometheus Alert UI -
        - kubectl port-forward svc/monitoring-kube-prometheus-alertmanager 9093:9093 -n monitoring &
      Create an alert-manager.yaml file
        - Ref - alert-manager.yaml
      Apply the file
        - kubectl apply -f alert-manager.yaml
    ```
11. Now, You can check your manual alert manager in prometheus alert manager ui
    ```
      Goto Alert Manger UI
      Click On Status
      Check for your changes there
    ```
12. Test Our own alert manager
    ```
      Load the CPUStress by using CPUStress Image, and when HostHighCpuLoad alert rule is in firing state, you will get email.
    ```
13. Monitoring Thrid Party Application (Redis)
    ```
      For monitor third party application we have 'exporters'
      - Exporters
        - Exporter gets metrics data from the service
        - It translatest hese service specific metrics to prometheus understandable metrics
        - Exporter exposes these translated metrics under /metrics endpoint
      We need to tell prometheus about this new exporter.
      For that 'ServiceMonitor (custom k8s resources)' needs to be deployed.
        - ServiceMonitor is the set of targets to be monitored by prometheus.
    ```
14. Deploy Redis Exporter
    ```
      Using Helm Chart to deploy
        - helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
        - helm repo update
        - helm install redis-exporter prometheus-community/prometheus-redis-exporter -f redis-values.yaml

      To Check the redis exporter target
        - Goto Prometheus UI
        - Click on Status => Target
        - Search for redis-exporter
    ```
15. Get The Redis Exporter URL
    ```
      - export POD_NAME=$(kubectl get pods --namespace default -l "app=prometheus-redis-exporter,release=redis-exporter" -o jsonpath="{.items[0].metadata.name}")
      - echo "Visit http://127.0.0.1:8080 to use your application"
      - kubectl port-forward $POD_NAME 8080:
    ```
16. Create Alert Rules for Redis
    ```
      - Ref - redis-rules.yaml
      - After Creating rule apply the file = kubectl apply -f <alert-rule file>
    ```
17. Now, You can check your redis rule in prometheus ui
    ```
      - Goto Prometheus UI
      - Click on Alert
      - Search for redis-rules
    ```
18. Import Redis Dashboard in Grafana UI
    ```
      Search Predefined Dashboard on grafana labs(https://grafana.com/grafana/dashboards)
        - Predefined Dashboard should have to use the same exporter.
        - For Redis Predefined Dashboard - https://grafana.com/grafana/dashboards/763-redis-dashboard-for-prometheus-redis-exporter-1-x/
    ```
Monitoring Own Application
==============================
  - There is no exporter available for our own application so, we have to define the metrics own.
  - Here, we need to choose prometheus 'client libraries' that matches the lang in which your application is written.
  - Prometheus Client Libraries abstract interface to expose your metrics.
  - Libraries implement the prometheus metric types 
    - Counter
    - Gauge
    - histogram
    - Summary

### Build Own Application With Prometheus Client Library
  1. Ref - NodeJs Monitoring Application - `https://github.com/nirdeshkumar02/NodeJs-Monitoring-Application`
  2. Build Docker Image for above application - `docker build -t nirdeshkumar02/my-app:node-monitor-app`
  3. Push this application image to Private Repo - `docker push nirdeshkumar02/my-app:node-monitor-app` 

### Steps to Monitor Own NodeJS Application
  1. Create kubernetes docker login secret - 
      `kubectl create secret docker-registry docker-login --docker-server=https://index.docker.io/v1/ --docker-username=<> --docker-password=<>`  
  2. Create Kubernetes Configuration file - Ref Prometheus-Learning/k8s-config.yaml
  3. Apply the configuration file - `kubectl apply -f k8s-config.yaml` 
  4. Create ServiceMonitor metrics-configuration file - Ref Prometheus-Learning/own-app-service-monitor.yaml
  5. Apply the service monitor configuration file - `kubectl apply -f own-app-service-monitor.yaml` 

- Ref - Predefined Prometheus Alert Rules - https://awesome-prometheus-alerts.grep.to/
- Ref - Prometheus Client Library - https://prometheus.io/docs/instrumenting/clientlibs/
