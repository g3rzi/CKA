Create privileged container from line:  
```
kubectl run priv-container --image=ubuntu --overrides='{"spec": {"template": {"spec": {"containers": [{"name": "priv-container", "image": "ubuntu", "command": ["sh", "-c", "sleep infinity"], "securityContext": {"privileged": true} }]}}}}' -- sh -c 'apt update && apt install wget -y && sleep infinity'
```  


From [YAML](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/):
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  volumes:
  - name: sec-ctx-vol
    emptyDir: {}
  containers:
  - name: sec-ctx-demo
    image: busybox
    command: [ "sh", "-c", "sleep 1h" ]
    volumeMounts:
    - name: sec-ctx-vol
      mountPath: /data/demo
    securityContext:
      allowPrivilegeEscalation: false
```
