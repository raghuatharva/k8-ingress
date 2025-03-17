# Ingress Controller

OIDC provider:
Need of OIDC provider
Let's say you have an application running in a Kubernetes Pod that needs to read/write data to S3.

With OIDC Provider: ✅ You create an IAM role with S3 permissions and link it to a Kubernetes service account.
✅ Only Pods using that service account can access S3.

Without OIDC Provider: ❌ The IAM role can exist, but your Pod can’t assume it because AWS doesn’t trust Kubernetes identities without an OIDC provider.
❌ The only workaround is granting permissions to the entire EC2 node, which means all Pods running on that node get S3 access (even if they don’t need it).

```
eksctl utils associate-iam-oidc-provider \
    --region us-east-1 \
    --cluster expense \
    --approve
```



```
curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.10.0/docs/install/iam_policy.json
```

```
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam-policy.json
```

eksctl creates an IAM role behind the scenes.
It attaches the specified policy (AWSLoadBalancerControllerIAMPolicy) to that role.
Then, it associates the role with the Kubernetes service account (aws-load-balancer-controller).

Who Gets the IAM Role?
The IAM role is not attached to your IAM user. 🚫
It is linked to the Kubernetes service account (aws-load-balancer-controller) inside the kube-system namespace. ✅
Any Pod using the service account (aws-load-balancer-controller) in EKS can assume this role.

```
eksctl create iamserviceaccount \
--cluster=expense \
--namespace=kube-system \
--name=aws-load-balancer-controller \
--attach-policy-arn=arn:aws:iam::315069654700:policy/AWSLoadBalancerControllerIAMPolicy \
--override-existing-serviceaccounts \
--region us-east-1 \
--approve
```

```
helm repo add eks https://aws.github.io/eks-charts
```
This command installs the AWS Load Balancer Controller in your EKS cluster using Helm.

✅ The AWS Load Balancer Controller gets deployed in EKS.
✅ It uses the IAM service account (aws-load-balancer-controller) to get the required permissions.
✅ Now, Kubernetes Ingress and Service objects can automatically create ALBs and NLBs

```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=expense --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
```