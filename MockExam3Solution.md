### 1. Create a new service account with the name pvviewer. Grant this Service account access to list all PersistentVolumes in the cluster by creating an appropriate cluster role called pvviewer-role and ClusterRoleBinding called pvviewer-role-binding.
Next, create a pod called pvviewer with the image: redis and serviceAccount: pvviewer in the default namespace

```
kubectl create sa pvviewer

kubectl create clusterrole pvviewer-role --verb=list --resource=persistentvolumes
kubectl create clusterrolebinding pvviewer-role-binding --clusterrole=pvviewer-role --serviceaccount=default:pvviewer
```
https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/
# Edit the service account and add the field "serviceAccountName: pvviewer"
```
kubectl run --generator=run-pod/v1 --dry-run --image redis pvviewer -o yaml > pod.yaml
kubectl create -f pod.yaml
```


### 2. List the InternalIP of all nodes of the cluster. Save the result to a file /root/node_ips
Answer should be in the format: InternalIP of master<space>InternalIP of node1<space>InternalIP of node2<space>InternalIP of node3 (in a single line)

```
//kubectl get nodes -o=custom-columns=InternalIP:.status.addresses[0].address > /root/node_ips
// kubectl get nodes -o=jsonpath='{range .items[*]}{"InternalIP of "}{.status.addresses[1].address}{" "}{.status.addresses[0].address}{" "}' > /root/node_ips
//kubectl get nodes -o=jsonpath='{range .items[*]}{"InternalIP of "}{.status.addresses[1].address}{" "}{.status.addresses[0].address}{" "}' > /root/node_ips

kubectl get nodes -o=jsonpath='{.items[*].status.addresses[0].address}' > /root/node_ips
```

### 3. Create a pod called multi-pod with two containers.
Container 1, name: alpha, image: nginx
Container 2: beta, image: busybox, command sleep 4800.

Environment Variables:
container 1:
name: alpha

Container 2:
name: beta

```
kubectl run --generator=run-pod/v1 --dry-run --image nginx multi-pod -o yaml > pod2.yaml

kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: multi-pod
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: alpha
    env:
      - name: name
        value: alpha
  - image: busybox
    name: beta
    command: ["sleep", "4800"]
    env:
      - name: name
        value: beta
EOF
```

### 4. Create a Pod called non-root-pod , image: redis:alpine
runAsUser: 1000
fsGroup: 2000

```
kubectl create -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: non-root-pod
spec:
  containers:
  - image: redis:alpine
    name: non-root-pod
  securityContext:
    runAsUser: 1000
    fsGroup: 2000
EOF
```

### 5. We have deployed a new pod called np-test-1 and a service called np-test-service. Incoming connections to this service are not working. Troubleshoot and fix it.
Create NetworkPolicy, by the name ingress-to-nptest that allows incoming connections to the service over port 80


Important: Don't delete any current objects deployed.


https://kubernetes.io/docs/concepts/services-networking/network-policies
```
kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: ingress-to-nptest
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: np-test-1
  policyTypes:
  - Ingress
  ingress:
  - from:
    ports:
    - protocol: TCP
      port: 80
EOF
```
* Notice that this network policy apply on all pods with label run=np-test-1 and accept traffing from everywhere.


### 6. Taint the worker node node01 to be Unschedulable. Once done, create a pod called dev-redis, image redis:alpine to ensure workloads are not scheduled to this worker node. Finally, create a new pod called prod-redis and image redis:alpine with toleration to be scheduled on node01.


key:env_type, value:production and operator:NoSchedule


https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/

```
kubectl taint nodes node01 env_type=production:NoSchedule

kubectl run --generator=run-pod/v1 --image redis:alpine dev-redis
```


Check that the pod is not scheduled on node01:
```
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: dev-redis
spec:
  nodeName: node01
  containers:
  - name: redis
    image: redis:alpine
EOF
```

Pod with tolerator on node01
```
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: prod-redis
spec:
  containers:
  - name: redis
    image: redis:alpine
  tolerations:
  - key: "env_type"
    operator: "Equal"
    value: "production"
    effect: "NoSchedule"
EOF
```
