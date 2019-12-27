# CKA

# General commands

`kubectl run --dry-run=false` -> of true, it prints the object without sending it.
`kubectl run --restart=Never` -> regular pod is created.

# Sections

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


116. Certificates API
`openssl genrsa -out jane.key 2048`
`openssl req -new -key jane.key -subj "/CN=jane" -out jane.csr`
Creating jane-csr.yaml:
kind: CertificateSigningRequest
...


`kubectl get csr`
`kubectl certificate approve jane`



## Network:
	160-161:
		`cat /etc/cni/net.d/` -> check the config file for the network provider
		`ip addr show weave` -> show ip addresses of the provider
