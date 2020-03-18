# Main Project

## Terraform data sources and resources

- Provider: [`Google`](https://www.terraform.io/docs/providers/google/index.html "Provider: Google")
  - Data Sources:
    - [`google_billing_account`](https://www.terraform.io/docs/providers/google/d/google_billing_account.html)
  - Resources:
    - [`google_project`](https://www.terraform.io/docs/providers/google/r/google_project.html "Resource: Google Project")
    - [`google_project_service`](https://www.terraform.io/docs/providers/google/r/google_project_service.html "Resource: Google Project Service")
    - [`google_storage_bucket`](https://www.terraform.io/docs/providers/google/r/storage_bucket.html "Resource: Google Storage Bucket")
    - [`google_dns_managed_zone`](https://www.terraform.io/docs/providers/google/r/dns_managed_zone.html "Resource: Google DNS Managed Zone")  
    - [`google_bigquery_dataset`](https://www.terraform.io/docs/providers/google/r/bigquery_dataset.html "Resource: Google BigQuery DataSet")
    - [`google_logging_billing_account_sink`](https://www.terraform.io/docs/providers/google/r/logging_billing_account_sink.html "Resource: Google Logging Billing Account Sink")
    - [`google_project_iam_binding`](https://www.terraform.io/docs/providers/google/r/google_project_iam.html "Resource: Google Project IAM Binding")
    - [`google_project_iam_member`](https://www.terraform.io/docs/providers/google/r/google_project_iam.html "Resource: Google Project IAM Member")[<sup>G2</sup>](#global-reference)
    - [`google_project_iam_custom_role`](https://www.terraform.io/docs/providers/google/r/google_project_iam_custom_role.html "Resource: Google IAM Custom Role")
    - [`google_storage_bucket_iam_binding`](https://www.terraform.io/docs/providers/google/r/storage_bucket_iam.html "Resource: Google Storage Bucket IAM Binding")
    - [`google_storage_bucket_iam_member`](https://www.terraform.io/docs/providers/google/r/storage_bucket_iam.html "Resource: Google Storage Bucket IAM Member")[<sup>G2</sup>](#global-reference)

## Components

- Project:
  - `kubernetes-public`
- API:
  - `compute`
  - `logging`
  - `monitoring`
  - `bigquery-json`
  - `container`
  - `storage-component`
  - `oslogin`
  - `dns`
- GCS Bucket:
  - `gs://k8s-infra-clusters-terraform`:
    - bucketpolicyonly: `true`
    - location: `us`
- IAM Policy Binding:
  - `roles/bigquery.admin`:
    - `group:k8s-infra-bigquery-admins@kubernetes.io`
  - `roles/compute.viewer`:
    - `group:k8s-infra-cluster-admins@kubernetes.io`
  - `roles/container.admin`:
    - `group:k8s-infra-cluster-admins@kubernetes.io`
  - `projects/kubernetes-public/roles/ServiceAccountLister`:
    - `group:k8s-infra-cluster-admins@kubernetes.io`
  - `roles/container.clusterViewer`:
    - `group:gke-security-groups@kubernetes.io`
  - `roles/bigquery.jobUser`:
    - `group:k8s-infra-gcp-accounting@kubernetes.io`
  - `roles/dns.admin`:
    - `group:k8s-infra-dns-admins@kubernetes.io`
- IAM Role (Project IAM Custom Role):
  - `ServiceAccountLister`:
    - title: `Service Account Lister`
    - description: `Can list ServiceAccounts.`
    - stage: `GA`
    - permissions:
      - `iam.serviceAccounts.list`
- IAM:
  - `gs://k8s-infra-clusters-terraform`:
    - `group:k8s-infra-cluster-admins@kubernetes.io:objectAdmin`
    - `group:k8s-infra-cluster-admins@kubernetes.io:legacyBucketOwner`
- DNS Managed Zone:
  - `k8s-io` : `k8s.io`
  - `kubernetes-io` : `kubernetes.io`
  - `x-k8s-io` : `x-k8s.io`
  - `k8s-e2e-com` : `k8s-e2e.com`
  - `canary-k8s-io` : `canary.k8s.io`
  - `canary-kubernetes-io` : `canary.kubernetes.io`
  - `canary-x-k8s-io` : `canary.x-k8s.io`
  - `canary-k8s-e2e-com` : `canary.k8s-e2e.com`
- Bigquery DataSet:

  > [**@bartsmykla**]: Currently [at the end of the script which is provisioning resources for main project](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/ensure-main-project.sh#L179-L192) there is a comment and mechanism to acknowledging the knowledge about having to log in to the cloud console by human to enable billing export
  
  - `kubernetes_public_billing`:
    - `access`:
      - `groupByEmail:k8s-infra-gcp-accounting@kubernetes.io`:
        - `READER`
        - `roles/bigquery.metadataViewer`
        - `roles/bigquery.user`

## Yaml representation of Components[<sup>G1</sup>](../README.md#global-reference)<sup>,</sup>[<sup>G2</sup>](../README.md#global-reference)

```yaml
google_project:
  - name: kubernetes-public
google_project_service:
  - service: compute.googleapis.com
    project: kubernetes-public
  - service: logging.googleapis.com
    project: kubernetes-public
  - service: monitoring.googleapis.com
    project: kubernetes-public
  - service: bigquery-json.googleapis.com
    project: kubernetes-public
  - service: container.googleapis.com
    project: kubernetes-public
  - service: storage-component.googleapis.com
    project: kubernetes-public
  - service: oslogin.googleapis.com
    project: kubernetes-public
  - service: dns.googleapis.com
    project: kubernetes-public
google_storage_bucket:
  - name: k8s-infra-clusters-terraform
    bucket_policy_only: true
    location: us
    project: kubernetes-public
google_dns_managed_zone:
  - name: k8s-io
    dns_name: k8s.io.
    project: kubernetes-public
  - name: kubernetes-io
    dns_name: kubernetes.io.
    project: kubernetes-public
  - name: x-k8s-io
    dns_name: x-k8s.io.
    project: kubernetes-public
  - name: k8s-e2e-com
    dns_name: k8s-e2e.com.
    project: kubernetes-public
  - name: canary-k8s-io
    dns_name: canary.k8s.io.
    project: kubernetes-public
  - name: canary-kubernetes-io
    dns_name: canary.kubernetes.io.
    project: kubernetes-public
  - name: canary-x-k8s-io
    dns_name: canary.x-k8s.io.
    project: kubernetes-public
  - name: canary-k8s-e2e-com
    dns_name: canary.k8s-e2e.com.
    project: kubernetes-public
google_bigquery_dataset:
  - dataset_id: kubernetes_public_billing
    access:
      - role: READER
        group_by_email: k8s-infra-gcp-accounting@kubernetes.io
      - role: roles/bigquery.metadataViewer
        group_by_email: k8s-infra-gcp-accounting@kubernetes.io
      - role: roles/bigquery.user
        group_by_email: k8s-infra-gcp-accounting@kubernetes.io
      # https://cloud.google.com/logging/docs/api/tasks/exporting-logs#writing_to_the_destination
      - role: roles/bigquery.dataEditor
        # serviceAccount email will be taken from "google_logging_billing_account_sink" resource's
        # attributes reference
        user_by_email: "[google_logging_billing_account_sink.kubernetes_public_billing_sink.writer_identity]"
    delete_contents_on_destroy: false
    project: kubernetes-public
google_logging_billing_account_sink:
# To be able to create/manage this resource, the credentials
# used with Terraform has to have granted IAM role: "roles/logging.configWriter"
# Using this resource swhould remove the need of manual steps which needs to be done:
# https://github.com/kubernetes/k8s.io/blob/e62c18e79a75615d4868afaf5eebcf36bb265df9/infra/gcp/ensure-main-project.sh#L179-L192
  - name: kubernetes_public_billing_sink
    billing_account: 018801-93540E-22A20E
    destination: bigquery.googleapis.com/projects/kubernetes-public/datasets/kubernetes_public_billing
google_project_iam_custom_role:
  - role_id: ServiceAccountLister
    title: Can list ServiceAccounts.
    permissions:
      - iam.serviceAccounts.list
    stage: GA
    project: kubernetes-public
google_project_iam_binding:
  - role: roles/bigquery.admin
    members:
      - group:k8s-infra-bigquery-admins@kubernetes.io
    project: kubernetes-public
  - role: roles/compute.viewer
    members:
      - group:k8s-infra-cluster-admins@kubernetes.io
    project: kubernetes-public
  - role: roles/container.admin
    members:
      - group:k8s-infra-cluster-admins@kubernetes.io
    project: kubernetes-public
  - role: roles/container.clusterViewer
    members:
      - group:gke-security-groups@kubernetes.io
    project: kubernetes-public
  - role: roles/bigquery.jobUser
    members:
      - group:k8s-infra-gcp-accounting@kubernetes.io
    project: kubernetes-public
  - role: roles/dns.admin
    members:
      - group:k8s-infra-dns-admins@kubernetes.io
    project: kubernetes-public
  - role: projects/kubernetes-public/roles/ServiceAccountLister
    members:
      - group:k8s-infra-cluster-admins@kubernetes.io
    project: kubernetes-public
google_storage_bucket_iam_binding:
  - role: roles/storage.objectAdmin
    members:
      - group:k8s-infra-cluster-admins@kubernetes.io
    bucket: gs://k8s-infra-clusters-terraform
  - role: roles/storage.legacyBucketOwner
    members:
      - group:k8s-infra-cluster-admins@kubernetes.io
    bucket: gs://k8s-infra-clusters-terraform
```
