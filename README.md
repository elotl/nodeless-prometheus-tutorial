# Monitoring Nodeless Kubernetes with Prometheus, Grafana

### Step 1: Create 1-worker Nodeless Kubernetes cluster

[Follow instructions in this repo](https://github.com/elotl/kubeadm-aws) to create a {1 master, 1 worker} Nodeless Kubernetes cluster.

Log on to Kubernetes master, verify cluster is up.

```
$ ssh -i "myechuri-key2.pem" ubuntu@100.26.23.126

$ kubectl cluster-info
Kubernetes master is running at https://10.0.23.147:6443
KubeDNS is running at https://10.0.23.147:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

$ kubectl get nodes -o wide
NAME                          STATUS   ROLES    AGE   VERSION   INTERNAL-IP   EXTERNAL-IP     OS-IMAGE             KERNEL-VERSION   CONTAINER-RUNTIME
ip-10-0-20-188.ec2.internal   Ready    worker   24d   v1.16.1   10.0.20.188   54.227.45.93    Ubuntu 16.04.6 LTS   4.4.0-1092-aws   containerd://1.2.6
ip-10-0-23-147.ec2.internal   Ready    master   24d   v1.16.1   10.0.23.147   100.26.23.126   Ubuntu 16.04.6 LTS   4.4.0-1092-aws   docker://18.9.7
```

### Step 2: Deploy {Prometheus, kube-state-metrics, Grafana} stack

```
$ wget https://raw.githubusercontent.com/elotl/nodeless-prometheus-tutorial/master/prometheus-grafana-metrics.yaml
```

Create a 250GiB EBS volume in the same Availability Zone as your kubernetes cluster.
![alt text](https://github.com/elotl/nodeless-prometheus-tutorial/blob/master/prometheus-ebs-volume.png "Prometheus EBS Volume")

Insert volume-id of EBS volume in the manifest.
```
$ sed -i 's/PROMETHEUS_VOLUME_ID/vol-0758e36178760bd8a/g' prometheus-grafana-metrics.yaml
```

Create {Prometheus, kube-state-metrics, Grafana} stack.
```
$ kubectl create -f prometheus-grafana-metrics.yaml
```

Wait until all components of `monitoring` namespace are up and running.
```
$ kubectl get all -n monitoring
NAME                                         READY   STATUS    RESTARTS   AGE
pod/grafana-7666cdc5cb-sd222                 1/1     Running   0          71s
pod/kube-state-metrics-d657c8bf4-v7482       2/2     Running   0          70s
pod/prometheus-deployment-5fc9b49dcb-ttbhw   1/1     Running   0          71s

NAME                         TYPE           CLUSTER-IP       EXTERNAL-IP                                                               PORT(S)                         AGE
service/grafana              LoadBalancer   10.111.33.212    afd9f0c0a7f2a41df8e6c0dfa5017564-306308846.us-east-1.elb.amazonaws.com    3000:30008/TCP                  71s
service/kube-state-metrics   LoadBalancer   10.102.149.242   ab76c6fb1664b4c2d89be300d11dce54-1258304794.us-east-1.elb.amazonaws.com   8080:31698/TCP,8081:32066/TCP   71s
service/prometheus-service   LoadBalancer   10.102.69.139    a7b1bca776a2b44108113042cf6c05a9-22531467.us-east-1.elb.amazonaws.com     8080:30904/TCP                  71s

NAME                                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/grafana                 1/1     1            1           71s
deployment.apps/kube-state-metrics      1/1     1            1           71s
deployment.apps/prometheus-deployment   1/1     1            1           71s

NAME                                               DESIRED   CURRENT   READY   AGE
replicaset.apps/grafana-7666cdc5cb                 1         1         1       71s
replicaset.apps/kube-state-metrics-d657c8bf4       1         1         1       71s
replicaset.apps/prometheus-deployment-5fc9b49dcb   1         1         1       71s
```

We now have the following stack.
*Note: Stateful application Prometheus is deployed on the worker node, and stateless applications {Grafana, kube-state-metrics} are deployed via Nodeless fashion. Once persistent state support is available in Nodeless k8s, Prometheus will be deployed in Nodeless way as well.*
![alt text](https://github.com/elotl/nodeless-prometheus-tutorial/blob/master/promstack.png "Prometheus Stack")


### Step 3: Create Grafana dashboard for Nodeless Kubernetes

Get external loadbalancer address for Grafana service and access Grafana through a web browser.

```
$ kubectl get svc grafana -n monitoring -ojsonpath='{.status.loadBalancer.ingress[0].hostname}'
afd9f0c0a7f2a41df8e6c0dfa5017564-306308846.us-east-1.elb.amazonaws.com
```

Create Datasource for Prometheus called `DS_Prometheus` with http url set to `http://prometheus-service:8080` since Grafana and Prometheus are running in the same k8s cluster.
![alt text](https://github.com/elotl/nodeless-prometheus-tutorial/blob/master/prometheus-datasource.png "Prometheus Datasource")

Import [Grafana dashboard 11124](https://grafana.com/grafana/dashboards/11124).

![alt text](https://github.com/elotl/nodeless-prometheus-tutorial/blob/master/grafana-dashboard-1.png "Grafana Dashboard")

There are 3 Compute Cells in our cluster running {Grafana, kube-state-metrics} and a system pod.

### Step 4: Deploy Nodeless workloads, monitor using Prometheus

Deploy Nginx deployment with 3 replicas.

```
$ wget https://raw.githubusercontent.com/elotl/nodeless-prometheus-tutorial/master/nginx.yaml

$ kubectl create -f nginx.yaml
```
### Teardown

Follow [teardown instructions from kubeadm repo](https://github.com/elotl/kubeadm-aws#teardown).
