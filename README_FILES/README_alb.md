This error occurs when Terraform cannot find the specified add-on version for your EKS cluster. Let's analyze and fix this:

### Root Cause
The error `Error: reading EKS Add-On version info (aws-load-balancer-controller, 1.32): empty result` indicates:
1. AWS doesn't have a published version of the AWS Load Balancer Controller add-on for Kubernetes 1.32
2. Your configuration is missing required parameters for add-on version resolution

### Fix for Your Configuration

In your [tf-eks2/alb repo](https://github.com/KeenGWatanabe/tf-eks2/tree/alb), modify `eks.tf`:

```hcl
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "~> 20.31"

  cluster_name    = "${var.name_prefix}-eks-cluster"
  cluster_version = "1.29" # Changed from 1.32 (not all add-ons support latest versions)

  cluster_addons = {
    aws-load-balancer-controller = {
      most_recent = true # Let AWS select latest compatible version
      resolve_conflicts = "OVERWRITE"
    }
    # Other add-ons...
  }

  # Rest of your config...
}
```

### Key Changes Needed:

1. **Cluster Version Compatibility**:
   - As of June 2024, AWS Load Balancer Controller doesn't have published add-on versions for 1.32
   - Downgrade to 1.29 (or check [AWS docs](https://docs.aws.amazon.com/eks/latest/userguide/eks-add-ons.html) for supported versions)

2. **Add-on Configuration**:
   ```diff
   cluster_addons = {
     aws-load-balancer-controller = {
   +   most_recent = true
   +   resolve_conflicts = "OVERWRITE"
     }
   }
   ```

### Alternative Solutions

**Option 1: Manual Add-on Installation**
```hcl
cluster_addons = {
  aws-load-balancer-controller = {
    addon_version = "v2.8.1-eksbuild.1" # Explicitly set known version
  }
}
```

**Option 2: Helm Installation Instead**
Remove from `cluster_addons` and use:
```hcl
resource "helm_release" "alb_controller" {
  name       = "aws-load-balancer-controller"
  repository = "https://aws.github.io/eks-charts"
  chart      = "aws-load-balancer-controller"
  namespace  = "kube-system"
  version    = "1.7.1" # Chart version

  set {
    name  = "clusterName"
    value = module.eks.cluster_name
  }

  depends_on = [module.eks]
}
```

### Verification Steps

After applying changes:
```bash
# Check add-on status
aws eks describe-addon \
  --cluster-name taskmgr-eks-cluster \
  --addon-name aws-load-balancer-controller

# Verify controller pods
kubectl get pods -n kube-system -l app.kubernetes.io/name=aws-load-balancer-controller
```

### Prevention Tips

1. Always check [AWS EKS Add-Ons documentation](https://docs.aws.amazon.com/eks/latest/userguide/eks-add-ons.html) for version compatibility
2. Use `most_recent = true` unless you need specific versions
3. Consider Terraform outputs to validate versions:
   ```hcl
   output "addon_versions" {
     value = module.eks.cluster_addons
   }
   ```

Would you like me to submit a PR to your repo with these fixes?