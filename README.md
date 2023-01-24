Prometheus
============
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
    - Download a Node Exporter  (Exporter available as docker images)
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
        - default value for each jobs will be ```metric_path: "/metrics", scheme: "http"```

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
 
Configure Monitoring For Microservices Application
===================================================
#### Project Workflow 
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

### Project Workflow Actions
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