# EBS CSI Plugin configuration

The Amazon EBS CSI plugin requires IAM permissions to make calls to AWS APIs on your behalf.

Create an IAM role and attach a policy. AWS maintains an AWS managed policy or you can create your own custom policy. You can create an IAM role and attach the AWS managed policy with the following command. Replace my-cluster with the name of your cluster. The command deploys an AWS CloudFormation stack that creates an IAM role and attaches the IAM policy to it. 

```
eksctl create iamserviceaccount \
    --name ebs-csi-controller-sa \
    --namespace kube-system \
    --cluster <YOUR-CLUSTER-NAME> \
    --role-name AmazonEKS_EBS_CSI_DriverRole \
    --role-only \
    --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
    --approve
```

Run the following command. Replace <AWS-ACCOUNT-ID> with the name of your cluster, <AWS-ACCOUNT-ID> with your account ID.

```
eksctl create addon --name aws-ebs-csi-driver --cluster <YOUR-CLUSTER-NAME> --service-account-role-arn arn:aws:iam::<AWS-ACCOUNT-ID>:role/AmazonEKS_EBS_CSI_DriverRole --force
```

**Note**: If your cluster is in the AWS GovCloud (US-East) or AWS GovCloud (US-West) AWS Regions, then replace arn:aws: with arn:aws-us-gov:.

**References**:
https://repost.aws/knowledge-center/eks-persistent-storage
------------------------------------------------------------------------------------------
**IN MY CASE**
**ADDRESS NOT REFLECTING IN INGRESS resource after KUBECTL APPLY INGRESS**----------------------------------

**SOLUTION**

**Encountered IAM Role Issue** -->In my case 

4. IAM Role issue?
Ensure the IAM role associated with the AWS Load Balancer Controller has the correct policy attached.

You can check with:

kubectl describe serviceaccount aws-load-balancer-controller -n kube-system


**** Your AWS Load Balancer Controller is failing because of a missing IAM permission**:**


❌ Error Summary:

UnauthorizedOperation: You are not authorized to perform this operation. 
Action: ec2:CreateSecurityGroup 
Resource: VPC arn:aws:ec2:us-east-1:142000000:vpc/vpc-092d100000000

This means the IAM role AmazonEKSLoadBalancerControllerRole does not have the ec2:CreateSecurityGroup permission — which is required for the controller to set up ALB-related security groups.

**🔧 Steps to Fix**
1. Download AWS-recommended IAM policy
Run this command to get the latest recommended policy from AWS:

curl -o iam_policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json

2. Attach the policy to the IAM Role

If your IAM role is named AmazonEKSLoadBalancerControllerRole, use:

aws iam put-role-policy \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --policy-name AWSLoadBalancerControllerIAMPolicy \
  --policy-document file://iam_policy.json
📌 Note: Make sure your terminal is configured with an IAM user that has permission to update IAM roles (e.g., via aws configure or EC2 instance profile).



4. Re-deploy or restart the controller
You can restart the pod to retry:

kubectl delete pod -n kube-system -l app.kubernetes.io/name=aws-load-balancer-controller


NOW ADDRESS (**DNS name of the AWS Load Balancer (ALB)** ) REFLECTS IN INGRESS ,, U CAN ACCESS VIA BROWSER









