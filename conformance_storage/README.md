# Conformance Storage <!-- omit in toc -->

Analyzed script: [ensure-conformance-storage.sh](https://github.com/kubernetes/k8s.io/blob/e62c18e79a75615d4868afaf5eebcf36bb265df9/infra/gcp/ensure-conformance-storage.sh)

---

## Table of Content <!-- omit in toc -->

- [Terraform resources](#terraform-resources)
- [Variables used in Components](#variables-used-in-components)
  - [`[BUCKETS]`](#buckets)
- [Components](#components)
- [Yaml representation of Components](#yaml-representation-of-componentsg1g2)

---

## Terraform resources

- Provider: [`Google`](https://www.terraform.io/docs/providers/google/index.html "Provider: Google")
  - [`google_project`](https://www.terraform.io/docs/providers/google/r/google_project.html "Resource: Google Project")
  - [`google_project_service`](https://www.terraform.io/docs/providers/google/r/google_project_service.html "Resource: Google Project Service")
  - [`google_storage_bucket`](https://www.terraform.io/docs/providers/google/r/storage_bucket.html "Resource: Google Storage Bucket")
  - [`google_project_iam_binding`](https://www.terraform.io/docs/providers/google/r/google_project_iam.html "Resource: Google Project IAM Binding")
  - [`google_project_iam_member`](https://www.terraform.io/docs/providers/google/r/google_project_iam.html "Resource: Google Project IAM Member")[<sup>G2</sup>](#global-reference)
  - [`google_storage_bucket_iam_binding`](https://www.terraform.io/docs/providers/google/r/storage_bucket_iam.html "Resource: Google Storage Bucket IAM Binding")
  - [`google_storage_bucket_iam_member`](https://www.terraform.io/docs/providers/google/r/storage_bucket_iam.html "Resource: Google Storage Bucket IAM Member")[<sup>G2</sup>](#global-reference)
  - [`google_storage_bucket_acl`](https://www.terraform.io/docs/providers/google/r/storage_bucket_acl.html "Resource: Google Storage Bucket ACL")

## Variables used in Components

### `[BUCKETS]`

- `capi-openstack`
- `cri-o`
- `huaweicloud`

## Components

- **Components for project: `k8s-conform`**:
  - Project:
    - `k8s-conform`
  - API:
    - `storage-component`
  - **Components for project: `k8s-conform` per [`[BUCKET]`](#buckets)**:
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

## Yaml representation of Components[<sup>G1</sup>](../README.md#global-reference),</sup>[<sup>G2</sup>](../README.md#global-reference)

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
