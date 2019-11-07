# Monitoring Nodeless Kubernetes with Prometheus, Grafana

### Step 1: Create 1-worker Nodeless Kubernetes cluster

[Follow instructions in this repo](https://github.com/elotl/kubeadm-aws) to create a {1 master, 1 worker} Nodeless Kubernetes cluster.

### Step 2: Verify cluster is up

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

### Step 3: Create {Prometheus, kube-state-metrics, Grafana} stack


Create a 250GiB EBS volume in the same Availability Zone as your kubernetes cluster.

Inline-style:
![alt text](https://github.com/elotl/nodeless-prometheus-tutorial/blob/master/prometheus-ebs-volume.png "Prometheus EBS Volume")

### Teardown

Follow [teardown instructions from kubeadm repo](https://github.com/elotl/kubeadm-aws#teardown).
