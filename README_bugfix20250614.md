I checked your repository (`https://github.com/KeenGWatanabe/k8-eks`), and here are the fixes needed for your Kubernetes deployment to work properly with AWS EKS + ALB Ingress.

---

## **Issues Found & Fixes**
### **1. `ingress.yaml` Fix**
Your current `ingress.yaml` has:
- Incorrect `service.name` (`secrets-app-service` instead of `app-service`).
- No hostname (optional, but recommended for production).
- Health check port `5000` (but your app likely listens on `80`).

#### **Fixed `ingress.yaml`**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/healthcheck-path: /health
    alb.ingress.kubernetes.io/healthcheck-port: "80"  # Changed to match service port
spec:
  ingressClassName: alb
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app-service  # Must match service.yaml
            port:
              number: 80
```

---

### **2. `service.yaml` Fix**
Your `service.yaml` is correct, but ensure it matches the Ingress:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: app-service  # This must match ingress.yaml's service.name
spec:
  selector:
    app: app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000  # Ensure your app listens on 5000
  type: NodePort
```

**If your app listens on `5000`**, keep `targetPort: 5000`.  
**If your app listens on `80`**, change it to `targetPort: 80`.

---

### **3. `deployment.yaml` Fix**
Your `deployment.yaml` is mostly correct, but:
- Ensure `containerPort` matches `targetPort` in `service.yaml`.
- If using `5000`, keep it. If using `80`, update it.

#### **Example (if app runs on 5000)**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: app
  template:
    metadata:
      labels:
        app: app
    spec:
      containers:
      - name: app
        image: your-ecr-repo/your-app:latest
        ports:
        - containerPort: 5000  # Must match service.yaml's targetPort
        envFrom:
        - secretRef:
            name: app-secrets
```

---

## **Debugging Steps**
### **1. Apply Fixed Configs**
```sh
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml
```

### **2. Check ALB Status**
```sh
kubectl get ingress -w
```
- Wait for `ADDRESS` to appear (takes 1-5 mins).  
- If it stays empty, check logs:
  ```sh
  kubectl logs -n kube-system $(kubectl get pods -n kube-system | grep alb-ingress-controller | awk '{print $1}')
  ```

### **3. Verify Service & Pods**
```sh
kubectl get svc  # Ensure 'app-service' exists
kubectl get pods # Ensure pods are 'Running'
kubectl describe ingress app-ingress  # Check for errors
```

### **4. Access Your App**
Once `kubectl get ingress` shows an `ADDRESS` (e.g., `xxxxx.elb.amazonaws.com`), open it in a browser:
```sh
curl http://<ALB-DNS>/
```

---

## **Final Notes**
- **ALB DNS**: You’ll find it in `kubectl get ingress` or AWS **EC2 → Load Balancers**.
- **Health Checks**: Ensure `/health` endpoint exists (or modify `healthcheck-path`).
- **Custom Domain**: If you want `myapp.example.com`, add `host:` to `ingress.yaml`.

Let me know if you still face issues! 🚀