# cluster-template-charts
Helm Charts for use with the [cluster-templates-operator].
Refer to the Operator's documentation for more information.

## Charts

### hypershift-aws-template

#### Prerequisites

To use this template you must have:

1. Met all the [Hypershift prerequisites].

2. Hypershift installed on a Kubernetes/OpenShift cluster. This can be accomplished using one of the following methods:
    - [Install Multicluster Engine Operator] (MCE) 2.5+ (preferred)
    - [Install Hypershift Operator]

    If using MCE, create the OIDC provider secret with the S3 bucket from the prior step. E.g.

    ```shell
    kubectl create secret generic hypershift-operator-oidc-provider-s3-credentials \
      --from-file=credentials=$HOME/.aws/credentials \
      --from-literal=bucket=<s3-bucket-name> \
      --from-literal=region=<s3-bucket-region> \
      --namespace local-cluster
    ```

3. Created the `cluster-provisioner` service account and RBAC policies in the current namespace.

    ```shell
    kubectl create -f k8s/cluster-provisioner.yaml
    ```

4. Generated a Role ARN for use with the hypershift CLI. E.g.

    ```shell
    hypershift create iam cli-role \
      --aws-creds "$HOME/.aws/credentials" \
      --name "$USER-hypershift-cli-role" \
      --additional-tags="expirationDate=$(date -d '1 year' --iso=minutes --utc)"
    ```

5. Created the `hypershift` secret in the current namespace. E.g.

    ```shell
    kubectl create secret generic hypershift \
      --from-file=aws-credentials=$HOME/.aws/credentials \
      --from-file=pull-secret=<path-to-ocp-pull-secret>
    ```

#### Usage

To create a cluster (and infra/iam):

```shell
export VERSION="4.16.11"
export BASE_DOMAIN=<route53-domain>
export ROLE_ARN=<hypershift-cli-role-arn>

helm install \
  --wait \
  --wait-for-jobs \
  --timeout 20m \
  --set "version=$VERSION,baseDomain=$BASE_DOMAIN,hypershiftRoleArn=$ROLE_ARN" \
  "$USER-cluster" \
  ./charts/hypershift-aws-template
```

To destroy a cluster (and infra/iam):

```shell
helm uninstall --timeout 20m "$USER-cluster"
```

## Releases

A GitHub Actions [workflow](.github/workflows/release.yaml) is used to automatically release new
charts upon merge to the `main` branch.

The chart repository is hosted with GitHub Pages and located at
https://konflux-ci.dev/cluster-template-charts.

[cluster-templates-operator]: https://github.com/stolostron/cluster-templates-operator/
[Hypershift prerequisites]: https://hypershift-docs.netlify.app/getting-started#prerequisites
[Install Multicluster Engine Operator]: https://docs.redhat.com/en/documentation/red_hat_advanced_cluster_management_for_kubernetes/2.9/html/clusters/cluster_mce_overview#mce-install-intro
[Install Hypershift Operator]: https://hypershift-docs.netlify.app/getting-started/#install-hypershift-operator
