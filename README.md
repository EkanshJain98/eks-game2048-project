# eks-game2048-project

## Deploying Game2048 application in AWS EKS

1. eksctl create cluster --name game-2048 --region ap-south-1 --fargate

![Pasted Graphic](https://github.com/user-attachments/assets/163e6006-45d8-4e5a-a718-5316043aba1e)

![Pasted Graphic 13](https://github.com/user-attachments/assets/39804d14-8c34-4b43-8ca4-ce877f5e8dce)

## Networking of VPC
![Pasted Graphic 18](https://github.com/user-attachments/assets/6b08aa98-12b6-468a-9862-3ab4cc80915a)

2. eksctl create fargateprofile --cluster game-2048 --region ap-south-1  --name game2046-fargateprofile --namespace game-2048

![Pasted Graphic 1](https://github.com/user-attachments/assets/38d5f59b-f3a4-4e4e-b261-816a7fabfbd4)

3. kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.8.0/docs/examples/2048/2048_full.yaml

```
namespace/game-2048 created
deployment.apps/deployment-2048 created
service/service-2048 created
ingress.networking.k8s.io/ingress-2048 created
```

4. kubectl get po -n game-2048

```
NAME                              READY   STATUS    RESTARTS   AGE
deployment-2048-bdbddc878-hj5bc   1/1     Running   0          105s
deployment-2048-bdbddc878-jvjwl   1/1     Running   0          105s
deployment-2048-bdbddc878-nl7lr   1/1     Running   0          105s
deployment-2048-bdbddc878-vrq58   1/1     Running   0          105s
deployment-2048-bdbddc878-x6kkw   1/1     Running   0          105s
```

5. curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.12.0/docs/install/iam_policy.json

```
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  8912  100  8912    0     0   515k      0 --:--:-- --:--:-- --:--:--  543k
```

6. aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicyForGame2048 --policy-document file://iam_policy.json

```
{
    "Policy": {
        "PolicyName": "AWSLoadBalancerControllerIAMPolicyForGame2048",
        "PolicyId": "ANPAZNKPEIHBJUHxxxxxx",
        "Arn": "arn:aws:iam::xxxxxxxxxxxx:policy/AWSLoadBalancerControllerIAMPolicyForGame2048",
        "Path": "/",
        "DefaultVersionId": "v1",
        "AttachmentCount": 0,
        "PermissionsBoundaryUsageCount": 0,
        "IsAttachable": true,
        "CreateDate": "2025-05-01T07:05:39+00:00",
        "UpdateDate": "2025-05-01T07:05:39+00:00"
    }
}
```

7.
oidc_id=$(aws eks describe-cluster --name game-2048 --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5)
aws iam list-open-id-connect-providers | grep $oidc_id | cut -d "/" -f4
eksctl utils associate-iam-oidc-provider --cluster game-2048 --approve

```
2025-05-01 07:10:15 [ℹ]  will create IAM Open ID Connect provider for cluster "game-2048" in "ap-south-1"
2025-05-01 07:10:16 [✔]  created IAM Open ID Connect provider for cluster "game-2048" in "ap-south-1"
```

8. eksctl create iamserviceaccount --cluster=game-2048 --namespace=kube-system --name=aws-load-balancer-controller-for-game2048 --attach-policy-arn=arn:aws:iam::xxxxxxxxxxxx:policy/AWSLoadBalancerControllerIAMPolicyForGame2048 --override-existing-serviceaccounts --region ap-south-1 --approve

```
2025-05-01 07:14:59 [ℹ]  1 iamserviceaccount (kube-system/aws-load-balancer-controller-for-game2048) was included (based on the include/exclude rules)
2025-05-01 07:14:59 [!]  metadata of serviceaccounts that exist in Kubernetes will be updated, as --override-existing-serviceaccounts was set
2025-05-01 07:14:59 [ℹ]  1 task: { 
    2 sequential sub-tasks: { 
        create IAM role for serviceaccount "kube-system/aws-load-balancer-controller-for-game2048",
        create serviceaccount "kube-system/aws-load-balancer-controller-for-game2048",
    } }2025-05-01 07:14:59 [ℹ]  building iamserviceaccount stack "eksctl-game-2048-addon-iamserviceaccount-kube-system-aws-load-balancer-controller-for-game2048"
2025-05-01 07:14:59 [ℹ]  deploying stack "eksctl-game-2048-addon-iamserviceaccount-kube-system-aws-load-balancer-controller-for-game2048"
2025-05-01 07:14:59 [ℹ]  waiting for CloudFormation stack "eksctl-game-2048-addon-iamserviceaccount-kube-system-aws-load-balancer-controller-for-game2048"
2025-05-01 07:15:29 [ℹ]  waiting for CloudFormation stack "eksctl-game-2048-addon-iamserviceaccount-kube-system-aws-load-balancer-controller-for-game2048"
2025-05-01 07:15:30 [ℹ]  created serviceaccount "kube-system/aws-load-balancer-controller-for-game2048"
```

9. helm repo add eks https://aws.github.io/eks-charts

```
"eks" already exists with the same configuration, skipping
```

10. helm repo update eks

```
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "eks" chart repository
Update Complete. ⎈Happy Helming!⎈
```

11. helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=game-2048 --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller-for-game2048 --set region=ap-south-1 --set vpcId=vpc-0ec00a061e60xxxxx

```
NAME: aws-load-balancer-controller
LAST DEPLOYED: Thu May  1 07:19:21 2025
NAMESPACE: kube-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
AWS Load Balancer controller installed!
```

12. kubectl get po -n kube-system

```
NAME                                           READY   STATUS    RESTARTS   AGE
aws-load-balancer-controller-5c875989b-4vg4q   0/1     Running   0          51s
aws-load-balancer-controller-5c875989b-cqs86   0/1     Running   0          51s
.
.
```

13. kubectl get ingress -n game-2048

```
NAME           CLASS   HOSTS   ADDRESS                                                                    PORTS   AGE
ingress-2048   alb     *       k8s-game2048-ingress2-170f34f9e5-181944xxxx.ap-south-1.elb.amazonaws.com   80      22m
```

## How traffic flows from ALB to Pods

![Pasted Graphic 15](https://github.com/user-attachments/assets/3d4dd486-288c-46ed-9bf2-f54bebc9e0b6)

## Hit the DNS of ALB to access the application deployed in pods.

![Pasted Graphic 16](https://github.com/user-attachments/assets/a4096cc4-ac6e-482a-9c65-090c4837d898)

## Play the game (Game-2048)
![Pasted Graphic 17](https://github.com/user-attachments/assets/f6e8f653-5283-41f1-b621-7227a8b786e8)
