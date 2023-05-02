# Racher_HA_mode
Racher_HA_mode


CREATE CLUSTER FROM RKE and RESTORE RKE ClUSTER

1. Create new  4 nodes (1-deployment node and 3-cluster node | All should be passwordless from deployment node)


2. 

Create a file in Deployment node 
Example: rancher-cluster.yaml (Change node addresses accordingly)
##############################
private_registries:
  - url: docker.registry.server
    user: admin
    password: ********
    is_default: true
nodes:
  - address: 192.168.3.**
    user: niccloud
    role: [controlplane,worker,etcd]
  - address: 192.168.3.**
    user: niccloud
    role: [controlplane,worker,etcd]
  - address: 192.168.3.**
    user: niccloud
    role: [controlplane,worker,etcd]
services:
  etcd:
    snapshot: true

***********************
3. run rke command  where above rancher-cluster.yaml should present  (to fresh install rancher)

rke up --config ./rancher-cluster.yaml

a. after complition of first command kubeconfig will present in below location

$(pwd)/kube_config_cluster.yml

so use it by default by export command

export KUBECONFIG=$(pwd)/kube_config_cluster.yml

************************************
4. Check Health Of Cluster 
kubectl get pods --all-namespaces
kubectl get nodes
***********************************
5. Save a copy of the following files in a secure location:

rancher-cluster.yml: The RKE cluster configuration file.
kube_config_cluster.yml: The Kubeconfig file for the cluster, this file contains credentials for full access to the cluster.
rancher-cluster.rkestate: The Kubernetes Cluster State file, this file contains credentials for full access to the cluster.

***************************************
6. INSTALL RANCHER
kubectl create namespace cattle-system
kubectl -n cattle-system create secret tls tls-rancher-ingress --cert=tls.crt --key=tls.key

(tls.crt will be full chain)

Now install with helm chart

helm repo add rancher-stable https://releases.rancher.com/server-charts/stable
helm fetch rancher-/rancher --version=v2.5.9
unzip it
cd rancher
helm install rancher rancher --version v2.5.9 --namespace cattle-system --set hostname=devrancher.cloud.gov.in --set replicas=3 --set ingress.tls.source=secret
*****************************************

7 Set up nginx as a HA-Proxy

docker run -d --restart=unless-stopped    -p 80:80 -p 443:443    -v /etc/nginx/nginx.conf:/etc/nginx/nginx.conf nginx:1.17.4-alpine

/etc/nginx/nginx.conf will be as below:

worker_processes 2;
worker_rlimit_nofile 40000;

events {
    worker_connections 8192;
}

stream {
    upstream rancher_servers_http {
        least_conn;
        server 192.168.3.77:80 max_fails=3 fail_timeout=5s;
        server 192.168.3.21:80 max_fails=3 fail_timeout=5s;
        server 192.168.3.252:80 max_fails=3 fail_timeout=5s;
    }
    server {
        listen 80;
        proxy_pass rancher_servers_http;
    }

    upstream rancher_servers_https {
        least_conn;
	server 192.168.3.77:443 max_fails=3 fail_timeout=5s;
	server 192.168.3.21:443 max_fails=3 fail_timeout=5s;
	server 192.168.3.252:443 max_fails=3 fail_timeout=5s;
    }
    server {
        listen     443;
        proxy_pass rancher_servers_https;
    }

}





*****************************************



5. Do host entry in local machine /etc/hosts with metioned host in step 4

****************************************

4. Change host if want to change


kubectl get ingress --all-namespaces
kubectl edit ingress rancher -n cattle-system

and edit host name and enter 

**************************************








