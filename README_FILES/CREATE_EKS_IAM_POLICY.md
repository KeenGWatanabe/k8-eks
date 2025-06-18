This message indicates your **IAM user/role lacks explicit permissions** to interact with the Kubernetes API in EKS. Here's how to fix it:
![Cluster_IAM_Screenshot](/images/Cluster_IAM.png)
### Solution: Add IAM Access Entry

1. **Go to EKS Console** → Your Cluster → **Access tab** → **Add access entry**

2. **Configure Access**:
   - **Principal ARN**: Your IAM user/role ARN (get it via `aws sts get-caller-identity`)
   - **Type**: "Standard"
   - **Policies**: Select at least:
     - `AmazonEKSClusterAdminPolicy` (full access)  
     *or*  
     - `AmazonEKSAdminPolicy` (limited admin)

3. **Apply Changes**

### Alternative (Terraform)
Add this to your EKS module:
```hcl
resource "aws_eks_access_entry" "admin" {
  cluster_name = aws_eks_cluster.example.name
  principal_arn = "arn:aws:iam::123456789012:user/YourIAMUser"
  kubernetes_groups = ["system:masters"]
  type = "STANDARD"
}
```

### Verification
```bash
aws eks update-kubeconfig --name your-cluster --region us-east-1
kubectl get nodes  # Should now work
```

### Key Notes:
- This is required for **EKS API access** even if you have IAM permissions
- Replaces the old `aws-auth` ConfigMap method
- Must be done for each IAM principal needing access

The message will disappear once proper access is configured. For production, use more granular policies than `system:masters`.