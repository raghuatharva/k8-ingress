# Ingress Controller
<!-- Added detailed steps for installing the ingress controller -->

<!-- Step 1: Associate the IAM OIDC provider with the EKS Cluster -->
```bash
# Associates IAM OIDC provider with the EKS cluster (required for IRSA)
eksctl utils associate-iam-oidc-provider \
    --region us-east-1 \
    --cluster expense \
    --approve
```

<!-- Step 2: Download the IAM policy JSON file -->
```bash
# Downloads the IAM policy file needed for the AWS Load Balancer Controller setup.
curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.10.0/docs/install/iam_policy.json
```

<!-- Step 3: Create the AWS IAM policy -->
```bash
# Creates the IAM policy using the downloaded iam-policy.json.
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam-policy.json
```

<!-- Step 4: Begin creating the IAM service account for the controller -->
```bash
# Begins creation of the IAM service account (partial command – add additional parameters as needed).
eksctl create iamserviceaccount \
--cluster=expense \
--namespace=kube-system \
--name=aws-load-balancer-controller \
--attach-policy-arn=arn:aws:iam::315069654700:policy/AWSLoadBalancerControllerIAMPolicy \
--override-existing-serviceaccounts \
--region us-east-1 \
--approve
```

<!-- Step 5: Add the EKS Helm chart repository -->
```bash
# Adds the EKS Helm chart repository.
helm repo add eks https://aws.github.io/eks-charts
```

<!-- Step 6: Install the AWS Load Balancer Controller using Helm -->
```bash
# Installs the AWS Load Balancer Controller using Helm.
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=expense --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
```