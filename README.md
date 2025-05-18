1. **Run backend S3**
  https://github.com/KeenGWatanabe/tf-backend

2. **Run tf-eks***
  https://github.com/KeenGWatanabe/tf-eks2
  Pull out the `oidc arn and input into service-account.yaml ln6`


1. **Build & Push Docker Image**  
   - Use the same Docker image from `taskmgr repo` build ECR `image_uri` and push
   into `deployment.yaml ln18 `

   **Setup EKS cluster and config**
aws eks list-clusters
aws eks update-kubeconfig --name grp-4-eks-cluster --region us-east-1
kubectl create namespace grp-4-eks-activity


2. **Apply Kubernetes Manifests**  
   ```sh
kubectl apply -f configmap.yaml
kubectl apply -f ingress.yaml

kubectl apply -f deployment.yaml
kubectl apply -f service-account.yaml  
   ```
3. **Verify**  
   ```sh
   kubectl get pods
   kubectl get ingress  # Check ALB URL
   ```


***DeepSeek***
Great! You want to **migrate** the infrastructure from **Terraform ECS (`tf-ecs`)** to **Kubernetes (using `tf-eks2`)**. Let's analyze both repos and generate the necessary Kubernetes manifests.

---

### **1. Analysis of `tf-ecs` (Current ECS Setup)**
From [`tf-ecs/main.tf`](https://github.com/KeenGWatanabe/tf-ecs/blob/main/main.tf), the ECS infrastructure includes:
- **ECS Cluster** → Equivalent to a **K8s Cluster** (already handled by `tf-eks2`).
- **ECS Task Definition** → Equivalent to **K8s Deployment + Service**.
- **ALB (Application Load Balancer)** → Equivalent to **K8s Ingress**.
- **Security Groups** → Handled by **K8s Network Policies** + **AWS Security Groups** (via `tf-eks2`).
- **IAM Roles** → Handled by **K8s Service Accounts** + **IAM Roles for Service Accounts (IRSA)**.

---

### **2. Kubernetes Manifests to Replace `tf-ecs`**
Based on the `tf-ecs` setup, here are the required **K8s YAML files**:

#### **A. `deployment.yaml` (Replaces ECS Task Definition)**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
spec:
  replicas: 2  # Adjust as needed
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      serviceAccountName: app-service-account  # For IAM roles (IRSA)
      containers:
      - name: app-container
        image: YOUR_ECR_IMAGE_URL  # Replace with your Docker image
        ports:
        - containerPort: 80  # Same as ECS task's container port
        envFrom:
        - configMapRef:
            name: app-config
        - secretRef:
            name: app-secrets
---
# Service (internal LB)
apiVersion: v1
kind: Service
metadata:
  name: app-service
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP  # Use NodePort/LoadBalancer if needed
```

#### **B. `ingress.yaml` (Replaces ALB)**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app-service
            port:
              number: 80
```

#### **C. `configmap.yaml` (For Environment Variables)**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  ENV_VAR_1: "value1"
  ENV_VAR_2: "value2"
```

#### **D. `service-account.yaml` (For IAM Roles - IRSA)**
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-service-account
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::YOUR_ACCOUNT_ID:role/YOUR_IAM_ROLE
```

---

### **3. Key Adjustments from `tf-ecs` to `tf-eks2`**
1. **Replace `aws_ecs_task_definition` → K8s `Deployment`**  
   - Define replicas, resources (CPU/memory), and env vars in `deployment.yaml`.
2. **Replace `aws_lb` → K8s `Ingress` with ALB annotations**  
   - The `tf-eks2` repo already sets up the AWS Load Balancer Controller.
3. **IAM Roles → IRSA (IAM Roles for Service Accounts)**  
   - Use `serviceAccountName` in the Deployment and annotate with the IAM role ARN.
4. **Security Groups → K8s Network Policies**  
   - Define `NetworkPolicy` if pod-level restrictions are needed.

---

### **4. Steps to Deploy**
1. **Build & Push Docker Image**  
   - Use the same Docker image from `tf-ecs` (or modify if needed).
2. **Apply Kubernetes Manifests**  
   ```sh
   kubectl apply -f deployment.yaml
   kubectl apply -f service.yaml
   kubectl apply -f ingress.yaml
   kubectl apply -f configmap.yaml
   ```
3. **Verify**  
   ```sh
   kubectl get pods
   kubectl get ingress  # Check ALB URL
   ```

---

### **5. Optional Improvements**
- **Autoscaling**: Add `HorizontalPodAutoscaler` (HPA).
- **Secrets**: Use `kubectl create secret` for sensitive data.
- **Monitoring**: Integrate Prometheus/Grafana (if needed).

---

### **Next Steps**
Would you like me to:
1. Provide **specific YAMLs** for your app’s exact ports/env vars?
2. Help debug any **Terraform-to-K8s migration issues**?
3. Optimize the setup further (e.g., **Helm charts, CI/CD**)?

Let me know how you'd like to proceed! 🚀