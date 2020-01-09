# CKA - Summary for the exam

# General commands

`kubectl run --dry-run=false` -> of true, it prints the object without sending it.  
`kubectl run --restart=Never` -> regular pod is created.  

# Sections
## Section 3: Scheduling  
### 48. Node Selectors  
Two ways to assign pod to nodes:  
1. Node Selectors     
    Assign lable to node:  
	`kubectl label nodse <node-name> <label-key>=<label-value>`  
	Example: `kubectl lable nodes node-1 size=Large`  

	Create the pod with `nodeSelector` field like that:  
	```
	apiVersion: v1
	kind: Pod
	metadata:
	  name: myapp-pod
	spec:
	  nodeSelector:
	    kubernetes.io/hostname: node03
	  containers:
	  - image: data-processor
	    name: data-processor
	```

	Good method but sometimes you need more complexed assignments like: Large OR Medium, Not Small, etc.  
	For this, you will need to use Node Affinity.  

2. Node Affinity

## Section 6: Cluster Maintenance
### 105. Backup and Restore Methods
When configure ETCD, it is possible to configure where all the data will be store.  
It appears in the `etcd.service` (which can be seen by `cat /etc/kubernetes/manifests/etcd.yaml`) under the flag `--data-dir=/var/lib/etcd`.  

We can take a snapshot of the database by running:  
`ETCDCTL_API=3 etcdctl snapshot save snapshot.db`   

Remember to use it will all the details:  
```
ETCDCTL_API=3 etcdctl snapshot save snapshot.db \
              --endpoints=https://127.0.0.1:2379
	      --cacert=/etc/etcd/ca.crt \
	      --cert=/etc/etcd/etcd-server.crt \
	      --key=/etc/etcd/etcd-server.key
```
To restore the backup:  
Stop the service:  
`service kube-apiserver stop`  
Then run:
```
ETCDCTL_API=3 etcdctl snapshot restore snapshot.db \
--data-dir /var/lib/etcd-from-backup \
--initial-cluster master-1=https://192.168.5.11:2380,master-2=https://192.168.5.12:2380 \
--initial-advertise-peer-urls https://${INTERNAL_IP}:2380
```

Then we will update `etcd.service` file with: 
```
--initial-cluster-token etcd-cluster-1 \
...
--data-dir=/var/lib/etcd-from-backup
```

Then restart the daemon:  
```
systemctl daemon-reload
service etcd restart
service kube-apiserver start
```

## Section 7: Security
```
openssl x509 -req -in /etc/kubernetes/pki/apiserver-etcd-client.csr \
-CA /etc/kubernetes/pki/etcd/ca.crt \
-CAkey /etc/kubernetes/pki/etcd/ca.key \
-CAcreateserial \
-out /etc/kubernetes/pki/apiserver-etcd-client.crt \
```

### view certificate
`openssl x509 -in <certificate path> -text -noout`  


### 116. Certificates API  
`openssl genrsa -out jane.key 2048`  
`openssl req -new -key jane.key -subj "/CN=jane" -out jane.csr`  
Creating jane-csr.yaml:  
kind: CertificateSigningRequest  
...  


`kubectl get csr`  
`kubectl certificate approve jane`  



## Network:  
### 159 & 160. CNI weave   
`cat /etc/cni/net.d/` -> check the config file for the network provider  
`ip addr show weave` -> show ip addresses of the provider  
### 161. Practice Test Deploy Network Solution
 
`kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"` -> Install weave-network

### 163. Service Networking		
```
kubectl get pods -o wide
kubectl get service
# Showing the rules from the service to the pod's port.
iptables -L -t net | grep db-service
# Showing what proxy is being used
cat /var/log/kube-proxy.log
```

### 164. Practice Test - Service Networking		

Check the range of IP addresses configured for PODs on the cluster:  
The network is configured with weave. Check the weave pods logs using command `kubectl logs -n kube-system <weave-pod-name> weave` and look for ipalloc-range

Check the IP range configured for the services within the cluster:  
Inspect the setting on kube-api server by running on command `ps -aux | grep kube-api`
You should find something like that:  
`--service-cluster-ip-range=10.96.0.0/12`

