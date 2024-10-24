---
title: "Introduction"
sidebar_position: 3
---

Each ACK service controller is packaged into a separate container image that is published in a public repository corresponding to an individual ACK service controller. For each AWS service that we wish to provision, resources for the corresponding controller must be installed in the Amazon EKS cluster. Helm charts and official container images for ACK are available [here](https://gallery.ecr.aws/aws-controllers-k8s).

In this section, since we will be working with Amazon DynamoDB ACK, we need to install the ACK controller first by using helm chart.
The first step is to create IAM role that allows to create a DynamoDB table:
```bash
$ aws iam create-policy \
    --policy-name eks-workshop-create-dynamo \
    --policy-document \
'{
    "Statement": [
        {
            "Action": "dynamodb:*",
            "Effect": "Allow",
            "Resource": [
                "arn:aws:dynamodb:us-west-2:511816149858:*"
            ],
            "Sid": "AllAPIActionsOnCart"
        }
    ],
    "Version": "2012-10-17"
}'

```
Then we will create IRSA for the ACK cntroller to use: 
```bash
$ export DDB_ACK_POLICY_ARN=$(aws iam list-policies --query 'Policies[?PolicyName==`eks-workshop-create-dynamo`].Arn' --output text)

$ eksctl create iamserviceaccount --name ack-dynamodb \
  --namespace ack-dynamodb --cluster $EKS_CLUSTER_NAME \
  --role-name ${EKS_CLUSTER_NAME}-carts-ack \
  --attach-policy-arn $DDB_ACK_POLICY_ARN --approve
```
Now the DynamoDB ACK controller can be installed by using the following commend: 
```bash
$ aws ecr-public get-login-password --region us-east-1 | helm registry login --username AWS --password-stdin public.ecr.aws
$ helm install -n ack-dynamodb ack-dynamodb \
  oci://public.ecr.aws/aws-controllers-k8s/dynamodb-chart \
  --version=$1.1.1  \
  --set=aws.region=$AWS_REGION \
  --set serviceAccount.create=false \
  --set serviceAccount.name=ack-dynamodb
```

Once the controller is installed, it is running as a deployment in its own Kubernetes namespace. To see what's under the hood, lets run the below.

```bash
$ kubectl describe deployment ack-dynamodb -n ack-dynamodb
```
