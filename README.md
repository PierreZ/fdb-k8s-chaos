# fdb-k8s-chaos

This is an experiment to run at the same time:

* [FoundationDB/fdb-kubernetes-operator](https://github.com/FoundationDB/fdb-kubernetes-operator)
* [pingcap/go-ycsb](https://github.com/pingcap/go-ycsb)
* [pingcap/chaos-mesh](https://github.com/pingcap/chaos-mesh)
* [PierreZ/fdb-prometheus-exporter](https://github.com/PierreZ/fdb-prometheus-exporter/)

## Requirements

* A large Kubernetes cluster. I am using [OVHcloud's Managed Kubernetes](https://www.ovhcloud.com/en/public-cloud/kubernetes/)
* FDB's operator
* [Prometheus deployed through Helm](https://github.com/helm/charts/tree/master/stable/prometheus-operator) for metrics

## How-to

```bash
# Setup the cluster with metrics
kubectl apply -f setup/

# Wait for the cluster to be healthy
kubectl get foundationdbcluster sample-cluster

# Wait for the metrics exporter to be healthy
kubectl get pods fdb-prometheus-exporter

# Start a workload
kubectl apply -f workload/load-fdb.yaml

# View workload
kubectl logs fdb-bench

# Get a shell to run 'status' in it
kubectl exec -it sample-cluster-log-1 fdbcli

# start running a chaos
kubectl apply -f chaos/kill-one-pod-randomly.yaml

# Access Prometheus
export POD_NAME=$(kubectl get pods --namespace default -l "app=prometheus,component=server" -o jsonpath="{.items[0].metadata.name}")
kubectl --namespace default port-forward $POD_NAME 9090
firefox localhost:9090 # one cool metric is 'fdb_workload_transactions_per_second'
```
