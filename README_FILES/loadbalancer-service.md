apiVersion: v1
kind: Service
metadata:
  name: taskmgr-loadbalancer-service
  namespace: taskmgr-eks-app
spec:
  type: LoadBalancer
  selector:
    app: app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000