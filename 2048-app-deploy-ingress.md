# 2048 App

## Create Fargate profile

```
eksctl create fargateprofile \
    --cluster demo-cluster \
    --region us-east-1 \
    --name alb-sample-app \
    --namespace game-2048
```

## Deploy the deployment, service and Ingress

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml
```


<img width="1079" height="960" alt="Screenshot From 2026-10-13 05-43-08" src="https://github.com/user-attachments/assets/441b8fb2-08a6-45f7-b89c-dbde70d7afeb" />

