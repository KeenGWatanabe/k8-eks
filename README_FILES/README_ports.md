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

