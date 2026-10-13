# How to setup alb add on

Download IAM policy

```
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json
```

Create IAM Policy

```
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
```
Reads iam_policy.json from your local filesystem
Creates a managed policy (AWSLoadBalancerControllerIAMPolicy) in IAM with that iam_policy.json document as its content

Create IAM Role

```
eksctl create iamserviceaccount \
  --cluster=<your-cluster-name> \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```

| Flag | What it does |
|---|---|
| `--cluster` | Which EKS cluster this SA belongs to |
| `--namespace=kube-system` | Creates the SA in `kube-system` (where the controller pod runs) |
| `--name` | The **K8s ServiceAccount** name — pods in this namespace can use `aws-load-balancer-controller service` |
| `--role-name` | The name of the **IAM Role** to be create (the role the SA will assume via IRSA) |
| `--attach-policy-arn` | Attaches the policy you just created (`AWSLoadBalancerControllerIAMPolicy`) to that role |
| `--approve` | Skip the interactive "are you sure?" prompt |

## Deploy ALB controller

Add helm repo

```
helm repo add eks https://aws.github.io/eks-charts
```

What it does
1.Downloads the index.yaml from that URL (a manifest of all available charts + versions) 
2.Saves it locally so Helm knows what charts exist there

Update the repo

```
helm repo update eks
```

Install

```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system \
  --set clusterName=<your-cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=<your-region> \
  --set vpcId=<your-vpc-id>
```

Full flow after this command
Ingress resource created in K8s
  → Controller pod (in kube-system) watches and detects it
  → Pod uses IRSA to get AWS creds
  → Calls elbv2:CreateLoadBalancer, creates target group, registers targets
  → ALB is now live, routing traffic to your pods
Verify that the deployments are running.

```
kubectl get deployment -n kube-system aws-load-balancer-controller
```

<img width="852" height="106" alt="Screenshot From 2026-10-13 06-46-57" src="https://github.com/user-attachments/assets/746e9db3-ced4-4fa7-bad5-26a12b84519f" />


<img width="860" height="171" alt="Screenshot From 2026-10-13 06-49-18" src="https://github.com/user-attachments/assets/9a871d19-7a14-4689-aec8-0d2629e4b805" />

<img width="1583" height="525" alt="Screenshot From 2026-10-13 05-43-57" src="https://github.com/user-attachments/assets/cf450ccd-f108-4084-a5d3-72177f4564bf" />
Active ALB configured by the AWS controller
