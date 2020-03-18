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
    - [Main Project](./main_project/README.md)
      - [Terraform data sources and resources](./main_project/README.md#terraform-data-sources-and-resources)
      - [Components](./main_project/README.md#components)
      - [Yaml representation of Components](./main_project/README.md#yaml-representation-of-componentsg1g2)
    - [Prod Storage](#prod-storage)
      - [Terraform resources for Prod Storage](#terraform-resources-for-prod-storage)
      - [Prod Storage `[PROJECTS]`](#prod-storage-projects)
      - [Prod Storage `[REGIONS]`](#prod-storage-regions)
      - [Prod Storage / Components](#prod-storage--components)
      - [Yaml representation of *Prod Storage*](#yaml-representation-of-prod-storageg1g2)
      - [Notes for Prod Storage](#notes-for-prod-storage)
      - [Reference for Prod Storage](#reference-for-prod-storage)
    - [Prod Storage GCLB](#prod-storage-gclb)
      - [Terraform resources for Prod Storage GCLB](#terraform-resources-for-prod-storage-gclb)
      - [Prod Storage GCLB / Components](#prod-storage-gclb--components)
      - [Yaml representation of *Prod Storage GCLB*](#yaml-representation-of-prod-storage-gclbg1)
      - [Reference for Prod Storage GCLB](#reference-for-prod-storage-gclb)
    - [Staging Storage](#staging-storage)
      - [Terraform resources for Staging Storage](#terraform-resources-for-staging-storage)
      - [Staging Storage `[PROJECTS]`](#staging-storage-projects)
      - [Staging Storage `[RS_PROJECTS]`](#staging-storage-rs_projects)
      - [Staging Storage / Components](#staging-storage--components)
      - [Yaml representation of *Staging Storage*](#yaml-representation-of-staging-storageg1g2)
    - [Conformance Storage](#conformance-storage)
      - [Terraform resources for Conformance Storage](#terraform-resources-for-conformance-storage)
      - [Conformance Storage `[BUCKETS]`](#conformance-storage-buckets)
      - [Conformance Storage / Components](#conformance-storage--components)
      - [Yaml representation of *Conformance Storage*](#yaml-representation-of-conformance-storageg1g2)
    - [CIP Auditor](#cip-auditor)
      - [CIP Auditor Env](#cip-auditor-env)
        - [Terraform resources for CIP Auditor Env](#terraform-resources-for-cip-auditor-env)
        - [CIP Auditor Env / Components](#cip-auditor-env--components)
        - [Yaml representation of *CIP Auditor Env Components*](#yaml-representation-of-cip-auditor-env-componentsg1g2)
      - [CIP Auditor Deploy](#cip-auditor-deploy)
        - [Terraform resources for CIP Auditor Deploy](#terraform-resources-for-cip-auditor-deploy)
        - [CIP Auditor Deploy / Components](#cip-auditor-deploy--components)
        - [Yaml representation of *CIP Auditor Deploy Components*](#yaml-representation-of-cip-auditor-deploy-componentsg1)
      - [Reference for CIP Auditor](#reference-for-cip-auditor)
    - [Release Projects](#release-projects)
      - [Terraform resources for Release Projects](#terraform-resources-for-release-projects)
      - [Release Projects `[PROJECTS]`](#release-projects-projects)
      - [Release Projects / Components](#release-projects--components)
      - [Yaml representation of *Release Projects Components*](#yaml-representation-of-release-projects-componentsg1g2)
      - [Reference for Release Projects](#reference-for-release-projects)
    - [Releng](#releng)
      - [Releng / Components](#releng--components)
      - [Terraform resources for Releng](#terraform-resources-for-releng)
      - [Yaml representation of *Releng Components*](#yaml-representation-of-releng-componentsg1g2)
    - [GSuite](#gsuite)
      - [Terraform resources for GSuite](#terraform-resources-for-gsuite)
      - [GSuite / Components](#gsuite--components)
      - [Yaml representation of *GSuite Components*](#yaml-representation-of-gsuite-componentsg1g2)
      - [Reference for GSuite](#reference-for-gsuite)
    - [Namespaces](#namespaces)
      - [Terraform resources for Namespaces](#terraform-resources-for-namespaces)
      - [Namespaces `[PROJECTS]`](#namespaces-projects)
      - [Namespaces / Components](#namespaces--components)
      - [Yaml representation of *Namespaces Components*](#yaml-representation-of-namespaces-componentsg1)
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
