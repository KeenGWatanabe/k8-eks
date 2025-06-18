1. **Run backend S3**
  https://github.com/KeenGWatanabe/tf-backend

2. **Run tf-eks***
  https://github.com/KeenGWatanabe/tf-eks2
  tf-eks2 > eks.tf > `ln15 - name` node grp
  Pull out the `oidc arn and input into service-account.yaml ln6`


3. **Build & Push Docker Image**

/main use `taskmgr-eks` > use backend > tf-eks2 > alb by helm resource
/dev use `tf-secretsAPP`
/secrets use `secrets`
/module use > backend > vpc > tf-eks2 > alb by kubectl

- Use the same Docker image from `taskmgr repo` build ECR `image_uri` and push
into `deployment.yaml ln18 `

4.   **Setup EKS cluster and config**
have to create IAM policy on EKS console
![add IAM access entry](/images/IAMaccess.png)
Choose AmazonEKSClusterAdminPolicy here
![EKS_IAM](/README_FILES/CREATE_EKS_IAM_POLICY.md)

aws eks list-clusters
aws eks update-kubeconfig --name thunder-eks-cluster --region us-east-1


kubectl create namespace thunder-eks-app


# install controller
helm repo add eks https://aws.github.io/eks-charts
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=thunder-eks-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller

# alb reinstall if fails
helm upgrade -i aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=custom-eks-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller  

5. **Apply Kubernetes Manifests**  
   ```sh
# First: Infra components 
> update eks.amazonaws.com/role-arn in service-account.yaml ln7, source data from tf-eks2 output.
```bash
kubectl apply -f service-account.yaml   
kubectl apply -f configmap.yaml
kubectl apply -f app-secrets.yaml
```
# Second: Your Helm ALB Terraform repo
Just ensure the cluster_name in tf-helm-alb matches your EKS cluster name exactly.
cd ../tf-helm-alb  
```bash
terraform init
terraform apply    # Deploys AWS Load Balancer Controller
```
# Third: Verify ALB controller is running
kubectl get pods -n kube-system | grep aws-load-balancer
# Wait until pods show "Running" (1-2 mins)
# Verify webhook service is ready
kubectl get svc -n kube-system aws-load-balancer-webhook-service
# Test with a simpe Service (optional)
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port=80 --type=LoadBalancer
# Should auto-create an AWS ALB

# Fourth: Core app components 
```bash
kubectl apply -f deployment.yaml
# Lastly: Networking components that reference the app
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml
```

  

   ```
5a. **Verify**  
   ```sh
   kubectl get pods
   kubectl get ingress  # Check ALB URL
   ```
# Force restart pods
kubectl delete pod -l app=my-app 
kubectl get pods -w  


![EKScontroller](/images/EKScontroller.png) 



# cluster   
![EKScluster](/images/EKScluster.png)
# nodes
![EKSnodes](/images/EKSnodes.png)
# IAM roles
![EKSiam](/images/EKSiam.png)
# pods
![EKSpods](/images/EKSpods.png)
# pods events
![EKSpodsEvents](/images/EKSevents.png)
# secrets file added
![EKSevents2](/images/EKSevents2.png)

# kubectl get svc
![EKSsvc](/images/EKSsvc.png)

---------------------------------------------------------------------------------------
