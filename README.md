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
      - [Components](#components)
    - [Prod Storage](#prod-storage)
      - [Prod Storage `[PROJECTS]`](#prod-storage-projects)
      - [Prod Storage `[REGIONS]`](#prod-storage-regions)
      - [Prod Storage / Components](#prod-storage--components)
      - [Notes for Prod Storage](#notes-for-prod-storage)
      - [Reference for Prod Storage](#reference-for-prod-storage)
    - [Prod Storage GCLB](#prod-storage-gclb)
      - [Prod Storage GCLB / Components](#prod-storage-gclb--components)
      - [Reference for Prod Storage GCLB](#reference-for-prod-storage-gclb)
    - [Staging Storage](#staging-storage)
      - [What components are needed for each [PROJECT]](#what-components-are-needed-for-each-project)
      - [What components are needed for each of RELEASE_STAGING_PROJECTS [RS_PROJECT]](#what-components-are-needed-for-each-of-release_staging_projects-rs_project)
      - [What components are needed for project: k8s-staging-release-test](#what-components-are-needed-for-project-k8s-staging-release-test)
    - [CIP Auditor](#cip-auditor)
      - [Env CIP Auditor](#env-cip-auditor)
        - [Env CIP Auditor / Components](#env-cip-auditor--components)
      - [CIP Auditor Deploy](#cip-auditor-deploy)
        - [CIP Auditor Deploy / Components](#cip-auditor-deploy--components)
      - [Reference for CIP Auditor](#reference-for-cip-auditor)
    - [Release Projects](#release-projects)
      - [Release Projects `[PROJECTS]`](#release-projects-projects)
      - [Release Projects / Components](#release-projects--components)
      - [Reference for Release Projects](#reference-for-release-projects)
- [Proposed structure of new tooling](#proposed-structure-of-new-tooling)
- [Plan of work](#plan-of-work)
- [FAQ](#faq)
  - [Why terraform?](#why-terraform)
  - [How can I help?](#how-can-i-help)
- [todos](#todos)

---

## Preface

I strongly believe in rule that before you'll start solving the problem with the code you should understand well what is the problem and solve it conceptualy first.

[IMPORTANT!] As you can assume, I've spent some time working on this document. I come to some conclusions and have some thoughts and ideas but they are just MY conclusions and just MY foughts and ideas. PLEASE don't feel obliged to agree with me just because I spent my time working on it. If you think I'm wrong or/and you think we could improve it PLEASE write about it. Even if it would completely scrap my ideas/suggestions/plans. Let's treat it as a working document and have fruitful discussion.

### tl;dr

### How to read this document

### Why I created this document

### What problem are we trying to solve

## Current state of the tools

### Flow chart

### Components list

### Components diagram

### Analysis

What I don't like is there are places like CIP Auditor which need some scripts being run in a specified sequence for it to be able to correctly provision resources. We need to make it more atomic.

#### Main Project

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

  > [**@bartsmykla**]: In [current code](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/ensure-main-project.sh#L146-L177) we are appending three roles below to already existing ones. We need to make sure we know about every other role and explicitly pur them in terraform or if they are the default roles if terraform will apply them by default
  >
  > [**@bartsmykla**]: Currently [at the end of the script which is provisioning resources for main project](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/ensure-main-project.sh#L179-L192) there is a comment and mechanism to acknowledging the knowledge about having to log in to the cloud console by human to enable billing export
  
  - `kubernetes_public_billing`:
    - `access`:
      - `groupByEmail:k8s-infra-gcp-accounting@kubernetes.io"`:
        - `READER`
        - `roles/bigquery.metadataViewer`
        - `roles/bigquery.user`

---

#### Prod Storage

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

- **Components per [[PROJECT]](#prod-storage-projects)**:
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
      - project: `[PROJECT]`
      - display_name: `k8s-infra container image promoter`

  - **Components per [[PROJECT]](#prod-storage-projects) per [[REGION]](#prod-storage-regions)**:

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
      - website[<sup>1</sup>](#reference "Special case"):
        - main_page_suffix: `index.html`
    - `gs://k8s-artifacts-cni`[<sup>2</sup>](#reference "Special case"):
      - location: `us`
      - bucketpolicyonly: `true`
      - retention: `10y`
  - IAM:
    - `gs://k8s-artifacts-cni`[<sup>2</sup>](#reference "Special case"):
      - `allUsers:objectViewer`
      - `group:k8s-infra-artifact-admins@kubernetes.io:objectAdmin`
      - `group:k8s-infra-artifact-admins@kubernetes.io:legacyBucketOwner`
      - `group:k8s-infra-push-cni@kubernetes.io:objectAdmin`
      - `group:k8s-infra-push-cni@kubernetes.io:legacyBucketReader`
  - IAM Policy Binding:
    - `roles/run.admin`[<sup>10</sup>](#reference "Special case"):
      - `group:k8s-infra-artifact-admins@kubernetes.io`
    - `roles/serviceusage.serviceUsageConsumer`[<sup>10</sup>](#reference "Special case"):
      - `group:k8s-infra-artifact-admins@kubernetes.io`
    - `roles/logging.logWriter`[<sup>11</sup>](#reference "Special case"):
      - `k8s-infra-gcr-auditor@k8s-artifacts-prod.iam.gserviceaccount.com`
    - `roles/errorreporting.writer`[<sup>11</sup>](#reference "Special case"):
      - `k8s-infra-gcr-auditor@k8s-artifacts-prod.iam.gserviceaccount.com`
  - IAM Policy Binding for Service Account[<sup>10</sup>](#reference "Special case"):
    - `k8s-infra-gcr-auditor@k8s-artifacts-prod.iam.gserviceaccount.com`:
      - role: `roles/iam.serviceAccountUser`
      - members:
        - `group:k8s-infra-artifact-admins@kubernetes.io`
  - IAM Service Account[<sup>11</sup>](#reference "Special case"):
    - `k8s-infra-gcr-auditor`:
      - project: `k8s-artifacts-prod`
      - display_name: `k8s-infra container image auditor`
    - `k8s-infra-gcr-auditor-invoker`:
      - project: `k8s-artifacts-prod`
      - display_name: `k8s-infra container image auditor invoker`
  - Cloud Run Service IAM Policy Binding[<sup>11</sup>](#reference "Special case"):
    - `cip-auditor`:
      - members:
        - `serviceAccount:k8s-infra-gcr-auditor-invoker@k8s-artifacts-prod.iam.gserviceaccount.com`
      - role: `roles/run.invoker`
      - platform: `managed`
      - preojct: `k8s-artifacts-prod`
      - region: `us-central1`

  - **Components per [FILE] in `[MODULE_PATH]/static/prod-storage` directory**[<sup>3</sup>](#reference "Special case")<sup>,</sup>[<sup>4</sup>](#reference "Example of how to achieve it with Terraform"):
    - GCS Bucket Object:
      - name: `[FILE]`
      - source: `[MODULE_PATH]/static/prod-storage/[FILE]`
      - bucket: `gs://k8s-artifacts-prod`

- **Components for project: `k8s-cip-test-prod`**[<sup>5</sup>](#reference "Special case"):
  - IAM Policy Binding:
    - `roles/viewer`:
      - `group:k8s-infra-staging-cip-test@kubernetes.io`
  
  - **Components for project: `k8s-cip-test-prod` per [[REGION]](#prod-storage-regions)**:
    - GCS Bucket:
      - `gs://[REGION].artifacts.k8s-cip-test-prod.appspot.com`
    - IAM:
      - `gs://[REGION].artifacts.k8s-cip-test-prod.appspot.com`:
        - `group:k8s-infra-staging-cip-test@kubernetes.io:objectAdmin`
        - `group:k8s-infra-staging-cip-test@kubernetes.io:legacyBucketOwner`

- **Components for project: `k8s-staging-cip-test`**[<sup>5</sup>](#reference "Special case"):
  - IAM Policy Binding:
    - `roles/viewer`:
      - `group:k8s-infra-staging-cip-test@kubernetes.io`
  - IAM[<sup>6</sup>](#reference "Special case"):
    - `gs://artifacts.k8s-staging-cip-test.appspot.com`
      - `serviceAccount:k8s-infra-gcr-promoter@k8s-cip-test-prod.iam.gserviceaccount.com:objectAdmin`
      - `serviceAccount:k8s-infra-gcr-promoter@k8s-cip-test-prod.iam.gserviceaccount.com:legacyBucketOwner`

  - **Components for project: `k8s-staging-cip-test` per [[REGION]](#prod-storage-regions)**:
    - GCS Bucket:
      - `gs://[REGION].artifacts.k8s-staging-cip-test.appspot.com`
    - IAM:
      - `gs://[REGION].artifacts.k8s-staging-cip-test.appspot.com`:
        - `group:k8s-infra-staging-cip-test@kubernetes.io:objectAdmin`
        - `group:k8s-infra-staging-cip-test@kubernetes.io:legacyBucketOwner`

- **Components for project: `k8s-gcr-backup-test-prod`**[<sup>5</sup>](#reference "Special case"):
  - IAM Policy Binding:
    - `roles/viewer`:
      - `group:k8s-infra-staging-cip-test@kubernetes.io`
  
  - **Components for project: `k8s-gcr-backup-test-prod` per [[REGION]](#prod-storage-regions)**:
    - GCS Bucket:
      - `gs://[REGION].artifacts.k8s-gcr-backup-test-prod.appspot.com`
    - IAM:
      - `gs://[REGION].artifacts.k8s-gcr-backup-test-prod.appspot.com`:
        - `group:k8s-infra-staging-cip-test@kubernetes.io:objectAdmin`
        - `group:k8s-infra-staging-cip-test@kubernetes.io:legacyBucketOwner`

- **Components for project: `k8s-gcr-backup-test-prod-bak`**[<sup>5</sup>](#reference "Special case"):
  - IAM Policy Binding:
    - `roles/viewer`:
      - `group:k8s-infra-staging-cip-test@kubernetes.io`
  
  - **Components for project: `k8s-gcr-backup-test-prod-bak` per [[REGION]](#prod-storage-regions)**:
    - GCS Bucket:
      - `gs://[REGION].artifacts.k8s-gcr-backup-test-prod-bak.appspot.com`
    - IAM:
      - `gs://[REGION].artifacts.k8s-gcr-backup-test-prod-bak.appspot.com`:
        - `group:k8s-infra-staging-cip-test@kubernetes.io:objectAdmin`
        - `group:k8s-infra-staging-cip-test@kubernetes.io:legacyBucketOwner`

- **Components for project: `k8s-gcr-audit-test-prod`**[<sup>5</sup>](#reference "Special case"):
  - IAM Policy Binding:
    - `roles/viewer`:
      - `group:k8s-infra-staging-cip-test@kubernetes.io`
    - `roles/errorreporting.admin`[<sup>7</sup>](#reference "Special case")
      - `serviceAccount:k8s-infra-gcr-promoter@k8s-gcr-audit-test-prod.iam.gserviceaccount.com`
    - `roles/logging.admin`[<sup>7</sup>](#reference "Special case")
      - `serviceAccount:k8s-infra-gcr-promoter@k8s-gcr-audit-test-prod.iam.gserviceaccount.com`
    - `roles/pubsub.admin`[<sup>7</sup>](#reference "Special case")
      - `serviceAccount:k8s-infra-gcr-promoter@k8s-gcr-audit-test-prod.iam.gserviceaccount.com`
    - `roles/resourcemanager.projectIamAdmin`[<sup>7</sup>](#reference "Special case")
      - `serviceAccount:k8s-infra-gcr-promoter@k8s-gcr-audit-test-prod.iam.gserviceaccount.com`
    - `roles/run.admin`[<sup>7</sup>](#reference "Special case")
      - `serviceAccount:k8s-infra-gcr-promoter@k8s-gcr-audit-test-prod.iam.gserviceaccount.com`
    - `roles/serverless.serviceAgent`[<sup>7</sup>](#reference "Special case")
      - `serviceAccount:k8s-infra-gcr-promoter@k8s-gcr-audit-test-prod.iam.gserviceaccount.com`
    - `roles/storage.admin`[<sup>7</sup>](#reference "Special case")
      - `serviceAccount:k8s-infra-gcr-promoter@k8s-gcr-audit-test-prod.iam.gserviceaccount.com`
  
  - **Components for project: `k8s-gcr-audit-test-prod` per [[REGION]](#prod-storage-regions)**:
    - GCS Bucket:
      - `gs://[REGION].artifacts.k8s-gcr-audit-test-prod.appspot.com`
    - IAM:
      - `gs://[REGION].artifacts.k8s-gcr-audit-test-prod.appspot.com`:
        - `group:k8s-infra-staging-cip-test@kubernetes.io:objectAdmin`
        - `group:k8s-infra-staging-cip-test@kubernetes.io:legacyBucketOwner`

- **Components for project: `k8s-release-test-prod`**[<sup>8</sup>](#reference "Special case"):
  - IAM Policy Binding:
    - `roles/viewer`:
      - `group:k8s-infra-staging-kubernetes@kubernetes.io`
      - `group:k8s-infra-staging-release-test@kubernetes.io`
  - IAM:
    - `gs://k8s-release-test-prod`[<sup>9</sup>](#reference "Special case"):
      - `serviceAccount:615281671549@cloudbuild.gserviceaccount.com:objectAdmin`
      - `serviceAccount:615281671549@cloudbuild.gserviceaccount.com:legacyBucketReader`
  
  - **Components for project: `k8s-release-test-prod` per [[REGION]](#prod-storage-regions)**:
    - GCS Bucket:
      - `gs://[REGION].artifacts.k8s-release-test-prod.appspot.com`
    - IAM:
      - `gs://[REGION].artifacts.k8s-release-test-prod.appspot.com`:
        - `group:k8s-infra-staging-kubernetes@kubernetes.io:objectAdmin`
        - `group:k8s-infra-staging-kubernetes@kubernetes.io:legacyBucketOwner`
        - `group:k8s-infra-staging-release-test@kubernetes.io:objectAdmin`
        - `group:k8s-infra-staging-release-test@kubernetes.io:legacyBucketOwner`

---

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

---

#### Prod Storage GCLB

##### Prod Storage GCLB / Components

- **Additional components for project: `k8s-artifacts-prod`**
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
  - Compute SSL Certificate:
    - `k8s-artifacts-prod`:
      - domains:
        - `artifacts.k8s.io`
  - Compute Global Forwarding Rule:
    - `k8s-artifacts-prod-http`:
      - target_http_proxy: `k8s-artifacts-prod`
      - ports:
        - `80`
      - ip_address: [GLOBAL_ADDRESS][<sup>1</sup>](#prod-storage-gclb--reference "Example how to get [GLOBAL_ADDRESS] using terraform")
    - `k8s-artifacts-prod-https`:
      - target_https_proxy: `k8s-artifacts-prod`
      - ports:
        - `443`
      - ip_address: [GLOBAL_ADDRESS][<sup>1</sup>](#prod-storage-gclb--reference "Example how to get [GLOBAL_ADDRESS] using terraform")

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

##### What components are needed for each [PROJECT]

- Project `k8s-staging-[PROJECT]` ([ensure-staging-storage.***ensure_project***](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/ensure-staging-storage.sh#L110) / [lib.ensure_project.***gcloud***](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/lib.sh#L81-L96)):
  - organization: `758905017065` ([ensure-staging-storage.***ensure_project***](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/ensure-staging-storage.sh#L110) / [lib.ensure_project.***gcloud***](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/lib.sh#L81-L84))
  - billing_account: `018801-93540E-22A20E` ([ensure-staging-storage.***ensure_project***](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/ensure-staging-storage.sh#L110) / [lib.ensure_project.***gcloud***](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/lib.sh#L98-L99))
- IAM Policy Binding:
  - `roles/viewer`:
    - `group:k8s-infra-staging-[PROJECT]@kubernetes.io` ([ensure-staging-storage.***empower_group_as_viewer***](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/ensure-staging-storage.sh#L114) / [lib.empower_group_as_viewer.***gcloud***](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/lib.sh#L127-L130))
    - `group:k8s-infra-artifact-admins@kubernetes.io` ([ensure-staging-storage.***empower_gcr_admins***](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/ensure-staging-storage.sh#L128) / [lib.***empower_group_as_viewer***](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/lib.sh#L266) / [lib.empower_group_as_viewer.***gcloud***](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/lib.sh#L127-L130))
  - `roles/cloudbuild.builds.editor`:
    - `group:k8s-infra-staging-[PROJECT]@kubernetes.io`
  - `roles/serviceusage.serviceUsageConsumer`:
    - `group:k8s-infra-staging-[PROJECT]@kubernetes.io`
  - `roles/cloudbuild.builds.builder`:
    - `serviceAccount:deployer@k8s-prow.iam.gserviceaccount.com`
- GCR:
  - `k8s-staging-[PROJECT]` ([ensure-staging-storage.***ensure_gcr_repo***](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/ensure-staging-storage.sh#L124) / [lib_gcr.ensure_gcr_repo.**gcloud**](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/lib_gcr.sh#L69-L79))
- IAM:
  - `gs://artifacts.k8s-staging-[PROJECT].appspot.com`:
    - `allUsers:objectViewer` ([ensure-staging-storage.***ensure_gcr_repo***](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/ensure-staging-storage.sh#L124) / [lib_gcr.ensure_gcr_repo.***gsutil***](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/lib_gcr.sh#L81))
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
  - `containerregistry` ([ensure-staging-storage.***enable_api***](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/ensure-staging-storage.sh#L120) / [lib.enable_api.***gcloud***](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/lib.sh#L113))
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
  - `gs://artifacts.k8s-staging-[PROJECT].appspot.com` ([ensure-staging-storage.***ensure_gcr_repo***](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/ensure-staging-storage.sh#L124) / [lib_gcr.ensure_gcr_repo.***gcloud***](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/lib_gcr.sh#L69-L79)):
    - bucketpolicyonly: `true` ([ensure-staging-storage.***ensure_gcr_repo***](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/ensure-staging-storage.sh#L124) / [lib_gcr.ensure_gcr_repo.***gsutil***](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/lib_gcr.sh#L82))

##### What components are needed for each of RELEASE_STAGING_PROJECTS [RS_PROJECT]

> [@bartsmykla: I think it's not consistent to put resources for release projects here, even if they are release ones. I think there should be created separate place for it]

- IAM Policy Binding:
  - `roles/viewer`:
    - `group:k8s-infra-release-viewers@kubernetes.io`

##### What components are needed for project: k8s-staging-release-test

- IAM Policy Binding:
  - `roles/cloudkms.admin`:
    - `group:k8s-infra-release-admins@kubernetes.io`
  - `roles/cloudkms.cryptoKeyEncrypterDecrypter`:
    - `group:k8s-infra-release-admins@kubernetes.io`

---

#### CIP Auditor

Even if there are two scripts for CIP (Container Image Promoter) Auditor ([`ensure-env-cip-auditor.sh`](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/ensure-env-cip-auditor.sh) and [`cip-auditor/deploy.sh`](ttps://github.com/kubernetes/k8s.io/blob/master/infra/gcp/cip-auditor/deploy.sh)) I decided to combine them into one section because I think we shouldn't split them.

> [**todo(@listx)**]: I don't undestand why we first create dummy Cloud Run Service in [`ensure-env-cip-auditor.sh`](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/ensure-env-cip-auditor.sh#L160-L164) and then overwriting it in [`cip-auditor/deploy.sh`](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/cip-auditor/deploy.sh#L47-L54). What stands behind the decision to do it that way? Can we eliminate the step for the dummy service?

##### Env CIP Auditor

###### Env CIP Auditor / Components

- Components for project `k8s-artifacts-prod`:
  
  > Atribute: `[PROJECT_NUMBER]` will be taken from the project component attributes reference
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

##### CIP Auditor Deploy

> [**todo(@listx)**]: Do we need the `[PROJECT]` to be a variable? Is CIP Auditor currently deployed for other then `k8s-artifacts-prod` projects too?

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
        service_account_name = "serviceAccount:k8s-infra-gcr-auditor-invoker@k8s-artifacts-prod.iam.gserviceaccount.com"
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
        service_account_email = "serviceAccount:k8s-infra-gcr-auditor-invoker@k8s-artifacts-prod.iam.gserviceaccount.com"
      }
    }
  }
  ```

- <sup>3</sup> Maybe we can take the `[CIP_AUDITOR_DIGEST]` from the data source: `[google_container_registry_image](https://www.terraform.io/docs/providers/google/d/google_container_registry_image.html)`

---

#### Release Projects

##### Release Projects `[PROJECTS]`

- `k8s-staging-release-test`
- `k8s-release-test-prod`

##### Release Projects / Components

- **Components per [[PROJECT]](#release-projects-projects)**:
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

##### Reference for Release Projects

- <sup>1</sup> [There is a comment from @justaugustus](https://github.com/kubernetes/k8s.io/blob/9e17cdf48d4e9f343e0a11ecb06247897a81dd84/infra/gcp/lib.sh#L316-L320) it's temporary and we should refactor this once we develop custom roles

---

## Proposed structure of new tooling

## Plan of work

## FAQ

### Why terraform?

### How can I help?

## todos

- @listx:
  - > [[**todo(@listx)**]: I don't undestand why we first create dummy Cloud Run Service in `ensure-env-cip-auditor.sh` and then overwriting it in `cip-auditor/deploy.sh`. What stands behind the decision to do it that way? Can we eliminate the step for the dummy service?](#cip-auditor)
  - > [[**todo(@listx)**]: Do we really need the `[PROJECT]` to be a variable? Is CIP Auditor currently deployed for other then `k8s-artifacts-prod` projects too?](#cip-auditor-deploy)
