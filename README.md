# Prerequisites for EKS Setup
1. Install AWS Load Balancer Controller
helm repo add eks https://aws.github.io/eks-charts
helm repo update
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=your-cluster-name \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller
2. Install Secrets Store CSI Driver
helm repo add secrets-store-csi-driver https://kubernetes-sigs.github.io/secrets-store-csi-driver/charts
helm install csi-secrets-store secrets-store-csi-driver/secrets-store-csi-driver \
  --namespace kube-system \
  --set syncSecret.enabled=true
3. Install AWS Provider for Secrets Store CSI Driver
kubectl apply -f https://raw.githubusercontent.com/aws/secrets-store-csi-driver-provider-aws/main/deployment/aws-provider-installer.yaml

# Deployment Order
Apply namespace
Apply serviceaccount
Apply configmap
Apply secret-provider-class
Apply deployment
Apply service
Apply ingress
Apply hpa
Apply pdb
Apply network-policy (if needed)

# Customization Notes
Replace YOUR_ACCOUNT_ID, YOUR_ECR_REPO_URI, REGION, CERT_ID with your actual values
Update secret names in AWS Secrets Manager to match your existing setup
Adjust resource limits/requests based on your application requirements
Modify health check endpoints to match your application
Update domain names and certificate ARNs for your ingress
Configure autoscaling parameters based on your traffic patterns
