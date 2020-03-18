# GSuite <!-- omit in toc -->

Analyzed script: [ensure-gsuite.sh](https://github.com/kubernetes/k8s.io/blob/e62c18e79a75615d4868afaf5eebcf36bb265df9/infra/gcp/ensure-releng.sh)

---

## Table of Content <!-- omit in toc -->

- [Terraform resources](#terraform-resources)
- [Components](#components)
- [Yaml representation of Components](#yaml-representation-of-componentsg1g2)
- [Reference](#reference)

---

## Terraform resources

- Provider: [`Google`](https://www.terraform.io/docs/providers/google/index.html "Provider: Google")
  - [`google_project`](https://www.terraform.io/docs/providers/google/r/google_project.html "Resource: Google Project")
  - [`google_project_service`](https://www.terraform.io/docs/providers/google/r/google_project_service.html "Resource: Google Project Service")
  - [`google_service_account`](https://www.terraform.io/docs/providers/google/r/google_service_account.html "Resource: Google Service Account")
  - [`google_project_iam_binding`](https://www.terraform.io/docs/providers/google/r/google_project_iam.html "Resource: Google Project IAM Binding")
  - [`google_project_iam_member`](https://www.terraform.io/docs/providers/google/r/google_project_iam.html "Resource: Google Project IAM Member")[<sup>G2</sup>](../README.md#global-reference)

## Components

- **Components for project: `k8s-gsuite`**[<sup>1</sup>](#referencee):
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

## Yaml representation of Components[<sup>G1</sup>](../README.md#global-reference),</sup>[<sup>G2</sup>](../README.md#global-reference)

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

## Reference

- <sup>1</sup> [At the end of script for provisioning GSuite resources](https://github.com/kubernetes/k8s.io/blob/e62c18e79a75615d4868afaf5eebcf36bb265df9/infra/gcp/ensure-gsuite.sh#L72-L86) there is mention about some manual tasks human needs to do to grant GSuite's service account access (it also needs to be acknowdleged by the human who runs the script by pressing enter)
