
1.  Deploy a pod named `nginx-pod` using the `nginx:alpine` image.  

Once done, click on the Next Question button in the top right corner of this panel. 
You may navigate back and forth freely between all questions. Once done with all questions, 
click on End Exam. Your work will be validated at the end and score shown. Good Luck!

```
kubectl run --generator=run-pod/v1 nginx-pod --image=nginx:alpine
```

2. Deploy a messaging pod using the redis:alpine image with the labels set to tier=msg.  
```
kubectl run --generator=run-pod/v1 --image redis:alpine --labels=tier=msg messaging
```

3. Create a namespace named apx-x9984574
```
kubectl create ns apx-x9984574
```

4. Get the list of nodes in JSON format and store it in a file at /opt/outputs/nodes-z3444kd9.json
```
kubectl get nodes -o json > /opt/outputs/nodes-z3444kd9.json
```

5. Create a service messaging-service to expose the messaging application within the cluster on port 6379.
Use imperative commands

kubectl expose pod messaging --name=messaging-service --port 6379



6. Create a deployment named hr-web-app using the image kodekloud/webapp-color with 2 replicas

```
kubectl run hr-web-app --image kodekloud/webapp-color --replicas=2
```

7. Create a static pod named static-busybox that uses the busybox image and the command sleep 1000
```
kubectl run --generator=run-pod/v1 static-busybox --image busybox --command "sleep 1000" --dry-run -o yaml > /etc/kubernetes/manifests/static-busybox.yaml
```

8. Create a POD in the finance namespace named temp-bus with the image redis:alpine.
```
kubectl run --generator=run-pod/v1 -n finance temp-bus --image redis:alpine
```
9. A new application orange is deployed. There is something wrong with it. Identify and fix the issue.

Wrong command in init container.

10.  Expose the hr-web-app as service hr-web-app-service application on port 30082 on the nodes on the cluster
The web application listens on port 8080
```
kubectl expose deploy hr-web-app --name=hr-web-app-service --port 8080 --target-port 8080 --type NodePort --dry-run -o yaml > service.yaml
```
Edit the service and add the field `nodePort: 30082`


11. Use JSON PATH query to retrieve the osImages of all the nodes and store it in a file /opt/outputs/nodes_os_x43kj56.txt


The osImages are under the nodeInfo section under status of each node.

```
kubectl get nodes -o=jsonpath='{.items[*].status.nodeInfo.osImage}' > /opt/outputs/nodes_os_x43kj56.txt
```
12. 
Create a Persistent Volume with the given specification:
```
Volume Name: pv-analytics
Storage: 100Mi
Access modes: ReadWriteMany
Host Path: /pv/data-analytics
```

```
kubectl apply -f - <<EOF
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-analytics
spec:
  capacity:
    storage: 100Mi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: slow
  mountOptions:
    - hard
    - nfsvers=4.1
  hostPath:
    path: /pv/data-analytics
EOF
```
