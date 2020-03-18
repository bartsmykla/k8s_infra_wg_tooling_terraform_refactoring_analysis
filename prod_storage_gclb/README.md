# Prod Storage GCLB <!-- omit in toc -->

Analyzed script: [ensure-prod-storage-gclb.sh](https://github.com/kubernetes/k8s.io/blob/e62c18e79a75615d4868afaf5eebcf36bb265df9/infra/gcp/ensure-prod-storage-gclb.sh)

---

## Table of Content <!-- omit in toc -->

- [Terraform resources](#terraform-resources)
- [Prod Storage GCLB / Components](#prod-storage-gclb--components)
- [Yaml representation of Components](#yaml-representation-of-componentsg1g2)
- [Reference](#reference)

---

## Terraform resources

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

## Components

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
      - ip_address: `[GLOBAL_ADDRESS]`[<sup>1</sup>](#reference "Example how to get [GLOBAL_ADDRESS] using terraform")
    - `k8s-artifacts-prod-https`:
      - target_https_proxy: `k8s-artifacts-prod`
      - ports:
        - `443`
      - ip_address: `[GLOBAL_ADDRESS]`[<sup>1</sup>](#reference "Example how to get [GLOBAL_ADDRESS] using terraform")

## Yaml representation of Components[<sup>G1</sup>](#global-reference)

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

## Reference

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
