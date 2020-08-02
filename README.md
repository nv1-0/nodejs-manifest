# nodejs-manifest
<h2>This project consists of a helm chart for node js manifest having following configuration</h2>

* Node js server with minimum 10 replicas
* Deployment  autoscales at average 50% cpu and 60% memory.
* Server is exposed by EC2 Classic load balancer at 3000 port
* Any change to the deployment always ensures at least 7 replicas are running at all times.
* Higher priority than daemonset pods.

<h3> Running this helm command inside this repo will generate all the manifests in yaml format</h3>

```go
 helm template nodejs-chart/ --namespace nodejs
```

The manifests generated are
```yaml
---
# Source: nodejs-chart/templates/pod-disruption-budget.yaml
apiVersion: policy/v1beta1
kind: PodDisruptionBudget
metadata:
  name: node-js-pdb
  namespace: nodejs
spec:
  minAvailable: 7
  selector:
    matchLabels:
      run: node-js
---
# Source: nodejs-chart/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: node-js
  namespace: nodejs
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "classic"
  labels:
    run: node-js
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 3000
  selector:
    run: node-js
  type: LoadBalancer
---
# Source: nodejs-chart/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: node-js
  namespace: nodejs
spec:
  selector:
    matchLabels:
      run: node-js
  template:
    metadata:
      labels:
        run: node-js
    spec:
      containers:
      - name: node-js
        image: aws_account_id.dkr.ecr.us-west-2.amazonaws.com/nodejs-test:latest
        ports:
        - containerPort: 3000
        resources:
          limits:
            cpu: 500m
          requests:
            cpu: 100m
      priorityClassName: system-node-critical
---
# Source: nodejs-chart/templates/horizontal-pod-scaler.yaml
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: node-js-scaler
  namespace: nodejs
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: node-js
  minReplicas: 10
  maxReplicas: 30
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 60
```

<h3>To deploy this chart inside kubernetes run this command</h3>

```go
helm upgrade --install nodejs nodejs-chart/ --namespace nodejs
```
