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
	
## Security
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
