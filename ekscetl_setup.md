## EKS Cluster and AWS Load Balancer Controller Setup

Follow these steps to set up your EKS cluster and install the AWS Load Balancer Controller:

1. **Create EKS Cluster**
    ```sh
    eksctl create cluster \
      --name pranav-cluster \
      --region us-east-1 \
      --nodegroup-name pranav-node-group \
      --node-type t3.medium \
      --nodes 2 \
      --nodes-min 1 \
      --nodes-max 3 \
      --managed \
      --node-private-networking
    ```

2. **Associate IAM OIDC Provider**
    ```sh
    eksctl utils associate-iam-oidc-provider \
      --region us-east-1 \
      --cluster pranav-cluster \
      --approve
    ```

3. **Create IAM Service Account**
    ```sh
    eksctl create iamserviceaccount \
      --cluster pranav-cluster \
      --namespace kube-system \
      --name aws-load-balancer-controller \
      --attach-policy-arn arn:aws:iam::543816070942:policy/AWSLoadBalancerControllerIAMPolicy_Pranav \
      --approve
    ```

4. **Add EKS Helm Repository**
    ```sh
    helm repo add eks https://aws.github.io/eks-charts
    ```

5. **Update Helm Repositories**
    ```sh
    helm repo update
    ```

6. **Install AWS Load Balancer Controller**
    ```sh
    helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
      -n kube-system \
      --set clusterName=pranav-cluster \
      --set serviceAccount.create=false \
      --set serviceAccount.name=aws-load-balancer-controller
    ```
**optional**
With Optional Arguments
```sh
  helm install aws-load-balancer-controller eks/aws-load-balancer-controller 
  -n kube-system --set clusterName=pranav-cluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller --set region=us-east-1 --set vpcId=$(aws eks describe-cluster --name my-cluster --region us-west-2 --query "cluster.resourcesVpcConfig.vpcId"--output text)
```
