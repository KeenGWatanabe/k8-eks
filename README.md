# Branches use in this repo
/main use `taskmgr-eks` > use backend > tf-eks2 > alb by helm resource
/dev use `tf-secretsAPP`
/secrets use `secrets`
/module use > backend > vpc > tf-eks2 > alb by kubectl

# Deployment steps
# 1 Apply IAM Role (one-time)
```bash
aws iam create-role --role-name eks-alb-controller-role --assume-role-policy-document file://<(cat <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::YOUR_ACCOUNT_ID:oidc-provider/oidc.eks.us-east-1.amazonaws.com/id/YOUR_OIDC_ID"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "oidc.eks.us-east-1.amazonaws.com/id/YOUR_OIDC_ID:sub": "system:serviceaccount:kube-system:aws-load-balancer-controller"
        }
      }
    }
  ]
}
EOF
)

aws iam attach-role-policy \
  --role-name eks-alb-controller-role \
  --policy-arn arn:aws:iam::aws:policy/AWSLoadBalancerControllerIAMPolicy
  ```
  # Deploy Components
  kubectl apply -f alb-controller-iam.yaml
kubectl apply -f alb-controller-deployment.yaml
kubectl apply -f nodejs-app.yaml

# Verification
# Check ALB controller
kubectl get pods -n kube-system -l app.kubernetes.io/name=aws-load-balancer-controller

# Check your app
kubectl get ingress nodejs-ingress
kubectl get svc nodejs-service

# Apply Autoscaling
# Apply HPA for your app
kubectl apply -f nodejs-app.yaml

# Set up Cluster Autoscaler (one-time)
kubectl apply -f cluster-autoscaler.yaml

# Verification
# Check HPA status
kubectl get hpa

# Check cluster autoscaler logs
kubectl logs -n kube-system deployment/cluster-autoscaler

Recommended Additions
# 1. Custom Metrics (e.g. requests per second):
metrics:
- type: Pods
  pods:
    metric:
      name: http_requests_per_second
    target:
      type: AverageValue
      averageValue: 100

# 2. Pod Disruption Budget
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: nodejs-app-pdb
spec:
  minAvailable: 1
  selector:
    matchLabels:
      app: nodejs