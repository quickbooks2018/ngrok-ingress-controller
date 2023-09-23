# ngrok ingress controller

- eks cluster with terraform https://github.com/quickbooks2018/aws-eks-blueprints.git

- https://github.com/ngrok/kubernetes-ingress-controller

- deploy eks cluster with terraform with single command
```bash
terraform init && terraform apply -auto-approve
```

- signup for ngrok account is required to get authtoken and api key
- must reset the authoken because it is for teams
- https://ngrok.com/blog-post/ngrok-k8s

## Installation with helm

```bash
export NAMESPACE='[YOUR_K8S_NAMESPACE]'
export NGROK_AUTHTOKEN='[AUTHTOKEN]'
export NGROK_API_KEY='[API_KEY]'

helm repo add ngrok https://ngrok.github.io/kubernetes-ingress-controller
helm repo update
helm repo ls
helm search repo ngrok
helm search repo ngrok/kubernetes-ingress-controller
helm show values ngrok/kubernetes-ingress-controller
helm search repo ngrok/kubernetes-ingress-controller --versions
helm show chart ngrok/kubernetes-ingress-controller --version 0.11.0
helm show readme ngrok/kubernetes-ingress-controller --version 0.11.0
helm show values ngrok/kubernetes-ingress-controller --version 0.11.0
helm show all ngrok/kubernetes-ingress-controller --version 0.11.0
helm show values ngrok/kubernetes-ingress-controller --version 0.11.0 > values.yaml

helm upgrade --install ngrok-ingress-controller ngrok/kubernetes-ingress-controller \
  --version 0.11.0 \
  --namespace "$NAMESPACE" \
  --create-namespace \
  --set credentials.apiKey="$NGROK_API_KEY" \
  --set credentials.authtoken="$NGROK_AUTHTOKEN" \
  --set controller.ingressClassResource.name=ngrok
```

- ingress setup
```helm
helm create hello
cd hello
helm -n hello upgrade --install hello --create-namespace -f values.yaml ./
 ```
- ingress setup
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello
  namespace: hello
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
    spec:
      containers:
        - name: hello
          image: docker.io/nginx:latest
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: hello
  namespace: hello
spec:
  ports:
    - port: 80
      protocol: TCP
      targetPort: 80
  selector:
    app: hello
  type: ClusterIP

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hello-ingress
  namespace: hello
spec:
  ingressClassName: ngrok
  rules:
    - host: central-bat-merely.ngrok-free.app
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: hello
                port:
                  number: 80
```
