# branch: (module)
# modular deployment
taskmgr > tf-backend > vpc-nat-eip-ec2 > tf-eks2 > k8-eks (alb with kubectl yaml files)

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
# Deploy components
kubectl apply -f alb-controller-iam.yaml
kubectl apply -f alb-controller-deployment.yaml
kubectl apply -f nodejs-app.yaml

# Verification
# Check ALB controller
kubectl get pods -n kube-system -l app.kubernetes.io/name=aws-load-balancer-controller

# Check your app
kubectl get ingress nodejs-ingress
kubectl get svc nodejs-service
------------------------

# Key Benefits
Simpler - No Terraform Helm complexity
Visible - All config in YAML files
Standard - Pure Kubernetes native approach
Maintainable - Easy to version control
Would you like me to:
Add health checks to the Node.js deployment?
Include autoscaling configuration?
Add monitoring setup?

