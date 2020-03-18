# Release Projects <!-- omit in toc -->

Analyzed script: [ensure-release-projects.sh](https://github.com/kubernetes/k8s.io/blob/e62c18e79a75615d4868afaf5eebcf36bb265df9/infra/gcp/ensure-release-projects.sh)

---

## Table of Content <!-- omit in toc -->

- [Terraform resources](#terraform-resources)
- [Variables used in Components](#variables-used-in-components)
  - [`[PROJECTS]`](#projects)
- [Components](#components)
- [Yaml representation of Components](#yaml-representation-of-componentsg1g2)
- [Reference](#reference)

---

## Terraform resources

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

## Variables used in Components

### `[PROJECTS]`

- `k8s-staging-release-test`
- `k8s-release-test-prod`

## Components

- **Components per [`[PROJECT]`](#projects)**:
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
    - `roles/serviceusage.serviceUsageConsumer`[<sup>1</sup>](#reference "Comment about temporary nature of this role here"):
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

## Yaml representation of Components[<sup>G1</sup>](../README.md#global-reference),</sup>[<sup>G2</sup>](../README.md#global-reference)

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

## Reference

- <sup>1</sup> [There is a comment from @justaugustus](https://github.com/kubernetes/k8s.io/blob/e62c18e79a75615d4868afaf5eebcf36bb265df9/infra/gcp/lib.sh#L319-L323) it's temporary and we should refactor this once we develop custom roles