Checl what type of proxy the kube-proxy configured to use:  
Check the logs of the kube-proxy pods. Command: `kubectl logs -n kube-system <kube-proxy pod name>`
Most of the time it will be iptables.

### 166. CoreDNS in Kubernetes  
Configuration file of the DNS:    
`cat /etc/coredns/Corefile`  
This file can be seen using:     
`kubectl get configmap -n kube-system`  

The DNS also creates a service named: kube-dns to be available through the cluster.
We can see the DNS configuration in the kubelet configuration file:  
`/var/lib/kubelet/config.yaml`  

### 167. Practice Test - Explore DNS
Check what dns domain/zone configured on the cluster run:
`kubectl describe configmaps coredns -n kube-system`  

### 168. Ingress
#### Ingress Controller

```
#Deployment of the nginx-ingress image
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-ingress-controller
spec:
  replicas: 1
  selector:
    matchLabels:
      name: nginx-ingress
    template:
      metadata:
        labels:
          name: nginx-ingress
    spec:
      containers:
        - name: nginx-ingress-controller
          image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.21.0
        args:
          - /nginx-ingress-controller
          - --configmap=$(POD_NAMESPACE)/nginx-coinfguration
        env:
          - name: POD_NAME
            valueFrom:
              fieldRef:
                fieldPath: metadata.name
          - name: POD_NAMESPACE
            valueFrom:
              fieldRef:
                fieldPath: metadata.namespace
        ports:
          - name: http
            containerPort: 80
          - name: https
            containerPort: 443
```

```
# ConfiguMap to feed nginx configuration data
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-coinfguration
```

```
# Service to expose the deployment
apiVersion: v1
kind: Service
metadata:
  name: nginx-ingress
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
  - port: 443
    targetPort: 443
    protocol: TCP
    name: https
  selector:
    name: nginx-ingress
```

```
# The service account should have Roles, ClusterRoles and RoleBindings to access the above objects
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nginx-ingress-serviceaccount
```	
#### Ingress Resource
	
	
Ingress-wear.yaml
```
apiVersion: extensions/v1beta1
kind: Ingress
metdata:
  name: ingress-wear
spec:
  backend:
    serviceName: wear-service
	servicePort: 80
```
		
#### Rules
##### First method: Split traffic by URL (one rule) 
```
apiVersion: extensions/v1beta1
kind: Ingress
metdata:
  name: ingress-wear-watch
spec:
  rules:
  - http:
      paths:
      - path: /wear
        backend:
          serviceName: wear-service
          servicePort: 80
      - path: /watch
        backend:
          serviceName: watch-service
          servicePort: 80  	
```
	
##### Second method: Using domain names (two rules)
```
apiVersion: extensions/v1beta1
kind: Ingress
metdata:
  name: ingress-wear-watch
spec:
  rules:
  - host: wear.my-online-store.com
    http:
      paths:
      - backend:
          serviceName: wear-service
          servicePort: 80
    - host: watch.my-online-store.com
      http:
        paths:
        - backend: 
            serviceName: watch-service
            servicePort: 80   
```
	
`kubectl describe ingress ingress-wear-watch`
The `Default backend` is for the case where a user tries to access a URL that does not match any of the rules.
The user is directed to the service specified as the default backend. You must deploy the specified service.


### 217. Practice Test - Worker Node Failure 
Check kubelet status on the nodes:  
`journalctl -u kubelet`  
`service kubelet status`  
    
Related files to kubelet:  
 ```
/etc/kubernetes/kubelet.conf  
/etc/systemd/system/kubelet.service.d/10-kubeadm.conf  
/var/lib/kubelet/config.yaml -> IMPORTANT, contains kubelet settings  
```

### 222. Practice Test - Advanced Kubetl Commands
Find user aws-user inside a context:  
`kubectl config view --kubeconfig=my-kube-config -o jsonpath="{.contexts[?(@.context.user=='aws-user')].name}" > /opt/outputs/aws-context-name`
