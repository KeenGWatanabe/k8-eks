### Full Recovery Process
1. Uninstall any broken deployment:
   ```bash
   helm uninstall -n kube-system aws-load-balancer-controller
   ```

2. Clean up existing resources:
   ```bash
   kubectl delete mutatingwebhookconfiguration aws-load-balancer-webhook
   kubectl delete validatingwebhookconfiguration aws-load-balancer-webhook
   ```

3. Reinstall using the exact values from your cluster:
   ```bash
   helm install -f values.yaml aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system
   ```

After installation, wait 2-3 minutes for the webhook endpoints to become available before creating Services. The error should resolve once the controller pods are healthy and serving requests.