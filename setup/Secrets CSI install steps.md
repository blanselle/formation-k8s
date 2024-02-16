# Installation

```sh
helm repo add secrets-store-csi-driver https://kubernetes-sigs.github.io/secrets-store-csi-driver/charts
helm repo add aws-secrets-manager https://aws.github.io/secrets-store-csi-driver-provider-aws
helm repo update

helm install -n kube-system csi-secrets-store secrets-store-csi-driver/secrets-store-csi-driver --set syncSecret.enabled=true
helm install -n kube-system secrets-provider-aws aws-secrets-manager/secrets-store-csi-driver-provider-aws
```

# IAM Service Acount for EKS

Policy below created to allow to read every secrets in the account from EKS (useful for training purposes, restrict more for PROD)

```sh
AWS_POLICY_ARN=$(aws --profile formation-k8s --region "$AWS_REGION" --query Policy.Arn --output text iam create-policy --policy-name EKS_SecretsManager_Read_Policy --policy-document '{
    "Version": "2012-10-17",
    "Statement": [ {
        "Effect": "Allow",
        "Action": ["secretsmanager:GetSecretValue", "secretsmanager:DescribeSecret"],
        "Resource": ["arn:aws:secretsmanager:eu-west-1:562414170838:secret:*"]
    } ]
}')
```

Create the IAM role, granting the Kubernetes service account the AssumeRoleWithWebIdentity action. Copy the content below to a file named `aws-ssm-csi-driver-trust-policy.json`
Replace `111122223333` with your account ID. Replace `EXAMPLED539D4633E53DE1B71EXAMPLE` and `region-code` with the values returned in the previous step.

⚠️ In this example we create a trust relationship with every Kubernetes Service Account named `secrets-store-csi-sa` in our cluster, because the namespace is set to a wildcard. In a production scenario, you should restrict to the proper namespace an provision specific roles to each SA in you K8S Cluster.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::111122223333:oidc-provider/oidc.eks.region-code.amazonaws.com/id/EXAMPLED539D4633E53DE1B71EXAMPLE"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringLike": {
          "oidc.eks.region-code.amazonaws.com/id/EXAMPLED539D4633E53DE1B71EXAMPLE:sub": "system:serviceaccount:*:secrets-store-csi-sa",
          "oidc.eks.region-code.amazonaws.com/id/EXAMPLED539D4633E53DE1B71EXAMPLE:aud": "sts.amazonaws.com"
        }
      }
    }
  ]
}
```

Create the role using the trust policy.

```sh
aws iam create-role --profile formation-k8s \
  --role-name EKS_SSM_CSI_DriverRole \
  --assume-role-policy-document file://"aws-ssm-csi-driver-trust-policy.json"
```

Attach the permissions policy to the role with the following command.

```sh
aws iam attach-role-policy --profile formation-k8s \
  --policy-arn "$AWS_POLICY_ARN" \
  --role-name EKS_SSM_CSI_DriverRole
```

Create the service account in K8S for this role. Create a file named `secrets-store-csi-sa.yaml` and replace the ARN with the one you just created and the namespace with the one you will be using to host your pods.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    app.kubernetes.io/name: secrets-store-csi-driver-provider-aws
  name: secrets-store-csi-sa
  namespace: FIXME
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::111122223333:role/EKS_SSM_CSI_DriverRole
```

Then apply it to the cluster

```sh
kubectl apply -f secrets-store-csi-sa.yaml -n kube-system
```



# Usage

```sh
AWS_REGION=eu-west-1
AWS_CLUSTERNAME=formation-k8s
AWS_PROFILE=participant-formation-k8s
AWS_SECRET_NAME=MySecret

aws --profile "$AWS_PROFILE" --region "$AWS_REGION" secretsmanager  create-secret --name "$AWS_SECRET_NAME" --secret-string '{"username":"memeuser", "password":"hunter2"}'

AWS_SECRET_ARN=$(aws --profile "$AWS_PROFILE" --region "$AWS_REGION" secretsmanager  describe-secret --secret-id "$AWS_SECRET_NAME" --query "ARN" --output text)

echo "apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: test-secret
spec:
  provider: aws
  parameters:
    objects: |
        - objectName: \"$AWS_SECRET_ARN\"
          objectType: \"secretsmanager\"" > test-secret-provider-class.yaml
```

See detailed example in [test_secrets_full.yaml](file://test_secrets_full.yaml)



