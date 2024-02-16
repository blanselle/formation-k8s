# EKS Cluster installation and configuration

1. Create the cluster https://docs.aws.amazon.com/eks/latest/userguide/getting-started-console.html
2. Create an IAM OIDC Provider https://docs.aws.amazon.com/eks/latest/userguide/enable-iam-roles-for-service-accounts.html
3. Install the ALB CNI Add-on https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html
4. How to use ingresses https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html
5. Create a hosted zone on formation-k8s.widop.com
6. Install the EFS CSI Add-on https://docs.aws.amazon.com/eks/latest/userguide/efs-csi.html
7. Create an EFS https://github.com/kubernetes-sigs/aws-efs-csi-driver/blob/master/docs/efs-create-filesystem.md
8. Create a StorageClass https://github.com/kubernetes-sigs/aws-efs-csi-driver/blob/master/examples/kubernetes/dynamic_provisioning/README.md
9. Install Metrics Server for HPAs https://docs.aws.amazon.com/eks/latest/userguide/metrics-server.html
10. Installation du AWS Secrets and Configuration Provider (ASCP) Follow *Secrets CSI install steps.md* (base doc https://docs.aws.amazon.com/secretsmanager/latest/userguide/integrating_csi_driver.html)
11. Sync AWS Secrets into K8S Secrets https://www.arthurkoziel.com/sync-aws-secrets-manager-to-k8s-secrets/
12. Allow other  IAM Identities to use Kubectl on the cluster https://repost.aws/knowledge-center/eks-api-server-unauthorized-error 

# Give trainees access to the cluster

Install AWS ClI : https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

Configure access to the AWS Account (API Key/Secret givent during the training).
```shell
aws configure --profile participant-formation-k8s
```

Kubeconfig for trainees
```shell
aws eks update-kubeconfig --region eu-west-1 --name formation-k8s --profile participant-formation-k8s
```

