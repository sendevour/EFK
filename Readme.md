# Setting up EFK stack on Kubenetes

## Setup Kubenets with 3 nodes 

kubectl get nodes

## Deploy Elasticsearch Fluent-bit and Kibana.
1. Create kube-logging namespace

        kubectl create -f kube-logging-ns.yaml
        kubectl get ns
2. Create elastic search stateful set with a storage and expose it as service
        
        kubectl create -f elasticsearch_sts_svc.yaml
        kubectl get svc -n kube-logging
    Pod will come up one by one after init containers execute and fix, fix-permissions, increase-vm-max-map, increase-fd-ulimit commands 
       
        kubectl get sts,pods -n kube-logging
3. Create Kibana Deployment and expose it as Load balancer.

        kubectl create -f kibana.yaml
    Ensure kibana pod is up and kibana service is assigned with external IP
        
        kubectl get svc,pods -n kube-logging

4. Create config map for fluent-bit.

        kubectl create -f fluentbit-cm.yaml
        kubectl get cm -n kube-logging
    Refer Fluent-bit official documentation for Input, output, Filter, Service and Parser configurations.
5. Create fluent-bit daemonset
        
        kubectl create -f fluent-bit.yaml
    Check for Service account, Cluster role and Clusterrolebining and daemonset creation

        kubectl get sa -n kube-logging
        kubectl get clusterrole -n kube-logging | grep fluent
        kubectl get clusterrolebinding -n kube-logging | grep fluent
        kubectl get ds -n kube-logging 
With ths EFK stack deployment is complete. Now deploy some application pods to to ensure indexing and log collection.

6. Deploying a sample counter pod

        kubectl create -f counter.yaml
    Single counter pod should be running

        kubectl get pods
    Check for logs from counter pod

        kubectl logs counter -f
    Find the node in which counter pod is running
        
        kubectl get pods -o wide
    Ensure one fluent-bit pod is running in every node
        
        kubectl get pods -n kube-logging -o wide
    Check the logs for the fluentd pod

        kubectl logs fluent-bit-4nt5d -n kube-logging -f
7. Access External IP of Kibana in browser and check the index and logs. 

        kubectl get svc -n kube-logging

## References:

[1] Fluent-bit official documentation - https://docs.fluentbit.io/manual/installation/kubernetes

[2] Fluent-bit, Elasticsearch integration - https://docs.fluentbit.io/manual/pipeline/outputs/elasticsearch

[3] https://www.digitalocean.com/community/tutorials/how-to-set-up-an-elasticsearch-fluentd-and-kibana-efk-logging-stack-on-kubernetes

[4] Storage class - https://kubernetes.io/docs/tasks/administer-cluster/change-default-storage-class/

[5] Metrics collection - https://docs.fluentbit.io/manual/administration/monitoring#metrics-in-prometheus-format

## Troubleshooting:

[1] https://stackoverflow.com/questions/71902938/fluent-bit-giving-400-with-elastic-search-contains-an-unknown-parameter-type

[2] Disabling Security in non-prod regions with env setting (Not recommended in production)

          - name: xpack.security.enabled
            value: "false"


