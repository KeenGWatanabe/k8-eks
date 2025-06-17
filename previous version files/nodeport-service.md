apiVersion: v1
kind: Service
metadata:
  name: taskmgr-nodeport-service
  namespace: taskmgr-eks-app
spec:
  type: NodePort
  selector:
    app: app
  ports:
    - protocol: TCP
      port: 30001
      targetPort: 5000