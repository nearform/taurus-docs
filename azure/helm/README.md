# Install Kubernetes Add-Ons 

Use Helm to install Kubernetes add-ons. Helm, the package manager for Kubernetes, is a useful tool to install, upgrade and manage applications on a Kubernetes cluster. Helm packages are called charts. 

## Install Helm 3.x
A binary of Helm 3.x can be downloaded from here: https://github.com/helm/helm/releases

## Install Metrics Server
Use the following command to install metrics server:
```sh
helm upgrade --install "metrics-server" "stable/metrics-server" --version "2.8.5" --namespace "kube-system" \
--set replicas="2"
```

## Install AAD Pod Identity
Install AAD Pod Identity using the following commands:
```sh
helm repo add aad-pod-identity https://raw.githubusercontent.com/Azure/aad-pod-identity/master/charts

helm install aad-pod-identity aad-pod-identity/aad-pod-identity
```

## Install Application Gateway Ingress Controller
Application Gateway Ingress Controller was released late in 2019. It leverages Application Gateway from Azure to
route the traffic from the Internet into our AKS services. There is a number of dependencies that need to be installed
using the [official documentation](https://github.com/Azure/application-gateway-kubernetes-ingress/blob/master/docs/setup/install-new.md)
and depending on your needs.

Once they are satisfied, run the following command:
```sh
helm install ingress-azure \
  -f helm-config.yaml \
  application-gateway-kubernetes-ingress/ingress-azure \
  --version 1.2.1
```

## Install ExternalDNS
In order to deploy the external DNS we need to create a deployment:
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: external-dns
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: external-dns
rules:
- apiGroups: [""]
  resources: ["services","endpoints","pods"]
  verbs: ["get","watch","list"]
- apiGroups: ["extensions","networking.k8s.io"]
  resources: ["ingresses"] 
  verbs: ["get","watch","list"]
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["list"]
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: external-dns-viewer
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: external-dns
subjects:
- kind: ServiceAccount
  name: external-dns
  namespace: default
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: external-dns
spec:
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: external-dns
  template:
    metadata:
      labels:
        app: external-dns
    spec:
      serviceAccountName: external-dns
      containers:
      - name: external-dns
        image: k8s.gcr.io/external-dns/external-dns:v0.7.3
        args:
        - --source=service
        - --source=ingress
        - --domain-filter=<your-domain>
        - --provider=azure
        - --azure-resource-group=<your-resource-group>
        volumeMounts:
        - name: azure-config-file
          mountPath: /etc/kubernetes
          readOnly: true
      volumes:
      - name: azure-config-file
        secret:
          secretName: azure-config-file
```
Please be aware that the parameters `--domain-filter` and `--azure-resource-group` need to be customized.
Save it in a file called `external-dns.yaml` and execute:
```sh
kubectl apply -f external-dns.yaml
```

This deployment will configure external DNS in AKS to publish the DNS records as required.
