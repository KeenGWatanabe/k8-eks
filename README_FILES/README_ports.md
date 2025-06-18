### **Port Summary**
| Config File       | Key Field                            | Must Match                          |
|-------------------|--------------------------------------|-------------------------------------|
| `service.yaml`    | `spec.ports.targetPort`              | `deployment.yaml`’s `containerPort` |
| `ingress.yaml`    | `backend.service.port.number`        | `service.yaml`’s `port` (80)        |
| `deployment.yaml` | `spec.containers.ports.containerPort`| `service.yaml`’s `targetPort`       |

**Final Check:**  
Run these to verify:
```sh
kubectl describe svc/app-service  # Check "TargetPort"
kubectl describe pod <pod-name>   # Check "ContainerPort"
```

Absolutely! Here’s a clear reference table for **`service.name`** across your Kubernetes YAML files, similar to the `targetPort` table:  

---

### **service.name Cross-Reference Table**
| File                  | Field                                        | Must Match With                                  | Purpose                                                                 |
|-----------------------|----------------------------------------------|--------------------------------------------------|-------------------------------------------------------------------------|
| **`service.yaml`**    | `metadata.name`                              | **Referenced in `ingress.yaml`**                 | Defines the Service’s name, which the Ingress uses to route traffic.    |
| **`ingress.yaml`**    | `spec.rules.http.paths.backend.service.name` | **Must match `service.yaml`’s `metadata.name`**  | Tells the Ingress which Service to forward traffic to.                  |
| **`deployment.yaml`** | (Indirect) `spec.selector.app`               | **Matches `service.yaml`’s `spec.selector.app`** | Ensures the Service selects the correct Pods (via labels).              |

---

### **Visual Workflow**
```
User → Ingress (app-ingress) → Service (app-service) → Pods (labeled `app: app`)
```
- **Critical Links**:  
  - **Ingress → Service**: `name: app-service` in `ingress.yaml`.  
  - **Service → Pods**: `selector.app: app` in `service.yaml` + Pod labels.

------------------------------Service-account versus Service-----------------------------
Key Differences
Ah, I see the issue! Your `service-account.yaml` is incorrectly mixing **ServiceAccount** and **Service** definitions. Let me fix this:

### The Problem
- You're trying to create a `ServiceAccount` but included `Service` specifications (`type`, `selector`, `ports`)
- `ServiceAccount` resources don't have a `spec` field

### Corrected File
Here's the proper `service-account.yaml`:
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: aws-load-balancer-controller
  namespace: kube-system
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::255945442255:role/aws-load-balancer-controller
```

### If You Need a Service
For a LoadBalancer service, create a separate file (e.g., `service.yaml`):
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: LoadBalancer
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
```

### Key Differences
| Resource       | Purpose                          | Fields Allowed                  |
|----------------|----------------------------------|----------------------------------|
| ServiceAccount | Identity for pods                | Only `metadata` with annotations |
| Service        | Network access to applications   | Requires `spec` with ports       |

### Steps to Fix
1. Delete the incorrect file
2. Apply the corrected ServiceAccount:
   ```bash
   kubectl apply -f service-account.yaml
   ```
3. (Optional) If you need a service, create and apply the `service.yaml` separately

### Verification
```bash
kubectl get serviceaccount -n kube-system aws-load-balancer-controller -o yaml
```
Should show just the metadata without any `spec` section.

------------------------------Service-account versus Service-----------------------------