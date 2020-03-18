# Infrastructure tools refactoring <!-- omit in toc -->

- [Preface](#preface)
  - [tl;dr](#tldr)
  - [How to read this document](#how-to-read-this-document)
  - [Why I created this document](#why-i-created-this-document)
  - [What problem are we trying to solve](#what-problem-are-we-trying-to-solve)
- [Current state of the tools](#current-state-of-the-tools)
  - [Flow chart](#flow-chart)
  - [Components list](#components-list)
  - [Components diagram](#components-diagram)
  - [Analysis](#analysis)
    - [Main Project](#main-project)
      - [Terraform data sources and resources for Main Project](#terraform-data-sources-and-resources-for-main-project)
      - [Components](#components)
      - [Yaml representation of *Main Project Components*<sup>G1</sup><sup>,</sup>[<sup>G2</sup>](#global-reference)](#yaml-representation-of-main-project-componentssupg1supsupsupsupg2sup)
    - [Prod Storage](#prod-storage)
      - [Terraform resources for Prod Storage](#terraform-resources-for-prod-storage)
      - [Prod Storage `[PROJECTS]`](#prod-storage-projects)
      - [Prod Storage `[REGIONS]`](#prod-storage-regions)
      - [Prod Storage / Components](#prod-storage--components)
      - [Yaml representation of *Prod Storage*<sup>G1</sup><sup>,</sup>[<sup>G2</sup>](#global-reference)](#yaml-representation-of-prod-storagesupg1supsupsupsupg2sup)
      - [Notes for Prod Storage](#notes-for-prod-storage)
      - [Reference for Prod Storage](#reference-for-prod-storage)
    - [Prod Storage GCLB](#prod-storage-gclb)
      - [Terraform resources for Prod Storage GCLB](#terraform-resources-for-prod-storage-gclb)
      - [Prod Storage GCLB / Components](#prod-storage-gclb--components)
      - [Yaml representation of *Prod Storage GCLB*<sup>G1</sup>](#yaml-representation-of-prod-storage-gclbsupg1sup)
      - [Reference for Prod Storage GCLB](#reference-for-prod-storage-gclb)
    - [Staging Storage](#staging-storage)
      - [Terraform resources for Staging Storage](#terraform-resources-for-staging-storage)
      - [Staging Storage `[PROJECTS]`](#staging-storage-projects)
      - [Staging Storage `[RS_PROJECTS]`](#staging-storage-rs_projects)
      - [Staging Storage / Components](#staging-storage--components)
      - [Yaml representation of *Staging Storage*<sup>G1</sup>,</sup>[<sup>G2</sup>](#global-reference)](#yaml-representation-of-staging-storagesupg1supsupsupg2sup)
    - [Conformance Storage](#conformance-storage)
      - [Terraform resources for Conformance Storage](#terraform-resources-for-conformance-storage)
      - [Conformance Storage `[BUCKETS]`](#conformance-storage-buckets)
      - [Conformance Storage / Components](#conformance-storage--components)
      - [Yaml representation of *Conformance Storage*<sup>G1</sup>,</sup>[<sup>G2</sup>](#global-reference)](#yaml-representation-of-conformance-storagesupg1supsupsupg2sup)
    - [CIP Auditor](#cip-auditor)
      - [CIP Auditor Env](#cip-auditor-env)
        - [Terraform resources for CIP Auditor Env](#terraform-resources-for-cip-auditor-env)
        - [CIP Auditor Env / Components](#cip-auditor-env--components)
        - [Yaml representation of *CIP Auditor Env Components*<sup>G1</sup>,</sup>[<sup>G2</sup>](#global-reference)](#yaml-representation-of-cip-auditor-env-componentssupg1supsupsupg2sup)
      - [CIP Auditor Deploy](#cip-auditor-deploy)
        - [Terraform resources for CIP Auditor Deploy](#terraform-resources-for-cip-auditor-deploy)
        - [CIP Auditor Deploy / Components](#cip-auditor-deploy--components)
        - [Yaml representation of *CIP Auditor Deploy Components*<sup>G1</sup>](#yaml-representation-of-cip-auditor-deploy-componentssupg1sup)
      - [Reference for CIP Auditor](#reference-for-cip-auditor)
    - [Release Projects](#release-projects)
      - [Terraform resources for Release Projects](#terraform-resources-for-release-projects)
      - [Release Projects `[PROJECTS]`](#release-projects-projects)
      - [Release Projects / Components](#release-projects--components)
      - [Yaml representation of *Release Projects Components*<sup>G1</sup><sup>,</sup>[<sup>G2</sup>](#global-reference)](#yaml-representation-of-release-projects-componentssupg1supsupsupsupg2sup)
      - [Reference for Release Projects](#reference-for-release-projects)
    - [Releng](#releng)
      - [Releng / Components](#releng--components)
      - [Terraform resources for Releng](#terraform-resources-for-releng)
      - [Yaml representation of *Releng Components*<sup>G1</sup><sup>,</sup>[<sup>G2</sup>](#global-reference)](#yaml-representation-of-releng-componentssupg1supsupsupsupg2sup)
    - [GSuite](#gsuite)
      - [Terraform resources for GSuite](#terraform-resources-for-gsuite)
      - [GSuite / Components](#gsuite--components)
      - [Yaml representation of *GSuite Components*<sup>G1</sup><sup>,</sup>[<sup>G2</sup>](#global-reference)](#yaml-representation-of-gsuite-componentssupg1supsupsupsupg2sup)
      - [Reference for GSuite](#reference-for-gsuite)
    - [Namespaces](#namespaces)
      - [Terraform resources for Namespaces](#terraform-resources-for-namespaces)
      - [Namespaces `[PROJECTS]`](#namespaces-projects)
      - [Namespaces / Components](#namespaces--components)
      - [Yaml representation of *Namespaces Components*<sup>G1</sup>](#yaml-representation-of-namespaces-componentssupg1sup)
- [Proposed structure of new tooling](#proposed-structure-of-new-tooling)
- [Plan of work](#plan-of-work)
- [FAQ](#faq)
  - [Why terraform?](#why-terraform)
  - [How can I help?](#how-can-i-help)
- [todos](#todos)
- [Global Reference](#global-reference)

---

> [**todo(@bartsmykla)**]: Remember to write section with suggestion to rewrite [`k/k8s.io/groups`](https://github.com/kubernetes/k8s.io/tree/9e17cdf48d4e9f343e0a11ecb06247897a81dd84/groups) to custom terraform provider

## Preface

I strongly believe in rule that before you'll start solving the problem with the code you should understand well what is the problem and solve it conceptualy first.

[**IMPORTANT!**] As you can assume, I've spent some time working on this document. I come to some conclusions and have some thoughts and ideas but they are just MY conclusions and just MY foughts and ideas. PLEASE don't feel obliged to agree with me just because I spent my time working on it. If you think I'm wrong or/and you think we could improve it PLEASE write about it. Even if it would completely scrap my ideas/suggestions/plans. Let's treat it as a working document and have fruitful discussion.

### tl;dr

### How to read this document

### Why I created this document

### What problem are we trying to solve

## Current state of the tools

### Flow chart

### Components list

### Components diagram

### Analysis

One could ask why this much effort to analyse all this components by hand and not to just use [audit](https://github.com/kubernetes/k8s.io/tree/9e3204a10978dde2f4114940842e0546edfe32ce/audit) and take them drom there. This is a great question and the answer is I wanted to compare if all components from [audit](https://github.com/kubernetes/k8s.io/tree/9e3204a10978dde2f4114940842e0546edfe32ce/audit) are being created by [our tooling](https://github.com/kubernetes/k8s.io/tree/9e3204a10978dde2f4114940842e0546edfe32ce/infra/gcp).

What I don't like is there are places like CIP Auditor which need some scripts being run in a specified sequence for it to be able to correctly provision resources. We need to make it more atomic.

---

#### Main Project

---

##### Terraform data sources and resources for Main Project

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

##### Components

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

##### Yaml representation of *Main Project Components*[<sup>G1</sup>](#global-reference)<sup>,</sup>[<sup>G2</sup>](#global-reference)

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

---

#### Prod Storage

---

##### Terraform resources for Prod Storage

- Provider: [`Google`](https://www.terraform.io/docs/providers/google/index.html "Provider: Google")
  - [`google_project`](https://www.terraform.io/docs/providers/google/r/google_project.html "Resource: Google Project")
  - [`google_project_service`](https://www.terraform.io/docs/providers/google/r/google_project_service.html "Resource: Google Project Service")
  - [`google_container_registry`](https://www.terraform.io/docs/providers/google/r/container_registry.html "Resource: Google Container Registry")
  - [`google_storage_bucket`](https://www.terraform.io/docs/providers/google/r/storage_bucket.html "Resource: Google Storage Bucket")
  - [`google_service_account`](https://www.terraform.io/docs/providers/google/r/google_service_account.html "Resource: Google Service Account")
  - [`google_storage_bucket_object`](https://www.terraform.io/docs/providers/google/r/storage_bucket_object.html "Resource: Google Storage Bucket Object")
  - [`google_project_iam_binding`](https://www.terraform.io/docs/providers/google/r/google_project_iam.html "Resource: Google Project IAM Binding")
  - [`google_project_iam_member`](https://www.terraform.io/docs/providers/google/r/google_project_iam.html "Resource: Google Project IAM Member")[<sup>G2</sup>](#global-reference)
  - [`google_storage_bucket_iam_binding`](https://www.terraform.io/docs/providers/google/r/storage_bucket_iam.html "Resource: Google Storage Bucket IAM Binding")
  - [`google_storage_bucket_iam_member`](https://www.terraform.io/docs/providers/google/r/storage_bucket_iam.html "Resource: Google Storage Bucket IAM Member")[<sup>G2</sup>](#global-reference)
  - [`google_service_account_iam_binding`](https://www.terraform.io/docs/providers/google/r/google_service_account_iam.html "Resource: Google Service Account IAM Binding")
  - [`google_service_account_iam_member`](https://www.terraform.io/docs/providers/google/r/google_service_account_iam.html "Resource: Google Service Account IAM Member")[<sup>G2</sup>](#global-reference)
  - [`google_cloud_run_service_iam_binding`](https://www.terraform.io/docs/providers/google/r/cloud_run_service_iam.html "Resource: Google Cloud Run Service IAM Binding")
  - [`google_cloud_run_service_iam_member`](https://www.terraform.io/docs/providers/google/r/cloud_run_service_iam.html "Resource: Google Cloud Run Service IAM Member")[<sup>G2</sup>](#global-reference)
  
##### Prod Storage `[PROJECTS]`

- `k8s-artifacts-prod`
- `k8s-artifacts-prod-bak`
- `k8s-cip-test-prod`
- `k8s-staging-cip-test`
- `k8s-gcr-backup-test-prod`
- `k8s-gcr-backup-test-prod-bak`
- `k8s-gcr-audit-test-prod`
- `k8s-release-test-prod`

##### Prod Storage `[REGIONS]`

- `us`
- `eu`
- `asia`

##### Prod Storage / Components

- **Components per [`[PROJECT]`](#prod-storage-projects)**:
  - Project:
    - `[PROJECT]`
  - API:
    - `containerregistry`
    - `containeranalysis`
    - `storage-component`
  - GCS Bucket:
    - `gs://[PROJECT]`:
      - bucketpolicyonly: `true`
      - retention: `10y`
      - location: `us`
  - IAM:
    - `gs://[PROJECT]`:
      - `allUsers:objectViewer`
      - `group:k8s-infra-artifact-admins@kubernetes.io:objectAdmin`
      - `group:k8s-infra-artifact-admins@kubernetes.io:legacyBucketOwner`
  - IAM Policy Binding:
    - `roles/viewer`:
      - `group:k8s-infra-artifact-admins@kubernetes.io`
  - IAM Service Account:
    - `k8s-infra-gcr-promoter`:
      - display_name: `k8s-infra container image promoter`

  - **Components per [`[PROJECT]`](#prod-storage-projects) per [`[REGION]`](#prod-storage-regions)**:

    > [**Hint**]: Currently regions we are creating our resources at are: "`us`", "`eu`", "`asia`". So if you see for example GCR Resource with the name `[REGION]_[PROJECT]` for the project `k8s-artifacts-prod` (which we use for this example purposes) there will be three cloud registries (GCR) created: `us_k8s-artifacts-prod`, `eu_k8s-artifacts-prod` and `asia_k8s-artifacts-prod`

    - GCR:
      - `[REGION]_[PROJECT]`
    - GCS Bucket:
      - `gs://[REGION].artifacts.k8s-staging-[PROJECT].appspot.com`:
        - bucketpolicyonly: `true`
    - IAM:
      - `gs://[REGION].artifacts.k8s-staging-[PROJECT].appspot.com`:
        - `allUsers:objectViewer`
        - `group:k8s-infra-artifact-admins@kubernetes.io:objectAdmin`
        - `group:k8s-infra-artifact-admins@kubernetes.io:legacyBucketOwner`
        - `serviceAccount:k8s-infra-gcr-promoter@[PROJECT].iam.gserviceaccount.com:objectAdmin`
        - `serviceAccount:k8s-infra-gcr-promoter@[PROJECT].iam.gserviceaccount.com:legacyBucketOwner`

- **Additional components for project: `k8s-artifacts-prod`** (special cases):
  - GCS Bucket:
    - `gs://k8s-artifacts-prod`:
      - website[<sup>1</sup>](#reference-for-prod-storage "Special case"):
        - main_page_suffix: `index.html`
    - `gs://k8s-artifacts-cni`[<sup>2</sup>](#reference-for-prod-storage "Special case"):
      - location: `us`
      - bucketpolicyonly: `true`
      - retention: `10y`
  - IAM:
    - `gs://k8s-artifacts-cni`[<sup>2</sup>](#reference-for-prod-storage "Special case"):
      - `allUsers:objectViewer`
      - `group:k8s-infra-artifact-admins@kubernetes.io:objectAdmin`
      - `group:k8s-infra-artifact-admins@kubernetes.io:legacyBucketOwner`
      - `group:k8s-infra-push-cni@kubernetes.io:objectAdmin`
      - `group:k8s-infra-push-cni@kubernetes.io:legacyBucketReader`
  - IAM Policy Binding:
    - `roles/run.admin`[<sup>10</sup>](#reference-for-prod-storage "Special case"):
      - `group:k8s-infra-artifact-admins@kubernetes.io`
    - `roles/serviceusage.serviceUsageConsumer`[<sup>10</sup>](#reference-for-prod-storage "Special case"):
      - `group:k8s-infra-artifact-admins@kubernetes.io`
    - `roles/logging.logWriter`[<sup>11</sup>](#reference-for-prod-storage "Special case"):
      - `serviceAccount:k8s-infra-gcr-auditor@k8s-artifacts-prod.iam.gserviceaccount.com`
    - `roles/errorreporting.writer`[<sup>11</sup>](#reference-for-prod-storage "Special case"):
      - `serviceAccount:k8s-infra-gcr-auditor@k8s-artifacts-prod.iam.gserviceaccount.com`
  - IAM Policy Binding for Service Account[<sup>10</sup>](#reference-for-prod-storage "Special case"):
    - `k8s-infra-gcr-auditor@k8s-artifacts-prod.iam.gserviceaccount.com`:
      - role: `roles/iam.serviceAccountUser`
      - members:
        - `group:k8s-infra-artifact-admins@kubernetes.io`
  - IAM Service Account[<sup>11</sup>](#reference-for-prod-storage "Special case"):
    - `k8s-infra-gcr-auditor`:
      - project: `k8s-artifacts-prod`
      - display_name: `k8s-infra container image auditor`
    - `k8s-infra-gcr-auditor-invoker`:
      - project: `k8s-artifacts-prod`
      - display_name: `k8s-infra container image auditor invoker`
  - Cloud Run Service IAM Policy Binding[<sup>11</sup>](#reference-for-prod-storage "Special case"):
    - `cip-auditor`:
      - members:
        - `serviceAccount:k8s-infra-gcr-auditor-invoker@k8s-artifacts-prod.iam.gserviceaccount.com`
      - role: `roles/run.invoker`
      - platform: `managed`
      - project: `k8s-artifacts-prod`
      - region: `us-central1`

  - **Components per `[FILE]` in `[MODULE_PATH]/static/prod-storage` directory**[<sup>3</sup>](#reference-for-prod-storage "Special case")<sup>,</sup>[<sup>4</sup>](#reference-for-prod-storage "Example of how to achieve it with Terraform"):
    - GCS Bucket Object:
      - name: `[FILE]`
      - source: `[MODULE_PATH]/static/prod-storage/[FILE]`
      - bucket: `gs://k8s-artifacts-prod`

- **Components for project: `k8s-cip-test-prod`**[<sup>5</sup>](#reference-for-prod-storage "Special case"):
  - IAM Policy Binding:
    - `roles/viewer`:
      - `group:k8s-infra-staging-cip-test@kubernetes.io`
  
  - **Components for project: `k8s-cip-test-prod` per [`[REGION]`](#prod-storage-regions)**:
    - IAM:
      - `gs://[REGION].artifacts.k8s-cip-test-prod.appspot.com`:
        - `group:k8s-infra-staging-cip-test@kubernetes.io:objectAdmin`
        - `group:k8s-infra-staging-cip-test@kubernetes.io:legacyBucketOwner`

- **Components for project: `k8s-staging-cip-test`**[<sup>5</sup>](#reference-for-prod-storage "Special case"):
  - IAM Policy Binding:
    - `roles/viewer`:
      - `group:k8s-infra-staging-cip-test@kubernetes.io`
  - IAM[<sup>6</sup>](#reference-for-prod-storage "Special case"):
    - `gs://artifacts.k8s-staging-cip-test.appspot.com`
      - `serviceAccount:k8s-infra-gcr-promoter@k8s-cip-test-prod.iam.gserviceaccount.com:objectAdmin`
      - `serviceAccount:k8s-infra-gcr-promoter@k8s-cip-test-prod.iam.gserviceaccount.com:legacyBucketOwner`

  - **Components for project: `k8s-staging-cip-test` per [`[REGION]`](#prod-storage-regions)**:
    - IAM:
      - `gs://[REGION].artifacts.k8s-staging-cip-test.appspot.com`:
        - `group:k8s-infra-staging-cip-test@kubernetes.io:objectAdmin`
        - `group:k8s-infra-staging-cip-test@kubernetes.io:legacyBucketOwner`

- **Components for project: `k8s-gcr-backup-test-prod`**[<sup>5</sup>](#reference-for-prod-storage "Special case"):
  - IAM Policy Binding:
    - `roles/viewer`:
      - `group:k8s-infra-staging-cip-test@kubernetes.io`
  
  - **Components for project: `k8s-gcr-backup-test-prod` per [`[REGION]`](#prod-storage-regions)**:
    - IAM:
      - `gs://[REGION].artifacts.k8s-gcr-backup-test-prod.appspot.com`:
        - `group:k8s-infra-staging-cip-test@kubernetes.io:objectAdmin`
        - `group:k8s-infra-staging-cip-test@kubernetes.io:legacyBucketOwner`

- **Components for project: `k8s-gcr-backup-test-prod-bak`**[<sup>5</sup>](#reference-for-prod-storage "Special case"):
  - IAM Policy Binding:
    - `roles/viewer`:
      - `group:k8s-infra-staging-cip-test@kubernetes.io`
  
  - **Components for project: `k8s-gcr-backup-test-prod-bak` per [`[REGION]`](#prod-storage-regions)**:
    - IAM:
      - `gs://[REGION].artifacts.k8s-gcr-backup-test-prod-bak.appspot.com`:
        - `group:k8s-infra-staging-cip-test@kubernetes.io:objectAdmin`
        - `group:k8s-infra-staging-cip-test@kubernetes.io:legacyBucketOwner`

- **Components for project: `k8s-gcr-audit-test-prod`**[<sup>5</sup>](#reference-for-prod-storage "Special case"):
  - IAM Policy Binding[<sup>12</sup>](#reference-for-prod-storage "Found inconsistency"):
    - `roles/viewer`:
      - `group:k8s-infra-staging-cip-test@kubernetes.io`
    - `roles/errorreporting.admin`[<sup>7</sup>](#reference-for-prod-storage "Special case")
      - `serviceAccount:k8s-infra-gcr-promoter@k8s-gcr-audit-test-prod.iam.gserviceaccount.com`
    - `roles/logging.admin`[<sup>7</sup>](#reference-for-prod-storage "Special case")
      - `serviceAccount:k8s-infra-gcr-promoter@k8s-gcr-audit-test-prod.iam.gserviceaccount.com`
    - `roles/pubsub.admin`[<sup>7</sup>](#reference-for-prod-storage "Special case")
      - `serviceAccount:k8s-infra-gcr-promoter@k8s-gcr-audit-test-prod.iam.gserviceaccount.com`
    - `roles/resourcemanager.projectIamAdmin`[<sup>7</sup>](#reference-for-prod-storage "Special case")
      - `serviceAccount:k8s-infra-gcr-promoter@k8s-gcr-audit-test-prod.iam.gserviceaccount.com`
    - `roles/run.admin`[<sup>7</sup>](#reference-for-prod-storage "Special case")
      - `serviceAccount:k8s-infra-gcr-promoter@k8s-gcr-audit-test-prod.iam.gserviceaccount.com`
    - `roles/serverless.serviceAgent`[<sup>7</sup>](#reference-for-prod-storage "Special case")
      - `serviceAccount:k8s-infra-gcr-promoter@k8s-gcr-audit-test-prod.iam.gserviceaccount.com`
    - `roles/storage.admin`[<sup>7</sup>](#reference-for-prod-storage "Special case")
      - `serviceAccount:k8s-infra-gcr-promoter@k8s-gcr-audit-test-prod.iam.gserviceaccount.com`
  
  - **Components for project: `k8s-gcr-audit-test-prod` per [`[REGION]`](#prod-storage-regions)**:
    - IAM:
      - `gs://[REGION].artifacts.k8s-gcr-audit-test-prod.appspot.com`:
        - `group:k8s-infra-staging-cip-test@kubernetes.io:objectAdmin`
        - `group:k8s-infra-staging-cip-test@kubernetes.io:legacyBucketOwner`

- **Components for project: `k8s-release-test-prod`**[<sup>8</sup>](#reference-for-prod-storage "Special case"):
  - IAM Policy Binding:
    - `roles/viewer`:
      - `group:k8s-infra-staging-kubernetes@kubernetes.io`
      - `group:k8s-infra-staging-release-test@kubernetes.io`
  - IAM:
    - `gs://k8s-release-test-prod`[<sup>9</sup>](#reference-for-prod-storage "Special case"):
      - `serviceAccount:615281671549@cloudbuild.gserviceaccount.com:objectAdmin`
      - `serviceAccount:615281671549@cloudbuild.gserviceaccount.com:legacyBucketReader`
  
  - **Components for project: `k8s-release-test-prod` per [`[REGION]`](#prod-storage-regions)**:
    - IAM:
      - `gs://[REGION].artifacts.k8s-release-test-prod.appspot.com`:
        - `group:k8s-infra-staging-kubernetes@kubernetes.io:objectAdmin`
        - `group:k8s-infra-staging-kubernetes@kubernetes.io:legacyBucketOwner`
        - `group:k8s-infra-staging-release-test@kubernetes.io:objectAdmin`
        - `group:k8s-infra-staging-release-test@kubernetes.io:legacyBucketOwner`

##### Yaml representation of *Prod Storage*[<sup>G1</sup>](#global-reference)<sup>,</sup>[<sup>G2</sup>](#global-reference)

```yaml
# [PROJECTS]
#   - k8s-artifacts-prod
#   - k8s-artifacts-prod-bak
#   - k8s-cip-test-prod
#   - k8s-staging-cip-test
#   - k8s-gcr-backup-test-prod
#   - k8s-gcr-backup-test-prod-bak
#   - k8s-gcr-audit-test-prod
#   - k8s-release-test-prod
#
# [REGIONS]
#
#   - us
#   - eu
#   - asia

google_project:
  - name: "[PROJECT]"
google_project_service:
  - service: containerregistry.googleapis.com
    project: "[PROJECT]"
  - service: containeranalysis.googleapis.com
    project: "[PROJECT]"
  - service: storage-component.googleapis.com
    project: "[PROJECT]"
google_container_registry:
  project: "[REGION]_[PROJECT]"
google_storage_bucket:
  - name: "[PROJECT]"
    bucket_policy_only: true
    location: us
    retention_policy:
      retention_period: 315360000 # 10 years
  - name: "[REGION].artifacts.k8s-staging-[PROJECT].appspot.com"
    bucket_policy_only: true
  # START: Additional components for project: "k8s-artifacts-prod"
  - name: k8s-artifacts-prod
    website:
      main_page_suffix: index.html
    project: k8s-artifacts-prod
  - name: k8s-artifacts-cni
    location: us
    bucket_policy_only: true
    retention_policy:
      retention_period: 315360000 # 10 years
    project: k8s-artifacts-prod
  # END
google_project_iam_binding:
  - role: roles/viewer
    members:
      - group:k8s-infra-artifact-admins@kubernetes.io
    project: "[PROJECT]"
  # START: Additional components for project: "k8s-artifacts-prod"
  - role: roles/run.admin
    members:
      - group:k8s-infra-artifact-admins@kubernetes.io
    project: k8s-artifacts-prod
  - role: roles/serviceusage.serviceUsageConsumer
    members:
      - group:k8s-infra-artifact-admins@kubernetes.io
    project: k8s-artifacts-prod
  - role: roles/logging.logWriter
    members:
      - serviceAccount:k8s-infra-gcr-auditor@k8s-artifacts-prod.iam.gserviceaccount.com
    project: k8s-artifacts-prod
  - role: roles/errorreporting.writer
    members:
      - serviceAccount:k8s-infra-gcr-auditor@k8s-artifacts-prod.iam.gserviceaccount.com
    project: k8s-artifacts-prod
  # END
  # START: Additional components for project: "k8s-cip-test-prod"
  - role: roles/viewer
    members:
      - group:k8s-infra-staging-cip-test@kubernetes.io
    project: k8s-cip-test-prod
  # END
  # START: Additional components for project: "k8s-staging-cip-test"
  - role: roles/viewer
    members:
      - group:k8s-infra-staging-cip-test@kubernetes.io
    project: k8s-staging-cip-test
  # END
  # START: Additional components for project: "k8s-gcr-backup-test-prod"
  - role: roles/viewer
    members:
      - group:k8s-infra-staging-cip-test@kubernetes.io
    project: k8s-gcr-backup-test-prod
  # END
  # START: Additional components for project: "k8s-gcr-backup-test-prod-bak"
  - role: roles/viewer
    members:
      - group:k8s-infra-staging-cip-test@kubernetes.io
    project: k8s-gcr-backup-test-prod-bak
  # END
  # START: Additional components for project: "k8s-gcr-audit-test-prod"
  - role: roles/viewer
    members:
      - group:k8s-infra-staging-cip-test@kubernetes.io
    project: k8s-gcr-audit-test-prod
  - role: roles/errorreporting.admin
    members:
      - serviceAccount:k8s-infra-gcr-promoter@k8s-gcr-audit-test-prod.iam.gserviceaccount.com
    project: k8s-gcr-audit-test-prod
  - role: roles/logging.admin
    members:
      - serviceAccount:k8s-infra-gcr-promoter@k8s-gcr-audit-test-prod.iam.gserviceaccount.com
    project: k8s-gcr-audit-test-prod
  - role: roles/pubsub.admin
    members:
      - serviceAccount:k8s-infra-gcr-promoter@k8s-gcr-audit-test-prod.iam.gserviceaccount.com
    project: k8s-gcr-audit-test-prod
  - role: roles/resourcemanager.projectIamAdmin
    members:
      - serviceAccount:k8s-infra-gcr-promoter@k8s-gcr-audit-test-prod.iam.gserviceaccount.com
    project: k8s-gcr-audit-test-prod
  - role: roles/run.admin
    members:
      - serviceAccount:k8s-infra-gcr-promoter@k8s-gcr-audit-test-prod.iam.gserviceaccount.com
    project: k8s-gcr-audit-test-prod
  - role: roles/serverless.serviceAgent
    members:
      - serviceAccount:k8s-infra-gcr-promoter@k8s-gcr-audit-test-prod.iam.gserviceaccount.com
    project: k8s-gcr-audit-test-prod
  - role: roles/storage.admin
    members:
      - serviceAccount:k8s-infra-gcr-promoter@k8s-gcr-audit-test-prod.iam.gserviceaccount.com
    project: k8s-gcr-audit-test-prod
  # END
  # START: Additional components for project: "k8s-release-test-prod"
  - role: roles/viewer
    members:
      - group:k8s-infra-staging-kubernetes@kubernetes.io
      - group:k8s-infra-staging-release-test@kubernetes.io
    project: k8s-release-test-prod
  # END
# START: Additional components for project: "k8s-artifacts-prod"
google_service_account:
  - account_id: k8s-infra-gcr-auditor
    display_name: k8s-infra container image auditor
    project: k8s-artifacts-prod
  - account_id: k8s-infra-gcr-auditor-invoker
    display_name: k8s-infra container image auditor invoker
    project: k8s-artifacts-prod
google_service_account_iam_binding:
  - service_account_id: k8s-infra-gcr-auditor@k8s-artifacts-prod.iam.gserviceaccount.com
    role: roles/iam.serviceAccountUser
    members:
      - group:k8s-infra-artifact-admins@kubernetes.io
google_cloud_run_service_iam_binding:
  service_name: cip-auditor
  role: roles/run.invoker
  members:
    - serviceAccount:k8s-infra-gcr-auditor-invoker@k8s-artifacts-prod.iam.gserviceaccount.com
  location: us-central1
  project: k8s-artifacts-prod
google_storage_bucket_object:
# We'll be copying all static files from "static/prod-storage"
  - name: "[NAME]"
    source: "[MODULE_PATH]/static/prod-storage/[FILE]"
    bucket: gs://k8s-artifacts-prod
# END
google_storage_bucket_iam_binding:
  # gs://[PROJECT]
  - role: roles/storage.objectViewer
    members:
      - allUsers
    bucket: gs://[PROJECT]
  - role: roles/storage.objectAdmin
    members:
      - group:k8s-infra-artifact-admins@kubernetes.io
    bucket: gs://[PROJECT]
  - role: roles/storage.legacyBucketOwner
    members:
      - group:k8s-infra-artifact-admins@kubernetes.io
    bucket: gs://[PROJECT]
  # gs://[REGION].artifacts.k8s-staging-[PROJECT].appspot.com
  - role: roles/storage.objectViewer
    members:
      - allUsers
    bucket: gs://[REGION].artifacts.k8s-staging-[PROJECT].appspot.com
  - role: roles/storage.objectAdmin
    members:
      - group:k8s-infra-artifact-admins@kubernetes.io
      - serviceAccount:k8s-infra-gcr-promoter@[PROJECT].iam.gserviceaccount.com
    bucket: gs://[REGION].artifacts.k8s-staging-[PROJECT].appspot.com
  - role: roles/storage.legacyBucketOwner
    members:
      - group:k8s-infra-artifact-admins@kubernetes.io
      - serviceAccount:k8s-infra-gcr-promoter@[PROJECT].iam.gserviceaccount.com
    bucket: gs://[REGION].artifacts.k8s-staging-[PROJECT].appspot.com
  # gs://k8s-artifacts-cni
  # START: Additional components for project: "k8s-artifacts-prod"
  - role: roles/storage.objectViewer
    members:
      - allUsers
    bucket: gs://k8s-artifacts-cni
  - role: roles/storage.objectAdmin
    members:
      - group:k8s-infra-artifact-admins@kubernetes.io
      - group:k8s-infra-push-cni@kubernetes.io
    bucket: gs://k8s-artifacts-cni
  - role: roles/storage.legacyBucketOwner
    members:
      - group:k8s-infra-artifact-admins@kubernetes.io
    bucket: gs://k8s-artifacts-cni
  - role: roles/storage.legacyBucketReader
    members:
      - group:k8s-infra-push-cni@kubernetes.io
    bucket: gs://k8s-artifacts-cni
  # END
  # START: Additional components for project: "k8s-cip-test-prod"
  - role: roles/storage.objectAdmin
    members:
      - group:k8s-infra-staging-cip-test@kubernetes.io
    bucket: gs://[REGION].artifacts.k8s-cip-test-prod.appspot.com
  - role: roles/storage.legacyBucketOwner
    members:
      - group:k8s-infra-staging-cip-test@kubernetes.io
    bucket: gs://[REGION].artifacts.k8s-cip-test-prod.appspot.com
  # END
  # START: Additional components for project: "k8s-staging-cip-test"
  - role: roles/storage.objectAdmin
    members:
      - serviceAccount:k8s-infra-gcr-promoter@k8s-cip-test-prod.iam.gserviceaccount.com
    bucket: gs://artifacts.k8s-staging-cip-test.appspot.com
  - role: roles/storage.legacyBucketOwner
    members:
      - serviceAccount:k8s-infra-gcr-promoter@k8s-cip-test-prod.iam.gserviceaccount.com
    bucket: gs://artifacts.k8s-staging-cip-test.appspot.com
  - role: roles/storage.objectAdmin
    members:
      - group:k8s-infra-staging-cip-test@kubernetes.io
    bucket: gs://[REGION].artifacts.k8s-staging-cip-test.appspot.com
  - role: roles/storage.legacyBucketOwner
    members:
      - group:k8s-infra-staging-cip-test@kubernetes.io
    bucket: gs://[REGION].artifacts.k8s-staging-cip-test.appspot.com
  # END
  # START: Additional components for project: "k8s-gcr-backup-test-prod"
  - role: roles/storage.objectAdmin
    members:
      - group:k8s-infra-staging-cip-test@kubernetes.io
    bucket: gs://[REGION].artifacts.k8s-gcr-backup-test-prod.appspot.com
  - role: roles/storage.legacyBucketOwner
    members:
      - group:k8s-infra-staging-cip-test@kubernetes.io
    bucket: gs://[REGION].artifacts.k8s-gcr-backup-test-prod.appspot.com
  # END
  # START: Additional components for project: "k8s-gcr-backup-test-prod-bak"
  - role: roles/storage.objectAdmin
    members:
      - group:k8s-infra-staging-cip-test@kubernetes.io
    bucket: gs://[REGION].artifacts.k8s-gcr-backup-test-prod-bak.appspot.com
  - role: roles/storage.legacyBucketOwner
    members:
      - group:k8s-infra-staging-cip-test@kubernetes.io
    bucket: gs://[REGION].artifacts.k8s-gcr-backup-test-prod-bak.appspot.com
  # END
  # START: Additional components for project: "k8s-gcr-audit-test-prod"
  - role: roles/storage.objectAdmin
    members:
      - group:k8s-infra-staging-cip-test@kubernetes.io
    bucket: gs://[REGION].artifacts.k8s-gcr-audit-test-prod.appspot.com
  - role: roles/storage.legacyBucketOwner
    members:
      - group:k8s-infra-staging-cip-test@kubernetes.io
    bucket: gs://[REGION].artifacts.k8s-gcr-audit-test-prod.appspot.com
  # END
  # START: Additional components for project: "k8s-release-test-prod"
  - role: roles/storage.objectAdmin
    members:
      - serviceAccount:615281671549@cloudbuild.gserviceaccount.com
    bucket: gs://k8s-release-test-prod
  - role: roles/storage.legacyBucketOwner
    members:
      - serviceAccount:615281671549@cloudbuild.gserviceaccount.com
    bucket: gs://k8s-release-test-prod
  - role: roles/storage.objectAdmin
    members:
      - group:k8s-infra-staging-kubernetes@kubernetes.io
      - group:k8s-infra-staging-release-test@kubernetes.io
    bucket: gs://[REGION].artifacts.k8s-release-test-prod.appspot.com
  - role: roles/storage.legacyBucketOwner
    members:
      - group:k8s-infra-staging-kubernetes@kubernetes.io
      - group:k8s-infra-staging-release-test@kubernetes.io
    bucket: gs://[REGION].artifacts.k8s-release-test-prod.appspot.com
  # END
```

##### Notes for Prod Storage

- As we can see here is a lot of special cases. By nature of bash, whole process of provisioning components is procedural, so for purposes of not repeating themselves, creators of this script are first iterating over all prod projects and provisioning components which are common for each of them and then if there are still components which need to be provisioned (special cases) for some of the projects they are doing it. We need to find a better way for doing it when using terraform.
- At [line 212](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/ensure-prod-storage.sh#L211-L212) we are removing retention from `gs://k8s-cip-test-prod` - think about how to be able to doesn't need to explicitly remove settings which were earlier also set by us. It also means.
- ^ ***THIS CHANGE (REMOVING RETENTION) IS NOT REFLECTED AT LIST OF COMPONENTS ABOVE*** - it's because currently I'm representing all components the same way as it's done at existing scripts (first iterating over all projects with common components and then iterating again for special cases) and when we'll decide to use that components list to create terraform configurations it will be problematic, because declarative approach by nature won't have problem with firstly assigning some property and then removing it because we'll just describe the state of infrastructure we want to achieve - not the steps.

##### Reference for Prod Storage

- <sup>1</sup> [Special case: `ensure-prod-storage.sh#L139`](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/ensure-prod-storage.sh#L139)
- <sup>2</sup> [Special case: `ensure-prod-storage.sh#L149`](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/ensure-prod-storage.sh#L149)
- <sup>3</sup> [Special case: `ensure-prod-storage.sh#L143`](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/ensure-prod-storage.sh#L143)
- <sup>4</sup> Example of how to recursively upload files from directory

  > [**todo(@bartsmykla)**]: Verify it works

  ```terraform
  variable statics_path {
    default = "${path.module}/static/prod-storage"
  }

  resource "google_storage_bucket_object" "prod_storage_statics" {
    for_each = fileset(var.statics_path, "**")
    name     = each.value
    source   = "${var.statics_path}/${each.value}"
  }
  ```

- <sup>5</sup> [Special case: `ensure-prod-storage.sh#L163-L164`](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/ensure-prod-storage.sh#L163-L164)
- <sup>6</sup> [Special case: `ensure-prod-storage.sh#L181-L182`](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/ensure-prod-storage.sh#L181-L182)
- <sup>7</sup> [Special case: `ensure-prod-storage.sh#L187-L189`](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/ensure-prod-storage.sh#L187-L189)
- <sup>8</sup> [Special case: `ensure-prod-storage.sh#L194-L195`](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/ensure-prod-storage.sh#L194-L195)
- <sup>9</sup> [Special case: `ensure-prod-storage.sh#L204-L206`](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/ensure-prod-storage.sh#L204-L206)
- <sup>10</sup> [Special case: `ensure-prod-storage.sh#L214-L215`](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/ensure-prod-storage.sh#L214-L215)
- <sup>11</sup> [Special case: `ensure-prod-storage.sh#L219`](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/ensure-prod-storage.sh#L219)
- <sup>12</sup>[**todo(@bartsmykla)**]: When doing analysis of CIP Auditor scripts I found that [IAMs for `k8s-gcr-audit-test-prod`](https://github.com/kubernetes/k8s.io/blob/e62c18e79a75615d4868afaf5eebcf36bb265df9/audit/projects/k8s-gcr-audit-test-prod/iam.json) doesn't match exactly with those assigned by our scripts. I need to compare them more precisely

---

#### Prod Storage GCLB

---

##### Terraform resources for Prod Storage GCLB

- Provider: [`Google`](https://www.terraform.io/docs/providers/google/index.html "Provider: Google")
  - [`google_project`](https://www.terraform.io/docs/providers/google/r/google_project.html "Resource: Google Project")
  - [`google_project_service`](https://www.terraform.io/docs/providers/google/r/google_project_service.html "Resource: Google Project Service")
  - [`google_compute_global_address`](https://www.terraform.io/docs/providers/google/r/compute_address.html "Resource: Google Compute Address")
  - [`google_compute_backend_bucket`](https://www.terraform.io/docs/providers/google/r/compute_backend_bucket.html "Resource: Google Compute Backend Bucket")
  - [`google_compute_url_map`](https://www.terraform.io/docs/providers/google/r/compute_url_map.html "Resource: Google Compute URL Map")
  - [`google_compute_managed_ssl_certificate`](https://www.terraform.io/docs/providers/google/r/compute_managed_ssl_certificate.html "Resource: Google Compute Managed SSL Certificate")
  - [`google_compute_target_http_proxy`](https://www.terraform.io/docs/providers/google/r/compute_target_http_proxy.html "Resource: Google Compute Target HTTP Proxy")
  - [`google_compute_target_https_proxy`](https://www.terraform.io/docs/providers/google/r/compute_target_https_proxy.html "Resource: Google Compute Target HTTPS Proxy")
  - [`google_compute_global_forwarding_rule`](https://www.terraform.io/docs/providers/google/r/compute_global_forwarding_rule.html "Resource: Google Compute Global Forwarding Rule")

##### Prod Storage GCLB / Components

- **Components for project: `k8s-artifacts-prod`**
  - Project:
    - `k8s-artifacts-prod`
  - API:
    - `compute`
  - Compute Global Address:
    - `k8s-artifacts-prod`:
      - description: `IP Address for GCLB for binary artifacts`
  - Compute Backend Bucket:
    - `k8s-artifacts-prod`:
      - gcs_bucket_name: `k8s-artifacts-prod`
  - Compute Url Map:
    - `k8s-artifacts-prod`:
      - default_backend_bucket: `k8s-artifacts-prod`
  - Compute Target HTTP Proxy:
    - `k8s-artifacts-prod`:
      - url_map: `k8s-artifacts-prod`
  - Compute Target HTTPS Proxy:
    - `k8s-artifacts-prod`:
      - url_map: `k8s-artifacts-prod`
      - ssl_certificates:
        - `k8s-artifacts-prod`
  - Compute Managed SSL Certificate:
    - `k8s-artifacts-prod`:
      - domains:
        - `artifacts.k8s.io`
  - Compute Global Forwarding Rule:
    - `k8s-artifacts-prod-http`:
      - target_http_proxy: `k8s-artifacts-prod`
      - ports:
        - `80`
      - ip_address: `[GLOBAL_ADDRESS]`[<sup>1</sup>](#reference-for-prod-storage-gclb "Example how to get [GLOBAL_ADDRESS] using terraform")
    - `k8s-artifacts-prod-https`:
      - target_https_proxy: `k8s-artifacts-prod`
      - ports:
        - `443`
      - ip_address: `[GLOBAL_ADDRESS]`[<sup>1</sup>](#reference-for-prod-storage-gclb "Example how to get [GLOBAL_ADDRESS] using terraform")

##### Yaml representation of *Prod Storage GCLB*[<sup>G1</sup>](#global-reference)

```yaml
google_project:
  - name: k8s-artifacts-prod
google_project_service:
  - service: compute.googleapis.com
    project: k8s-artifacts-prod
google_compute_global_address:
  - name: k8s-artifacts-prod
    description: IP Address for GCLB for binary artifacts
    project: k8s-artifacts-prod
google_compute_backend_bucket:
  - name: k8s-artifacts-prod
    bucket_name: k8s-artifacts-prod
    project: k8s-artifacts-prod
google_compute_url_map:
# API describes "default_service" as a string with url to backend_service:
# https://cloud.google.com/compute/docs/reference/rest/v1/urlMaps
# but terraform documentation says you can put here the name of a bucket too
# https://www.terraform.io/docs/providers/google/r/compute_url_map.html
# and also in our scripts we are using bucket --default-backend-bucket flag
# https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/ensure-prod-storage-gclb.sh#L71-L73
#
# [todo(@bartsmykla)]: confirm that it's fine to use it that way
  - name: k8s-artifacts-prod
    default_service: k8s-artifacts-prod
    project: k8s-artifacts-prod
google_compute_managed_ssl_certificate:
# [Warning!]: This resource is in beta, and should be used 
# with the terraform-provider-google-beta provider
# https://www.terraform.io/docs/providers/google/r/compute_managed_ssl_certificate.html
  - name: k8s-artifacts-prod
    provider: google-beta
    managed:
      domains:
        - artifacts.k8s.io
    project: k8s-artifacts-prod
google_compute_target_http_proxy:
  - name: k8s-artifacts-prod
    url_map: k8s-artifacts-prod
    project: k8s-artifacts-prod
google_compute_target_https_proxy:
  - name: k8s-artifacts-prod
    url_map: k8s-artifacts-prod
    ssl_certificates:
      - k8s-artifacts-prod
    project: k8s-artifacts-prod
google_compute_global_forwarding_rule:
# [GLOBAL_ADDRESS] will be taken from resource: "google_compute_global_address" attribute reference
  - name: k8s-artifacts-prod-http
    target: k8s-artifacts-prod
    ports:
      - 80
    ip_address: [GLOBAL_ADDRESS]
  - name: k8s-artifacts-prod-https
    target: k8s-artifacts-prod
    ports:
      - 443
    ip_address: [GLOBAL_ADDRESS]
```

##### Reference for Prod Storage GCLB

- <sup>1</sup> The example which shows how to get the `[GLOBAL_ADDRESS]` using terraform with all intermediate components

  > [**todo(@bartsmykla)**]: Verify it works

  ```terraform
  resource "google_compute_global_address" "k8s_artifacts_prod" {
    name        = "k8s-artifacts-prod"
    description = "IP Address for GCLB for binary artifacts"
  }

  resource "google_storage_bucket" "k8s_artifacts_prod" {
    name          = "k8s-artifacts-prod"
    location      = "US"

    bucket_policy_only = true

    website {
      main_page_suffix = "index.html"
    }

    retention_policy {
      retention_period = 315360000 // 10 years
    }
  }

  resource "google_compute_backend_bucket" "k8s_artifacts_prod" {
    name        = "k8s-artifacts-prod"
    bucket_name = google_storage_bucket.k8s_artifacts_prod.name
  }

  resource "google_compute_url_map" "k8s_artifacts_prod" {
    name            = "k8s-artifacts-prod"
    default_service = google_compute_backend_bucket.k8s_artifacts_prod.self_link
  }

  resource "google_compute_target_http_proxy" "k8s_artifacts_prod" {
    name        = "k8s-artifacts-prod"
    url_map     = google_compute_url_map.k8s_artifacts_prod.self_link
  }

  resource "google_compute_global_forwarding_rule" "k8s_artifacts_prod_http" {
    name       = "k8s-artifacts-prod-http"
    target     = google_compute_target_http_proxy.k8s_artifacts_prod.self_link
    ip_address = google_compute_global_address.k8s_artifacts_prod.address
    port_range = "80"
  }
  ```

---

#### Staging Storage

---

##### Terraform resources for Staging Storage

- Provider: [`Google`](https://www.terraform.io/docs/providers/google/index.html "Provider: Google")
  - [`google_project`](https://www.terraform.io/docs/providers/google/r/google_project.html "Resource: Google Project")
  - [`google_project_service`](https://www.terraform.io/docs/providers/google/r/google_project_service.html "Resource: Google Project Service")
  - [`google_container_registry`](https://www.terraform.io/docs/providers/google/r/container_registry.html "Resource: Google Container Registry")
  - [`google_storage_bucket`](https://www.terraform.io/docs/providers/google/r/storage_bucket.html "Resource: Google Storage Bucket")
  - [`google_project_iam_binding`](https://www.terraform.io/docs/providers/google/r/google_project_iam.html "Resource: Google Project IAM Binding")
  - [`google_project_iam_member`](https://www.terraform.io/docs/providers/google/r/google_project_iam.html "Resource: Google Project IAM Member")[<sup>G2</sup>](#global-reference)
  - [`google_storage_bucket_iam_binding`](https://www.terraform.io/docs/providers/google/r/storage_bucket_iam.html "Resource: Google Storage Bucket IAM Binding")
  - [`google_storage_bucket_iam_member`](https://www.terraform.io/docs/providers/google/r/storage_bucket_iam.html "Resource: Google Storage Bucket IAM Member")[<sup>G2</sup>](#global-reference)
  - [`google_storage_bucket_acl`](https://www.terraform.io/docs/providers/google/r/storage_bucket_acl.html "Resource: Google Storage Bucket ACL")

##### Staging Storage `[PROJECTS]`

- `apisnoop`
- `artifact-promoter`
- `build-image`
- `cip-test`
- `cluster-api`
- `cluster-api-aws`
- `cluster-api-azure`
- `cluster-api-gcp`
- `capi-openstack`
- `capi-kubeadm`
- `capi-docker`
- `capi-vsphere`
- `coredns`
- `csi`
- `descheduler`
- `e2e-test-images`
- `external-dns`
- `kas-network-proxy`
- `kops`
- `kube-state-metrics`
- `kubeadm`
- `kubernetes`
- `metrics-server`
- `multitenancy`
- `nfd`
- `npd`
- `provider-azure`
- `publishing-bot`
- `release-test`
- `releng`
- `scl-image-builder`
- `service-apis`
- `txtdirect`

##### Staging Storage `[RS_PROJECTS]`

- `kubernetes`
- `release-test`
- `releng`

##### Staging Storage / Components

- **Components for [`[PROJECT]`](#staging-storage-projects)**:
  - Project `k8s-staging-[PROJECT]`:
    - organization: `758905017065`
    - billing_account: `018801-93540E-22A20E`
  - IAM Policy Binding:
    - `roles/viewer`:
      - `group:k8s-infra-staging-[PROJECT]@kubernetes.io`
      - `group:k8s-infra-artifact-admins@kubernetes.io`
    - `roles/cloudbuild.builds.editor`:
      - `group:k8s-infra-staging-[PROJECT]@kubernetes.io`
    - `roles/serviceusage.serviceUsageConsumer`:
      - `group:k8s-infra-staging-[PROJECT]@kubernetes.io`
    - `roles/cloudbuild.builds.builder`:
      - `serviceAccount:deployer@k8s-prow.iam.gserviceaccount.com`
  - GCR:
    - `k8s-staging-[PROJECT]`
  - IAM:
    - `gs://artifacts.k8s-staging-[PROJECT].appspot.com`:
      - `allUsers:objectViewer`
      - `group:k8s-infra-artifact-admins@kubernetes.io:objectAdmin`
      - `group:k8s-infra-artifact-admins@kubernetes.io:legacyBucketOwner`
      - `group:k8s-infra-staging-[PROJECT]@kubernetes.io:objectAdmin`
      - `group:k8s-infra-staging-[PROJECT]@kubernetes.io:legacyBucketReader`
    - `gs://k8s-staging-[PROJECT]`:
      - `allUsers:objectViewer`
      - `group:k8s-infra-artifact-admins@kubernetes.io:objectAdmin`
      - `group:k8s-infra-artifact-admins@kubernetes.io:legacyBucketOwner`
      - `group:k8s-infra-staging-[PROJECT]@kubernetes.io:objectAdmin`
      - `group:k8s-infra-staging-[PROJECT]@kubernetes.io:legacyBucketReader`
    - `gs://k8s-staging-[PROJECT]-gcb`:
      - `allUsers:objectViewer`
      - `group:k8s-infra-artifact-admins@kubernetes.io:objectAdmin`
      - `group:k8s-infra-artifact-admins@kubernetes.io:legacyBucketOwner`
      - `group:k8s-infra-staging-[PROJECT]@kubernetes.io:objectAdmin`
      - `group:k8s-infra-staging-[PROJECT]@kubernetes.io:legacyBucketReader`
      - `serviceAccount:deployer@k8s-prow.iam.gserviceaccount.com:objectCreator`
      - `serviceAccount:deployer@k8s-prow.iam.gserviceaccount.com:objectViewer`
  - API:
    - `containerregistry`
    - `storage-component`
    - `cloudkms`
    - `cloudbuild`
  - GCS Bucket:
    - `gs://k8s-staging-[PROJECT]`:
      - bucketpolicyonly: `true`
      - location: `us`
      - auto_deletion: `60 days`
    - `gs://k8s-staging-[PROJECT]-gcb`:
      - bucketpolicyonly: `true`
      - location: `us`
      - auto_deletion: `60 days`
    - `gs://artifacts.k8s-staging-[PROJECT].appspot.com`:
      - bucketpolicyonly: `true`

- **Components for [RELEASE_STAGING_PROJECTS `[RS_PROJECT]`](#staging-storage-rsprojects)**:

  > [@bartsmykla: I think it's not consistent to put resources for release projects here, even if they are release ones. I think there should be created separate place for it]

  - IAM Policy Binding:
    - `roles/viewer`:
      - `group:k8s-infra-release-viewers@kubernetes.io`

- **Components for project: `k8s-staging-release-test`**:

  - IAM Policy Binding:
    - `roles/cloudkms.admin`:
      - `group:k8s-infra-release-admins@kubernetes.io`
    - `roles/cloudkms.cryptoKeyEncrypterDecrypter`:
      - `group:k8s-infra-release-admins@kubernetes.io`

##### Yaml representation of *Staging Storage*[<sup>G1</sup>](#global-reference),</sup>[<sup>G2</sup>](#global-reference)

```yaml
# [PROJECTS]:
#   - apisnoop
#   - artifact-promoter
#   - build-image
#   - cip-test
#   - cluster-api
#   - cluster-api-aws
#   - cluster-api-azure
#   - cluster-api-gcp
#   - capi-openstack
#   - capi-kubeadm
#   - capi-docker
#   - capi-vsphere
#   - coredns
#   - csi
#   - descheduler
#   - e2e-test-images
#   - external-dns
#   - kas-network-proxy
#   - kops
#   - kube-state-metrics
#   - kubeadm
#   - kubernetes
#   - metrics-server
#   - multitenancy
#   - nfd
#   - npd
#   - provider-azure
#   - publishing-bot
#   - release-test
#   - releng
#   - scl-image-builder
#   - service-apis
#   - txtdirect
#
# [RS_PROJECTS]:
#   - kubernetes
#   - release-test
#   - releng

google_project:
  - name: k8s-staging-[PROJECT]
    org_id: 758905017065
    billing_account: 018801-93540E-22A20E
google_project_service:
  - service: containerregistry.googleapis.com
    project: k8s-staging-[PROJECT]
  - service: storage-component.googleapis.com
    project: k8s-staging-[PROJECT]
  - service: cloudkms.googleapis.com
    project: k8s-staging-[PROJECT]
  - service: cloudbuild.googleapis.com
    project: k8s-staging-[PROJECT]
google_container_registry:
  project: k8s-staging-[PROJECT]
google_storage_bucket:
  - name: k8s-staging-[PROJECT]
    bucket_policy_only: true
    location: us
    retention_policy:
      retention_period: 5184000 # 60 days
  - name: k8s-staging-[PROJECT]-gcb
    bucket_policy_only: true
    location: us
    retention_policy:
      retention_period: 5184000 # 60 days
  - name: artifacts.k8s-staging-[PROJECT].appspot.com
    bucket_policy_only: true
google_project_iam_binding:
  - role: roles/viewer
    members:
      - group:k8s-infra-staging-[PROJECT]@kubernetes.io
      - group:k8s-infra-artifact-admins@kubernetes.io
    project: k8s-staging-[PROJECT]
  - role: roles/cloudbuild.builds.editor
    members:
      - group:k8s-infra-staging-[PROJECT]@kubernetes.io
    project: k8s-staging-[PROJECT]
  - role: roles/serviceusage.serviceUsageConsumer
    members:
      - group:k8s-infra-staging-[PROJECT]@kubernetes.io
    project: k8s-staging-[PROJECT]
  - role: roles/cloudbuild.builds.builder
    members:
      - serviceAccount:deployer@k8s-prow.iam.gserviceaccount.com
    project: k8s-staging-[PROJECT]
  # Additional components for [RS_PROJECTS]
  - role: roles/viewer
    members:
      - group:k8s-infra-release-viewers@kubernetes.io
    project: k8s-staging-[RS_PROJECTS]
  # Additional components for project: "k8s-staging-release-test"
  - role: roles/cloudkms.admin
    members:
      - group:k8s-infra-release-admins@kubernetes.io
    project: k8s-staging-release-test
  - role: roles/cloudkms.cryptoKeyEncrypterDecrypter
    members:
      - group:k8s-infra-release-admins@kubernetes.io
    project: k8s-staging-release-test
google_storage_bucket_iam_binding:
  # gs://artifacts.k8s-staging-[PROJECT].appspot.com
  - role: roles/storage.objectViewer
    members:
      - allUsers
    bucket: gs://artifacts.k8s-staging-[PROJECT].appspot.com
  - role: roles/storage.objectAdmin
    members:
      - group:k8s-infra-artifact-admins@kubernetes.io
      - group:k8s-infra-staging-[PROJECT]@kubernetes.io
    bucket: gs://artifacts.k8s-staging-[PROJECT].appspot.com
  - role: roles/storage.legacyBucketOwner
    members:
      - group:k8s-infra-release-admins@kubernetes.io
    bucket: gs://artifacts.k8s-staging-[PROJECT].appspot.com
  - role: roles/storage.legacyBucketReader
    members:
      - group:k8s-infra-staging-[PROJECT]@kubernetes.io
    bucket: gs://artifacts.k8s-staging-[PROJECT].appspot.com
  # gs://k8s-staging-[PROJECT]
  - role: roles/storage.objectViewer
    members:
      - allUsers
    bucket: gs://k8s-staging-[PROJECT]
  - role: roles/storage.objectAdmin
    members:
      - group:k8s-infra-artifact-admins@kubernetes.io
      - group:k8s-infra-staging-[PROJECT]@kubernetes.io
    bucket: gs://k8s-staging-[PROJECT]
  - role: roles/storage.legacyBucketOwner
    members:
      - group:k8s-infra-release-admins@kubernetes.io
    bucket: gs://k8s-staging-[PROJECT]
  - role: roles/storage.legacyBucketReader
    members:
      - group:k8s-infra-staging-[PROJECT]@kubernetes.io
    bucket: gs://k8s-staging-[PROJECT]
  # gs://k8s-staging-[PROJECT]-gcb
  #
  # IAM bindings differenciate "gs://k8s-staging-[PROJECT]-gcb from "gs://k8s-staging-[PROJECT]"
  # and "gs://artifacts.[PROJECT].appspot.com" only by binding "roles/storage.objectCreator"
  # to "serviceAccount:deployer@k8s-prow.iam.gserviceaccount.com"
  # any by explicitly binding "serviceAccount:deployer@k8s-prow.iam.gserviceaccount.com"
  # as "roles/storage.objectViewer" which I'm not sure is necessary when "allUsers"
  # are bound to "roles/storage.objectViewer" role already.
  #
  # [todo(@bartsmykla)]: check if explicitly binding
  #                      "serviceAccount:deployer@k8s-prow.iam.gserviceaccount.com"
  #                      to "roles/storage.objectViewer" role is necessary here
  - role: roles/storage.objectViewer
    members:
      - allUsers
    bucket: gs://k8s-staging-[PROJECT]-gcb
  - role: roles/storage.objectAdmin
    members:
      - group:k8s-infra-artifact-admins@kubernetes.io
      - group:k8s-infra-staging-[PROJECT]@kubernetes.io
    bucket: gs://k8s-staging-[PROJECT]-gcb
  - role: roles/storage.legacyBucketOwner
    members:
      - group:k8s-infra-release-admins@kubernetes.io
    bucket: gs://k8s-staging-[PROJECT]-gcb
  - role: roles/storage.legacyBucketReader
    members:
      - group:k8s-infra-staging-[PROJECT]@kubernetes.io
    bucket: gs://k8s-staging-[PROJECT]-gcb
  - role: roles/storage.objectCreator
    members:
      - serviceAccount:deployer@k8s-prow.iam.gserviceaccount.com
    bucket: gs://k8s-staging-[PROJECT]-gcb
  - role: roles/storage.objectViewer
    members:
      - serviceAccount:deployer@k8s-prow.iam.gserviceaccount.com
    bucket: gs://k8s-staging-[PROJECT]-gcb
google_storage_bucket_acl:
# we need to discuss if we wan't to manage this resource because as far I'm aware,
# good practice is to use IAMs instead of ACLs, but in this case
# ("legacyBucketOwner" and "legacyBucketReader") the ACLs will be implicitly
# created, so I prefer to put them also here "explicitly".
#
# [IMPORTANT!] be aware that every role entity used in ACLs is in form
#              of type and proper entity separated by "-" (not ":"),
#              so for group "k8s-infra-release-admins@kubernetes.io"
#              it will be "group-k8s-infra-release-admins@kubernetes.io"
  - bucket: gs://artifacts.k8s-staging-[PROJECT].appspot.com
    role_entity:
      - OWNER:group-k8s-infra-release-admins@kubernetes.io
      - READER:group-k8s-infra-staging-[PROJECT]@kubernetes.io
  - bucket: gs://k8s-staging-[PROJECT]
    role_entity:
      - OWNER:group-k8s-infra-release-admins@kubernetes.io
      - READER:group-k8s-infra-staging-[PROJECT]@kubernetes.io
  - bucket: gs://k8s-staging-[PROJECT]-gcb
    role_entity:
      - OWNER:group-k8s-infra-release-admins@kubernetes.io
      - READER:group-k8s-infra-staging-[PROJECT]@kubernetes.io
```

---

#### Conformance Storage

---

##### Terraform resources for Conformance Storage

- Provider: [`Google`](https://www.terraform.io/docs/providers/google/index.html "Provider: Google")
  - [`google_project`](https://www.terraform.io/docs/providers/google/r/google_project.html "Resource: Google Project")
  - [`google_project_service`](https://www.terraform.io/docs/providers/google/r/google_project_service.html "Resource: Google Project Service")
  - [`google_storage_bucket`](https://www.terraform.io/docs/providers/google/r/storage_bucket.html "Resource: Google Storage Bucket")
  - [`google_project_iam_binding`](https://www.terraform.io/docs/providers/google/r/google_project_iam.html "Resource: Google Project IAM Binding")
  - [`google_project_iam_member`](https://www.terraform.io/docs/providers/google/r/google_project_iam.html "Resource: Google Project IAM Member")[<sup>G2</sup>](#global-reference)
  - [`google_storage_bucket_iam_binding`](https://www.terraform.io/docs/providers/google/r/storage_bucket_iam.html "Resource: Google Storage Bucket IAM Binding")
  - [`google_storage_bucket_iam_member`](https://www.terraform.io/docs/providers/google/r/storage_bucket_iam.html "Resource: Google Storage Bucket IAM Member")[<sup>G2</sup>](#global-reference)
  - [`google_storage_bucket_acl`](https://www.terraform.io/docs/providers/google/r/storage_bucket_acl.html "Resource: Google Storage Bucket ACL")

##### Conformance Storage `[BUCKETS]`

- `capi-openstack`
- `cri-o`
- `huaweicloud`

##### Conformance Storage / Components

- **Components for project: `k8s-conform`**:
  - Project:
    - `k8s-conform`
  - API:
    - `storage-component`
  - **Components for project: `k8s-conform` per [`[BUCKET]`](#conformance-storage-buckets)**:
    - GCS Bucket:
      - `gs://k8s-confirm-[BUCKET]`:
        - bucketpolicyonly: `true`
        - location: `us`
        - retention: `10y`
    - IAM:
      - `gs://k8s-confirm-[BUCKET]`:
        - `allUsers:objectViewer`
        - `group:k8s-infra-artifact-admins@kubernetes.io:objectAdmin`
        - `group:k8s-infra-artifact-admins@kubernetes.io:legacyBucketOwner`
        - `group:k8s-infra-conform-[BUCKET]@kubernetes.io:objectAdmin`
        - `group:k8s-infra-conform-[BUCKET]@kubernetes.io:legacyBucketReader`
    - IAM Policy Binding:
      - `roles/viewer`:
        - `group:k8s-infra-artifact-admins@kubernetes.io`

##### Yaml representation of *Conformance Storage*[<sup>G1</sup>](#global-reference),</sup>[<sup>G2</sup>](#global-reference)

```yaml
# [BUCKETS]:
#   - capi-openstack
#   - cri-o
#   - huaweicloud

google_project:
  - name: k8s-conform
google_project_service:
  - service: storage-component.googleapis.com
    project: k8s-conform
google_storage_bucket:
  - name: k8s-confirm-[BUCKET]
    bucket_policy_only: true
    location: us
    retention_policy:
      retention_period: 315360000 # 10 years
google_project_iam_binding:
  - role: roles/viewer
    members:
      - group:k8s-infra-artifact-admins@kubernetes.io
    project: k8s-conform
google_storage_bucket_iam_binding:
  - role: roles/storage.objectViewer
    members:
      - allUsers
    bucket: gs://k8s-confirm-[BUCKET]
  - role: roles/storage.objectAdmin
    members:
      - group:k8s-infra-artifact-admins@kubernetes.io
      - group:k8s-infra-conform-[BUCKET]@kubernetes.io
    bucket: gs://k8s-confirm-[BUCKET]
  - role: roles/storage.legacyBucketOwner
    members:
      - group:k8s-infra-release-admins@kubernetes.io
    bucket: gs://k8s-confirm-[BUCKET]
  - role: roles/storage.legacyBucketReader
    members:
      - group:k8s-infra-conform-[BUCKET]@kubernetes.io
    bucket: gs://k8s-confirm-[BUCKET]
google_storage_bucket_acl:
# we need to discuss if we wan't to manage this resource because as far I'm aware,
# good practice is to use IAMs instead of ACLs, but in this case
# ("legacyBucketOwner" and "legacyBucketReader") the ACLs will be implicitly
# created, so I prefer to put them also here "explicitly".
#
# [IMPORTANT!] be aware that every role entity used in ACLs is in form
#              of type and proper entity separated by "-" (not ":"),
#              so for group "k8s-infra-release-admins@kubernetes.io"
#              it will be "group-k8s-infra-release-admins@kubernetes.io"
  - bucket: gs://k8s-confirm-[BUCKET]
    role_entity:
      - OWNER:group-k8s-infra-release-admins@kubernetes.io
      - READER:group-k8s-infra-conform-[BUCKET]@kubernetes.io
```

---

#### CIP Auditor

---

Even if there are two scripts for CIP (Container Image Promoter) Auditor ([`ensure-env-cip-auditor.sh`](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/ensure-env-cip-auditor.sh) and [`cip-auditor/deploy.sh`](ttps://github.com/kubernetes/k8s.io/blob/master/infra/gcp/cip-auditor/deploy.sh)) I decided to combine them into one section because I think we shouldn't split them.

> [**todo(@listx)**]: I don't undestand why we first create dummy Cloud Run Service in [`ensure-env-cip-auditor.sh`](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/ensure-env-cip-auditor.sh#L160-L164) and then overwriting it in [`cip-auditor/deploy.sh`](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/cip-auditor/deploy.sh#L47-L54). What stands behind the decision to do it that way? Can we eliminate the step for the dummy service?

##### CIP Auditor Env

###### Terraform resources for CIP Auditor Env

- Provider: [`Google`](https://www.terraform.io/docs/providers/google/index.html "Provider: Google")
  - [`google_project_service`](https://www.terraform.io/docs/providers/google/r/google_project_service.html "Resource: Google Project Service")
  - [`google_cloud_run_service`](https://www.terraform.io/docs/providers/google/r/cloud_run_service.html "Resource: Google Cloud Run Service")
  - [`google_pubsub_topic`](https://www.terraform.io/docs/providers/google/r/pubsub_topic.html "Resource: Google PubSub Topic")
  - [`google_pubsub_subscription`](https://www.terraform.io/docs/providers/google/r/pubsub_subscription.html "Resource: Google PubSub Subscription")
  - [`google_project_iam_binding`](https://www.terraform.io/docs/providers/google/r/google_project_iam.html "Resource: Google Project IAM Binding")

###### CIP Auditor Env / Components

- Components for project `k8s-artifacts-prod`:
  
  > Atribute: `[PROJECT_NUMBER]` will be taken from the project resource attributes reference
  >
  > [**todo(@bartsmykla)**]: Put some terraform example maybe?

  - Service:
    - `serviceusage.googleapis.com`
    - `cloudresourcemanager.googleapis.com`
    - `stackdriver.googleapis.com`
    - `clouderrorreporting.googleapis.com`
    - `run.googleapis.com`
  - Cloud Run Service[<sup>1</sup>](#reference-for-cip-auditor "Example of terraform component for cloud run service"):
    - `cip-auditor`:
      - image: `gcr.io/cloudrun/hello`
      - platform: `managed`
      - no_allow_unauthenticated: `true`
      - region: `us-central1`
      - service_account: `serviceAccount:k8s-infra-gcr-auditor-invoker@k8s-artifacts-prod.iam.gserviceaccount.com`
  - PubSub Topic:
    - `gcr`
  - IAM Policy Binding:
    - `roles/iam.serviceAccountTokenCreator`:
      - `serviceAccount:service-[PROJECT_NUMBER]@gcp-sa-pubsub.iam.gserviceaccount.com`
  - PubSub Subscripion:
    - `cip-auditor-invoker`:
      - topic: `gcr`
      - expiration_policy: `never`
      - push_endpoint: `[CLOUD_RUN_SERVICE.cip-auditor.ENDPOINT]`
      - push_auth_service_account: `serviceAccount:k8s-infra-gcr-auditor-invoker@k8s-artifacts-prod.iam.gserviceaccount.com`[<sup>2</sup>](#reference-for-cip-auditor)

###### Yaml representation of *CIP Auditor Env Components*[<sup>G1</sup>](#global-reference),</sup>[<sup>G2</sup>](#global-reference)

```yaml
google_project_service:
  - service: serviceusage.googleapis.com
    project: k8s-artifacts-prod
  - service: cloudresourcemanager.googleapis.com
    project: k8s-artifacts-prod
  - service: stackdriver.googleapis.com
    project: k8s-artifacts-prod
  - service: clouderrorreporting.googleapis.com
    project: k8s-artifacts-prod
  - service: run.googleapis.com
    project: k8s-artifacts-prod
google_cloud_run_service:
  - name: cip-auditor
    location: us-central1
    template:
      spec:
        service_account_name: k8s-infra-gcr-auditor-invoker@k8s-artifacts-prod.iam.gserviceaccount.com
        containers:
          - image: gcr.io/cloudrun/hello
google_pubsub_topic:
  - name: gcr
google_pubsub_subscription:
  - name: cip-auditor-invoker
    topic: gcr
    expiration_policy:
      ttl: ""
    push_config:
      push_endpoint: [CLOUD_RUN_SERVICE.cip-auditor.ENDPOINT]
    oidc_token:
      service_account_email: k8s-infra-gcr-auditor-invoker@k8s-artifacts-prod.iam.gserviceaccount.com
google_project_iam_binding:
  - role: roles/iam.serviceAccountTokenCreator
    members:
      # Atribute: `[PROJECT_NUMBER]` will be taken from the project resource attributes reference
      - serviceAccount:service-[PROJECT_NUMBER]@gcp-sa-pubsub.iam.gserviceaccount.com
    project: k8s-artifacts-prod
```

##### CIP Auditor Deploy

> [**todo(@listx)**]: Do we need the `[PROJECT]` to be a variable? Is CIP Auditor currently deployed for other than `k8s-artifacts-prod` projects too?

###### Terraform resources for CIP Auditor Deploy

- Provider: [`Google`](https://www.terraform.io/docs/providers/google/index.html "Provider: Google")
  - [`google_cloud_run_service`](https://www.terraform.io/docs/providers/google/r/cloud_run_service.html "Resource: Google Cloud Run Service")

###### CIP Auditor Deploy / Components

- Components for `[PROJECT]` ([currently provided via CLI argument](https://github.com/kubernetes/k8s.io/blob/3ec33f49bf4181f355e30d3871174a79cda3aad2/infra/gcp/cip-auditor/deploy.sh#L75))
  - Cloud Run Service[<sup>1</sup>](#reference-for-cip-auditor "Example of terraform component for cloud run service"):

    > [`[CIP_AUDITOR_DIGEST]` is currently provided via CLI argument](https://github.com/kubernetes/k8s.io/blob/3ec33f49bf4181f355e30d3871174a79cda3aad2/infra/gcp/cip-auditor/deploy.sh#L76)

    - `cip-auditor`:
      - image: `us.gcr.io/k8s-artifacts-prod/artifact-promoter/cip-auditor@sha256:[CIP_AUDITOR_DIGEST]`
      - platform: `managed`
      - no_allow_unauthenticated: `true`
      - region: `us-central1`
      - service_account: `serviceAccount:k8s-infra-gcr-auditor-invoker@[PROJECT].iam.gserviceaccount.com`
      - env_vars:
        - `CIP_AUDIT_MANIFEST_REPO_URL` : `https://github.com/kubernetes/k8s.io`
        - `CIP_AUDIT_MANIFEST_REPO_BRANCH` : `master`
        - `CIP_AUDIT_MANIFEST_REPO_MANIFEST_DIR` : `k8s.gcr.io`
        - `CIP_AUDIT_GCP_PROJECT_ID` : `k8s-artifacts-prod`

###### Yaml representation of *CIP Auditor Deploy Components*[<sup>G1</sup>](#global-reference)

```yaml
google_cloud_run_service:
  - name: cip-auditor
    location: us-central1
    template:
      spec:
        service_account_name: k8s-infra-gcr-auditor-invoker@[PROJECT].iam.gserviceaccount.com
        containers:
          - image: us.gcr.io/k8s-artifacts-prod/artifact-promoter/cip-auditor@sha256:[CIP_AUDITOR_DIGEST]
            env:
              - name: CIP_AUDIT_MANIFEST_REPO_URL
                value: https://github.com/kubernetes/k8s.io
              - name: CIP_AUDIT_MANIFEST_REPO_BRANCH
                value: master
              - name: CIP_AUDIT_MANIFEST_REPO_MANIFEST_DIR
                value: k8s.gcr.io
              - name: CIP_AUDIT_GCP_PROJECT_ID
                value: k8s-artifacts-prod
```

##### Reference for CIP Auditor

- <sup>1</sup> Example of fow to create Google Cloud Run Service component in terraform[<sup>3</sup>](#reference-for-cip-auditor)

  > [**todo(@bartsmykla)**]: Verify it works because I wrote it manually without real testing

  ```terraform
  resource "google_cloud_run_service" "cip_auditor" {
    name     = "cip-auditor"
    location = "us-central1"

    template {
      spec {
        containers {
          image = "us.gcr.io/k8s-artifacts-prod/artifact-promoter/cip-auditor@sha256:[CIP_AUDITOR_DIGEST]"
          env {
            name = "CIP_AUDIT_MANIFEST_REPO_URL"
            value = "https://github.com/kubernetes/k8s.io"
          }
          env {
            name = "CIP_AUDIT_MANIFEST_REPO_BRANCH"
            value = "master"
          }
          env {
            name = "CIP_AUDIT_MANIFEST_REPO_MANIFEST_DIR"
            value = "k8s.gcr.io"
          }
          env {
            name = "CIP_AUDIT_GCP_PROJECT_ID"
            value = "k8s-artifacts-prod"
          }
        }
        service_account_name = "k8s-infra-gcr-auditor-invoker@k8s-artifacts-prod.iam.gserviceaccount.com"
      }
    }
  }
  ```

- <sup>2</sup> In terraform resource it's achieved by argument `google_pubsub_subscription.push_config.oidc_token.service_account_email`

  > [**todo(@bartsmykla)**]: Verify it works because I wrote it manually without real testing

  ```terraform
  resource "google_pubsub_topic" "gcr" {
    name = "gcr"
  }

  resource "google_pubsub_subscription" "cip_auditor_invoker" {
    name  = "cip-auditor-invoker"
    topic = google_pubsub_topic.gcr.name

    expiration_policy {
      ttl = ""
    }

    push_config {
      push_endpoint = google_cloud_run_service.cip_auditor.status.url

      oidc_token = {
        service_account_email = "k8s-infra-gcr-auditor-invoker@k8s-artifacts-prod.iam.gserviceaccount.com"
      }
    }
  }
  ```

- <sup>3</sup> Maybe we can take the `[CIP_AUDITOR_DIGEST]` from the data source: `[google_container_registry_image](https://www.terraform.io/docs/providers/google/d/google_container_registry_image.html)`

---

#### Release Projects

---

##### Terraform resources for Release Projects

- Provider: [`Google`](https://www.terraform.io/docs/providers/google/index.html "Provider: Google")
  - [`google_project`](https://www.terraform.io/docs/providers/google/r/google_project.html "Resource: Google Project")
  - [`google_project_service`](https://www.terraform.io/docs/providers/google/r/google_project_service.html "Resource: Google Project Service")
  - [`google_container_registry`](https://www.terraform.io/docs/providers/google/r/container_registry.html "Resource: Google Container Registry")
  - [`google_storage_bucket`](https://www.terraform.io/docs/providers/google/r/storage_bucket.html "Resource: Google Storage Bucket")
  - [`google_project_iam_binding`](https://www.terraform.io/docs/providers/google/r/google_project_iam.html "Resource: Google Project IAM Binding")
  - [`google_project_iam_member`](https://www.terraform.io/docs/providers/google/r/google_project_iam.html "Resource: Google Project IAM Member")[<sup>G2</sup>](#global-reference)
  - [`google_storage_bucket_iam_binding`](https://www.terraform.io/docs/providers/google/r/storage_bucket_iam.html "Resource: Google Storage Bucket IAM Binding")
  - [`google_storage_bucket_iam_member`](https://www.terraform.io/docs/providers/google/r/storage_bucket_iam.html "Resource: Google Storage Bucket IAM Member")[<sup>G2</sup>](#global-reference)
  - [`google_storage_bucket_acl`](https://www.terraform.io/docs/providers/google/r/storage_bucket_acl.html "Resource: Google Storage Bucket ACL")

##### Release Projects `[PROJECTS]`

- `k8s-staging-release-test`
- `k8s-release-test-prod`

##### Release Projects / Components

- **Components per [`[PROJECT]`](#release-projects-projects)**:
  - Project:
    - `[PROJECT]`
  - IAM Policy Binding:
    - `roles/viewer`:
      - `group:k8s-infra-release-admins@kubernetes.io`
      - `group:k8s-infra-release-editors@kubernetes.io`
      - `group:k8s-infra-release-viewers@kubernetes.io`
      - `group:k8s-infra-artifact-admins@kubernetes.io`
    - `roles/cloudbuild.builds.editor`:
      - `group:k8s-infra-release-admins@kubernetes.io`
      - `group:k8s-infra-release-editors@kubernetes.io`
    - `roles/serviceusage.serviceUsageConsumer`[<sup>1</sup>](#reference-for-release-projects "Comment about temporary nature of this role here"):
      - `group:k8s-infra-release-admins@kubernetes.io`
      - `group:k8s-infra-release-editors@kubernetes.io`
    - `roles/cloudbuild.builds.builder`:
      - `serviceAccount:deployer@k8s-prow.iam.gserviceaccount.com`
    - `roles/cloudkms.admin`:
      - `group:k8s-infra-release-admins@kubernetes.io`
    - `roles/cloudkms.cryptoKeyEncrypterDecrypter`:
      - `group:k8s-infra-release-admins@kubernetes.io`
  - API:
    - `containerregistry`
    - `storage-component`
    - `cloudbuild`
    - `cloudkms`
  - GCR:
    - `[PROJECT]`
  - IAM:
    - `gs://artifacts.[PROJECT].appspot.com`
      - `allUsers:objectViewer`
      - `group:k8s-infra-artifact-admins@kubernetes.io:objectAdmin`
      - `group:k8s-infra-artifact-admins@kubernetes.io:legacyBucketOwner`
      - `group:k8s-infra-release-admins@kubernetes.io:objectAdmin`
      - `group:k8s-infra-release-admins@kubernetes.io:legacyBucketReader`
      - `group:k8s-infra-release-editors@kubernetes.io:objectAdmin`
      - `group:k8s-infra-release-editors@kubernetes.io:legacyBucketReader`
    - `gs://[PROJECT]`:
      - `allUsers:objectViewer`
      - `group:k8s-infra-artifact-admins@kubernetes.io:objectAdmin`
      - `group:k8s-infra-artifact-admins@kubernetes.io:legacyBucketOwner`
      - `group:k8s-infra-release-admins@kubernetes.io:objectAdmin`
      - `group:k8s-infra-release-admins@kubernetes.io:legacyBucketReader`
      - `group:k8s-infra-release-editors@kubernetes.io:objectAdmin`
      - `group:k8s-infra-release-editors@kubernetes.io:legacyBucketReader`
    - `gs://[PROJECT]-gcb`:
      - `allUsers:objectViewer`
      - `group:k8s-infra-artifact-admins@kubernetes.io:objectAdmin`
      - `group:k8s-infra-artifact-admins@kubernetes.io:legacyBucketOwner`
      - `group:k8s-infra-release-admins@kubernetes.io:objectAdmin`
      - `group:k8s-infra-release-admins@kubernetes.io:legacyBucketReader`
      - `group:k8s-infra-release-editors@kubernetes.io:objectAdmin`
      - `group:k8s-infra-release-editors@kubernetes.io:legacyBucketReader`
      - `serviceAccount:deployer@k8s-prow.iam.gserviceaccount.com:objectCreator`
      - `serviceAccount:deployer@k8s-prow.iam.gserviceaccount.com:objectViewer`
  - GCS Bucket:
    - `gs://artifacts.[PROJECT].appspot.com`:
      - bucketpolicyonly: `true`
    - `gs://[PROJECT]`:
      - bucketpolicyonly: `true`
      - location: `us`
    - `gs://[PROJECT]-gcb`:
      - bucketpolicyonly: `true`
      - location: `us`

##### Yaml representation of *Release Projects Components*[<sup>G1</sup>](#global-reference)<sup>,</sup>[<sup>G2</sup>](#global-reference)

```yaml
# [PROJECTS]:
#   - k8s-staging-release-test
#   - k8s-release-test-prod

google_project:
  - name: [PROJECT]
google_project_service:
  - service: containerregistry.googleapis.com
    project: [PROJECT]
  - service: storage-component.googleapis.com
    project: [PROJECT]
  - service: cloudbuild.googleapis.com
    project: [PROJECT]
  - service: cloudkms.googleapis.com
    project: [PROJECT]
google_container_registry:
  - project: [PROJECT]
google_storage_bucket:
  - name: artifacts.[PROJECT].appspot.com
    bucket_policy_only: true
  - name: [PROJECT]
    location: us
    bucket_policy_only: true
  - name: "[PROJECT]-gcb"
    location: us
    bucket_policy_only: true
google_project_iam_binding:
  - role: roles/viewer
    members:
      - group:k8s-infra-release-admins@kubernetes.io
      - group:k8s-infra-release-editors@kubernetes.io
      - group:k8s-infra-release-viewers@kubernetes.io
      - group:k8s-infra-artifact-admins@kubernetes.io
    project: [PROJECT]
  - role: roles/cloudbuild.builds.editor
    members:
      - group:k8s-infra-release-admins@kubernetes.io
      - group:k8s-infra-release-editors@kubernetes.io
    project: [PROJECT]
  - role: roles/serviceusage.serviceUsageConsumer
    members:
      - group:k8s-infra-release-admins@kubernetes.io
      - group:k8s-infra-release-editors@kubernetes.io
    project: [PROJECT]
  - role: roles/cloudbuild.builds.builder
    members:
      - serviceAccount:deployer@k8s-prow.iam.gserviceaccount.com
    project: [PROJECT]
  - role: roles/cloudkms.admin
    members:
      - group:k8s-infra-release-admins@kubernetes.io
    project: [PROJECT]
  - role: roles/cloudkms.cryptoKeyEncrypterDecrypter
    members:
      - group:k8s-infra-release-admins@kubernetes.io
    project: [PROJECT]
google_storage_bucket_iam_binding:
  # gs://artifacts.[PROJECT].appspot.com
  - role: roles/storage.objectViewer
    members:
      - allUsers
    bucket: gs://artifacts.[PROJECT].appspot.com
  - role: roles/storage.objectAdmin
    members:
      - group:k8s-infra-artifact-admins@kubernetes.io
      - group:k8s-infra-release-admins@kubernetes.io
      - group:k8s-infra-release-editors@kubernetes.io
    bucket: gs://artifacts.[PROJECT].appspot.com
  - role: roles/storage.legacyBucketOwner
    members:
      - group:k8s-infra-release-admins@kubernetes.io
    bucket: gs://artifacts.[PROJECT].appspot.com
  - role: roles/storage.legacyBucketReader
    members:
      - group:k8s-infra-release-admins@kubernetes.io
      - group:k8s-infra-release-editors@kubernetes.io
    bucket: gs://artifacts.[PROJECT].appspot.com
  # gs://[PROJECT]
  #
  # bindings are exactly the same as in gs://artifacts.[PROJECT].appspot.com
  - role: roles/storage.objectViewer
    members:
      - allUsers
    bucket: gs://[PROJECT]
  - role: roles/storage.objectAdmin
    members:
      - group:k8s-infra-artifact-admins@kubernetes.io
      - group:k8s-infra-release-admins@kubernetes.io
      - group:k8s-infra-release-editors@kubernetes.io
    bucket: gs://[PROJECT]
  - role: roles/storage.legacyBucketOwner
    members:
      - group:k8s-infra-release-admins@kubernetes.io
    bucket: gs://[PROJECT]
  - role: roles/storage.legacyBucketReader
    members:
      - group:k8s-infra-release-admins@kubernetes.io
      - group:k8s-infra-release-editors@kubernetes.io
    bucket: gs://[PROJECT]
  # gs://[PROJECT]-gcb
  #
  # IAM bindings differenciate "gs://[PROJECT]-gcb" from "gs://[PROJECT]"
  # and "gs://artifacts.[PROJECT].appspot.com" only by binding "roles/storage.objectCreator"
  # to "serviceAccount:deployer@k8s-prow.iam.gserviceaccount.com"
  # any by explicitly binding "serviceAccount:deployer@k8s-prow.iam.gserviceaccount.com"
  # as "roles/storage.objectViewer" which I'm not sure is necessary when "allUsers"
  # are bound to "roles/storage.objectViewer" role already.
  #
  # [todo(@bartsmykla)]: check if explicitly binding
  #                      "serviceAccount:deployer@k8s-prow.iam.gserviceaccount.com"
  #                      to "roles/storage.objectViewer" role is necessary here
  - role: roles/storage.objectViewer
    members:
      - allUsers
      - serviceAccount:deployer@k8s-prow.iam.gserviceaccount.com
    bucket: gs://[PROJECT]-gcb
  - role: roles/storage.objectAdmin
    members:
      - group:k8s-infra-artifact-admins@kubernetes.io
      - group:k8s-infra-release-admins@kubernetes.io
      - group:k8s-infra-release-editors@kubernetes.io
    bucket: gs://[PROJECT]-gcb
  - role: roles/storage.objectCreator
    members:
      - serviceAccount:deployer@k8s-prow.iam.gserviceaccount.com
    bucket: gs://[PROJECT]-gcb
  - role: roles/storage.legacyBucketOwner
    members:
      - group:k8s-infra-release-admins@kubernetes.io
    bucket: gs://[PROJECT]-gcb
  - role: roles/storage.legacyBucketReader
    members:
      - group:k8s-infra-release-admins@kubernetes.io
      - group:k8s-infra-release-editors@kubernetes.io
    bucket: gs://[PROJECT]-gcb
google_storage_bucket_acl:
# we need to discuss if we wan't to manage this resource because as far I'm aware,
# good practice is to use IAMs instead of ACLs, but in this case
# ("legacyBucketOwner" and "legacyBucketReader") the ACLs will be implicitly
# created, so I prefer to put them also here "explicitly".
#
# [IMPORTANT!] be aware that every role entity used in ACLs is in form
#              of type and proper entity separated by "-" (not ":"),
#              so for group "k8s-infra-release-admins@kubernetes.io"
#              it will be "group-k8s-infra-release-admins@kubernetes.io"
  - bucket: gs://artifacts.[PROJECT].appspot.com
    role_entity:
      - OWNER:group-k8s-infra-release-admins@kubernetes.io
      - READER:group-k8s-infra-release-admins@kubernetes.io
      - READER:group-k8s-infra-release-editors@kubernetes.io
  - bucket: gs://[PROJECT]
    role_entity:
      - OWNER:group-k8s-infra-release-admins@kubernetes.io
      - READER:group-k8s-infra-release-admins@kubernetes.io
      - READER:group-k8s-infra-release-editors@kubernetes.io
  - bucket: gs://[PROJECT]-gcb
    role_entity:
      - OWNER:group-k8s-infra-release-admins@kubernetes.io
      - READER:group-k8s-infra-release-admins@kubernetes.io
      - READER:group-k8s-infra-release-editors@kubernetes.io
```

##### Reference for Release Projects

- <sup>1</sup> [There is a comment from @justaugustus](https://github.com/kubernetes/k8s.io/blob/9e17cdf48d4e9f343e0a11ecb06247897a81dd84/infra/gcp/lib.sh#L316-L320) it's temporary and we should refactor this once we develop custom roles

---

#### Releng

---

##### Releng / Components

##### Terraform resources for Releng

- Provider: [`Google`](https://www.terraform.io/docs/providers/google/index.html "Provider: Google")
  - [`google_project`](https://www.terraform.io/docs/providers/google/r/google_project.html "Resource: Google Project")
  - [`google_project_service`](https://www.terraform.io/docs/providers/google/r/google_project_service.html "Resource: Google Project Service")
  - [`google_project_iam_binding`](https://www.terraform.io/docs/providers/google/r/google_project_iam.html "Resource: Google Project IAM Binding")
  - [`google_project_iam_member`](https://www.terraform.io/docs/providers/google/r/google_project_iam.html "Resource: Google Project IAM Member")[<sup>G2</sup>](#global-reference)

- **Components for project: `k8s-releng-prod`**:
  - Project:
    - `k8s-releng-prod`
  - IAM Policy Binding:
    - `roles/viewer`:
      - `group:k8s-infra-release-admins@kubernetes.io`
    - `roles/cloudkms.admin`:
      - `group:k8s-infra-release-admins@kubernetes.io`
    - `roles/cloudkms.cryptoKeyEncrypterDecrypter`:
      - `group:k8s-infra-release-admins@kubernetes.io`
  - API:
    - `cloudkms`

##### Yaml representation of *Releng Components*[<sup>G1</sup>](#global-reference)<sup>,</sup>[<sup>G2</sup>](#global-reference)

```yaml
google_project:
  - name: k8s-releng-prod
google_project_iam_binding:
  - role: roles/viewer
    members:
      - group:k8s-infra-release-admins@kubernetes.io
    project: k8s-releng-prod
  - role: roles/cloudkms.admin
    members:
      - group:k8s-infra-release-admins@kubernetes.io
    project: k8s-releng-prod
  - role: roles/cloudkms.cryptoKeyEncrypterDecrypter
    members:
      - group:k8s-infra-release-admins@kubernetes.io
    project: k8s-releng-prod
google_project_service:
  - service: cloudkms.googleapis.com
    project: k8s-gsuite
```

---

#### GSuite

---

##### Terraform resources for GSuite

- Provider: [`Google`](https://www.terraform.io/docs/providers/google/index.html "Provider: Google")
  - [`google_project`](https://www.terraform.io/docs/providers/google/r/google_project.html "Resource: Google Project")
  - [`google_project_service`](https://www.terraform.io/docs/providers/google/r/google_project_service.html "Resource: Google Project Service")
  - [`google_service_account`](https://www.terraform.io/docs/providers/google/r/google_service_account.html "Resource: Google Service Account")
  - [`google_project_iam_binding`](https://www.terraform.io/docs/providers/google/r/google_project_iam.html "Resource: Google Project IAM Binding")
  - [`google_project_iam_member`](https://www.terraform.io/docs/providers/google/r/google_project_iam.html "Resource: Google Project IAM Member")[<sup>G2</sup>](#global-reference)

##### GSuite / Components

- **Components for project: `k8s-gsuite`**[<sup>1</sup>](#reference-for-gsuite):
  - Project:
    - `k8s-gsuite`
  - API:
    - `admin`
    - `groupssettings`
  - IAM Service Account:
    - `gsuite-groups-manager`:
      - display_name: `Grants access to the googlegroups API in kubernetes.io GSuite`
  - IAM Policy Binding:
    - `roles/owner`:
      - `user:wg-k8s-infra-api@kubernetes.io`

##### Yaml representation of *GSuite Components*[<sup>G1</sup>](#global-reference)<sup>,</sup>[<sup>G2</sup>](#global-reference)

```yaml
google_project:
  - name: k8s-gsuite
google_project_service:
  - service: admin.googleapis.com
    project: k8s-gsuite
  - service: groupssettings.googleapis.com
    project: k8s-gsuite
google_service_account:
  - account_id: gsuite-groups-manager
    display_name: Grants access to the googlegroups API in kubernetes.io GSuite
    project: k8s-gsuite
google_project_iam_binding:
  - role: roles/owner
    members:
      - user:wg-k8s-infra-api@kubernetes.io
    project: k8s-gsuite
```

##### Reference for GSuite

- <sup>1</sup> [At the end of script for provisioning GSuite resources](https://github.com/kubernetes/k8s.io/blob/9e17cdf48d4e9f343e0a11ecb06247897a81dd84/infra/gcp/ensure-gsuite.sh#L72-L86) there is mention about some manual tasks human needs to do to grant GSuite's service account access (it also needs to be acknowdleged by the human who runs the script by pressing enter)

---

#### Namespaces

---

##### Terraform resources for Namespaces

- Provider: [`Kubernetes`](https://www.terraform.io/docs/providers/kubernetes/index.html "Provider: Kubernetes")
  - [`kubernetes_namespace`](https://www.terraform.io/docs/providers/kubernetes/r/namespace.html "Resource: Kubernetes Namespace")
  - [`kubernetes_role`](https://www.terraform.io/docs/providers/kubernetes/r/role.html "Resource: Kubernetes Role")
  - [`kubernetes_role_binding`](https://www.terraform.io/docs/providers/kubernetes/r/role_binding.html "Resource: Kubernetes Role Binding")

##### Namespaces `[PROJECTS]`

- `gcsweb`
- `publishing-bot`
- `k8s-io-prod`
- `k8s-io-canary`

##### Namespaces / Components

- **Components per [[PROJECT]](#namespaces-projects)**:
  - [Kubernetes Namespace](https://www.terraform.io/docs/providers/kubernetes/r/namespace.html "Resource: Kubernetes Namespace"):
    - `[PROJECT]`
  - [Kubernetes Role](https://www.terraform.io/docs/providers/kubernetes/r/role.html "Resource: Kubernetes Role"):
    - `namespace-user`:
      - namespace: `[PROJECT]`
      - rules:
        - `""`:
          - resources:
            - `configmaps`
            - `endpoints`
            - `persistentvolumeclaims`
            - `pods`
            - `resourcequotas`
            - `services`
          - verbs:
            - `*`
        - `""`:
          - resources:
            - `secrets`
          - verbs:
            - `list`
        - `""`:
          - resources:
            - `events`
          - verbs:
            - `get`
            - `list`
        - `certmanager.k8s.io`:
          - resources:
            - `certificates`
          - verbs:
            - `*`
        - `coordination.k8s.io`:
          - resources:
            - `leases`
          - verbs:
            - `*`
        - `batch`:
          - resources:
            - `cronjobs`
            - `jobs`
          - verbs:
            - `*`
        - `autoscaling`:
          - resources:
            - `horizontalpodautoscalers`
          - verbs:
            - `*`
        - `apps`:
          - resources:
            - `deployments`
          - verbs:
            - `*`
        - `extensions`:
          - resources:
            - `deployments`
            - `ingresses`
            - `networkpolicies`
          - verbs:
            - `*`
        - `networking.k8s.io`:
          - resources:
            - `networkpolicies`
          - verbs:
            - `*`
        - `storage.k8s.io`:
          - resources:
            - `storageclasses`
          - verbs:
            - `list`
        - `scheduling.k8s.io`:
          - resources:
            - `priorityclasses`
          - verbs:
            - `list`
        - `rbac.authorization.k8s.io`:
          - resources:
            - `clusterrolebindings`
            - `clusterroles`
            - `rolebindings`
            - `roles`
          - verbs:
            - `get`
            - `list`
  - [Kubernetes Role Binding](https://www.terraform.io/docs/providers/kubernetes/r/role_binding.html "Resource: Kubernetes Role Binding"):
    - `namespace-user`:
      - namespace: `[PROJECT]`
      - subjects:
        - `k8s-infra-rbac-[PROJECT]@kubernetes.io`:
          - kind: `Group`
      - role_ref:
        - name: `namespace-user`
        - kind: `Role`
        - api_group: `rbac.authorization.k8s.io`

##### Yaml representation of *Namespaces Components*[<sup>G1</sup>](#global-reference)

```yaml
kubernetes_namespace:
  - [PROJECT]
kubernetes_role:
  - metadata:
      name: namespace-user
      namespace: [PROJECT]
    rules:
      - api_groups:
          - ""
        resources:
          - configmaps
          - endpoints
          - persistentvolumeclaims
          - pods
          - resourcequotas
          - services
        verbs:
          - "*"
      - api_groups:
          - ""
        resources:
          - secrets
        verbs:
          - list
      - api_groups:
          - ""
        resources:
          - events
        verbs:
          - get
          - list
      - api_groups:
          - certmanager.k8s.io
        resources:
          - certificates
        verbs:
          - "*"
      - api_groups:
          - coordination.k8s.io
        resources:
          - leases
        verbs:
          - "*"
      - api_groups:
          - batch
        resources:
          - cronjobs
          - jobs
        verbs:
          - "*"
      - api_groups:
          - autoscaling
        resources:
          - horizontalpodautoscalers
        verbs:
          - "*"
      - api_groups:
          - apps
        resources:
          - deployments
        verbs:
          - "*"
      - api_groups:
          - extensions
        resources:
          - deployments
          - ingresses
          - networkpolicies
        verbs:
          - "*"
      - api_groups:
          - networking.k8s.io
        resources:
          - networkpolicies
        verbs:
          - "*"
      - api_groups:
          - storage.k8s.io
        resources:
          - storageclasses
        verbs:
          - list
      - api_groups:
          - scheduling.k8s.io
        resources:
          - priorityclasses
        verbs:
          - list
      - api_groups:
          - rbac.authorization.k8s.io
        resources:
          - clusterrolebindings
          - clusterroles
          - rolebindings
          - roles
        verbs:
          - get
          - list
kubernetes_role_binding:
  - metadata:
    name: namespace-user
    namespace: [PROJECT]
    subjects:
      - name: k8s-infra-rbac-[PROJECT]@kubernetes.io
        kind: Group
    role_ref:
      name: namespace-user
      kind: Role
      api_group: rbac.authorization.k8s.io
```

---

## Proposed structure of new tooling

## Plan of work

## FAQ

### Why terraform?

### How can I help?

## todos

- @listx:
  - > [[**todo(@listx)**]: I don't undestand why we first create dummy Cloud Run Service in `ensure-env-cip-auditor.sh` and then overwriting it in `cip-auditor/deploy.sh`. What stands behind the decision to do it that way? Can we eliminate the step for the dummy service?](#cip-auditor)
  - > [[**todo(@listx)**]: Do we really need the `[PROJECT]` to be a variable? Is CIP Auditor currently deployed for other than `k8s-artifacts-prod` projects too?](#cip-auditor-deploy)

## Global Reference

- <sup>G1</sup> The name of components and structure of Yaml was used in conjuction to terraform components
- <sup>G2</sup> Remember the IAM Binding resources used are authoritative (they will remove all other IAM Bindings from the resource) and are used here just as an example (they are easier to read). When components needed in our infrastructure will be described with different archetypes in mind ("project" will be an archetype instead of i.e. "prod storage") we should write terraform modules in that maner, that by default archetypes will get some IAMs granted by authoritative IAM Bindings and if archetype needs unusuall ones they will by granted by IAM Member resources which are non-authoritative. (for more information you can look for example at notes of [Terraform's Google Storage Bucket IAM resource documentation](https://www.terraform.io/docs/providers/google/r/storage_bucket_iam.html).
