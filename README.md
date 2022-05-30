1. Create an IAM policy using the policy

```
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
```

2. Create an IAM role. Create a Kubernetes service account named aws-load-balancer-controller in the kube-system namespace for the AWS Load Balancer Controller and annotate the Kubernetes service account with the name of the IAM role.

```
aws eks describe-cluster --name my-cluster --query "cluster.identity.oidc.issuer" --output text
```

to view OIDC provider URL

3. in the load-balancer-role-trust-policy.json. Replace 111122223333 with your account ID. Replace region-code and Replace EXAMPLED539D4633E53DE1B71EXAMPLE with the output returned in the OIDC provider URL after /id/

4 create the IAM role

```
aws iam create-role \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --assume-role-policy-document file://"load-balancer-role-trust-policy.json"
```

5 Attach the required Amazon EKS managed IAM policy to the IAM role. Replace 111122223333 with your account ID.

```
aws iam attach-role-policy \
  --policy-arn arn:aws:iam::111122223333:policy/AWSLoadBalancerControllerIAMPolicy \
  --role-name AmazonEKSLoadBalancerControllerRole
```

6. in the aws-load-balancer-controller-service-account.yaml, replacing 111122223333 with your account ID and apply it to the cluster

```
kubectl apply -f aws-load-balancer-controller-service-account.yaml
```

7. use helm to install load balancer controller

```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=cluster-name \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller
```

8. verify the load balancer is running

```
kubectl get deployment -n kube-system aws-load-balancer-controller
```

9. apply the nginx-ingress controller to the cluster

```
kubectl apply -f nginxcontroller.yml
```

10. apply deployments and ingress resources to the cluster

```
kubectl apply -f echo1.yml
kubectl apply -f echo2.yml
```
