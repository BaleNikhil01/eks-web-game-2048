
## To configure an IAM OIDC provider run the below command

```
eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve
```
This links your AWS account's IAM with your Kubernetes cluster so your pods can securely access AWS services (like S3) without needing hardcoded AWS keys.


K8s and AWS are two separate systems, and to let pods access AWS resources with scoped permissions, you link them via IRSA. Every EKS cluster has one fixed OIDC URL, which you register with AWS as an IAM OIDC Provider so AWS trusts tokens signed by that cluster. At runtime, a pod running as a ServiceAccount (e.g. `s3-reader`) gets a JWT signed by the K8s API server, sends it to STS, which fetches the public key from the OIDC URL to verify the signature (confirming the token is from *your* cluster and not forged), then checks the IAM Role's trust policy to confirm it's the right service account in the right namespace, and finally returns temporary AWS credentials that last about an hourto create the desired resources(alb in this case).


