1. Take a backup of the etcd cluster and save it to /tmp/etcd-backup.db
2. Create a Pod called redis-storage with image: redis:alpine with a Volume of type emptyDir that lasts for the life of the Pod. Specs on the right.

```
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: redis-storage
spec:
  containers:
  - image: redis:alpine
    name: test-container
    volumeMounts:
    - mountPath: /data/redis
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir: {}
EOF
```

3. Create a new pod called super-user-pod with image busybox:1.28. Allow the pod to be able to set system_time
# https://kubernetes.io/docs/tasks/configure-pod-container/security-context/
```
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: super-user-pod
spec:
  containers:
  - name: sec-ctx-4
    image: busybox:1.28
    command: ["sleep", "4800"]
    securityContext:
      capabilities:
        add: ["SYS_TIME"]
EOF
```

4. A pod definition file is created at /root/use-pv.yaml. Make use of this manifest file and mount the persistent volume called pv-1. Ensure the pod is running and the PV is bound.
https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims
```
kubectl apply -f - <<EOF
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Mi
EOF
```
https://kubernetes.io/docs/concepts/storage/persistent-volumes/#claims-as-volumes
Edit /root/use-pv.yaml:
Take this:
```
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim
```

```
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: use-pv
  name: use-pv
spec:
  containers:
  - image: nginx
    name: use-pv
    volumeMounts:
    - mountPath: "/data"
      name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: my-pvc
EOF
```




5. Create a new deployment called nginx-deploy, with image nginx:1.16 and 1 replica. Record the version. Next upgrade the deployment to version 1.17 using rolling update. Make sure that the version upgrade is recorded in the resource annotation.
```
kubectl run nginx-deploy --image nginx:1.16 --record
kubectl rollout history deployment
kubectl set image deployment/nginx-deploy nginx-deploy=nginx:1.17 --record
kubectl rollout history deployment
```

6. Create a new user called john. Grant him access to the cluster. John should have permission to create, list, get, update and delete pods in the development namespace . The private key exists in the location: /root/john.key and csr at /root/john.csr

```
kubectl create role developer -n development --resource=pods --verb=create,list,get,update,delete
kubectl create rolebinding developer-binding -n development --role developer --user=john
```

https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/
```
cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: john-developer
spec:
  request: $(cat /root/john.csr | base64 | tr -d '\n')
  usages:
  - digital signature
  - key encipherment
  - server auth
EOF

kubectl get csr
kubectl certificate approve john-developer
```


7. Create an nginx pod called nginx-resolver using image nginx, expose it internally with a service called nginx-resolver-service. Test that you are able to look up the service and pod names from within the cluster. Use the image: busybox:1.28 for dns lookup. Record results in /root/nginx.svc and /root/nginx.pod
`kubectl run --restart=Never --image nginx nginx-resolver --labels=app=nginx-resolver`
https://kubernetes.io/docs/concepts/services-networking/service/
```
kubectl apply -f - <<EOF
apiVersion: v1
kind: Service
metadata:
  name: nginx-resolver-service
spec:
  selector:
    app: nginx-resolver
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
EOF
```

`kubectl run --restart=Never --image busybox:1.28 dns -- sleep 1000`
```
# IP of the service
# kubectl get svc nginx-resolver-service -o=jsonpath='{.spec.clusterIP}'
# kubectl exec -it dns -- nslookup 10.111.255.127 > /root/nginx.svc
kubectl exec -it dns -- nslookup $(kubectl get svc nginx-resolver-service -o=jsonpath='{.spec.clusterIP}') > /root/nginx.svc

#kubectl get pods nginx-resolver -o=jsonpath='{.status.podIP}'
kubectl exec -it dns -- nslookup $(kubectl get pods nginx-resolver -o=jsonpath='{.status.podIP}') > /root/nginx.pod
```


8. Create a static pod on node01 called nginx-critical with image nginx. Create this pod on node01 and make sure that it is recreated/restarted automatically in case of a failure.
`ssh node01`
https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/
`cat /var/lib/kubelet/config.yaml | grep stat`

```
cat <<EOF > /etc/kubernetes/manifests/nginx-critical.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-critical
spec:
  containers:
    - name: web
      image: nginx
      ports:
        - name: web
          containerPort: 80
          protocol: TCP
EOF
```

From the master:
`kubectl delete pods`







