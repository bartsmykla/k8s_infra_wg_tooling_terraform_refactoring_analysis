# Releng <!-- omit in toc -->

Analyzed script: [ensure-releng.sh](https://github.com/kubernetes/k8s.io/blob/e62c18e79a75615d4868afaf5eebcf36bb265df9/infra/gcp/ensure-releng.sh)

---

## Table of Content <!-- omit in toc -->

- [Terraform resources](#terraform-resources)
- [Components](#components)
- [Yaml representation of Components](#yaml-representation-of-componentsg1g2)

---

## Terraform resources

- Provider: [`Google`](https://www.terraform.io/docs/providers/google/index.html "Provider: Google")
  - [`google_project`](https://www.terraform.io/docs/providers/google/r/google_project.html "Resource: Google Project")
  - [`google_project_service`](https://www.terraform.io/docs/providers/google/r/google_project_service.html "Resource: Google Project Service")
  - [`google_project_iam_binding`](https://www.terraform.io/docs/providers/google/r/google_project_iam.html "Resource: Google Project IAM Binding")
  - [`google_project_iam_member`](https://www.terraform.io/docs/providers/google/r/google_project_iam.html "Resource: Google Project IAM Member")[<sup>G2</sup>](../README.md##global-reference)

## Components

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

## Yaml representation of Components[<sup>G1</sup>](../README.md#global-reference)<sup>,</sup>[<sup>G2</sup>](../README.md#global-reference)

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
    project: k8s-releng-prod
```
