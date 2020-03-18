# CIP Auditor <!-- omit in toc -->

Analyzed scripts: [ensure-env-cip-auditor.sh](https://github.com/kubernetes/k8s.io/blob/e62c18e79a75615d4868afaf5eebcf36bb265df9/infra/gcp/ensure-env-cip-auditor.sh) and [cip-auditor/deploy.sh](https://github.com/kubernetes/k8s.io/blob/e62c18e79a75615d4868afaf5eebcf36bb265df9/infra/gcp/cip-auditor/deploy.sh)

---

## Table of Content <!-- omit in toc -->

- [Preface](#preface)
- [CIP Auditor Env](#cip-auditor-env)
  - [Terraform resources for CIP Auditor Env](#terraform-resources-for-cip-auditor-env)
  - [CIP Auditor Env / Components](#cip-auditor-env--components)
  - [Yaml representation of *CIP Auditor Env Components*](#yaml-representation-of-cip-auditor-env-componentsg1g2)
- [CIP Auditor Deploy](#cip-auditor-deploy)
  - [Terraform resources for CIP Auditor Deploy](#terraform-resources-for-cip-auditor-deploy)
  - [CIP Auditor Deploy / Components](#cip-auditor-deploy--components)
  - [Yaml representation of *CIP Auditor Deploy Components*](#yaml-representation-of-cip-auditor-deploy-componentsg1)
- [Reference](#reference)

## Preface

Even if there are two scripts for CIP (Container Image Promoter) Auditor ([`ensure-env-cip-auditor.sh`](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/ensure-env-cip-auditor.sh) and [`cip-auditor/deploy.sh`](ttps://github.com/kubernetes/k8s.io/blob/master/infra/gcp/cip-auditor/deploy.sh)) I decided to combine them into one section because I think we shouldn't split them.

> [**todo(@listx)**]: I don't undestand why we first create dummy Cloud Run Service in [`ensure-env-cip-auditor.sh`](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/ensure-env-cip-auditor.sh#L160-L164) and then overwriting it in [`cip-auditor/deploy.sh`](https://github.com/kubernetes/k8s.io/blob/master/infra/gcp/cip-auditor/deploy.sh#L47-L54). What stands behind the decision to do it that way? Can we eliminate the step for the dummy service?

## CIP Auditor Env

### Terraform resources for CIP Auditor Env

- Provider: [`Google`](https://www.terraform.io/docs/providers/google/index.html "Provider: Google")
  - [`google_project_service`](https://www.terraform.io/docs/providers/google/r/google_project_service.html "Resource: Google Project Service")
  - [`google_cloud_run_service`](https://www.terraform.io/docs/providers/google/r/cloud_run_service.html "Resource: Google Cloud Run Service")
  - [`google_pubsub_topic`](https://www.terraform.io/docs/providers/google/r/pubsub_topic.html "Resource: Google PubSub Topic")
  - [`google_pubsub_subscription`](https://www.terraform.io/docs/providers/google/r/pubsub_subscription.html "Resource: Google PubSub Subscription")
  - [`google_project_iam_binding`](https://www.terraform.io/docs/providers/google/r/google_project_iam.html "Resource: Google Project IAM Binding")
  - [`google_project_iam_member`](https://www.terraform.io/docs/providers/google/r/google_project_iam.html "Resource: Google Project IAM Member")[<sup>G2</sup>](../README.md#global-reference)

### CIP Auditor Env / Components

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
  - Cloud Run Service[<sup>1</sup>](#reference "Example of terraform component for cloud run service"):
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
      - push_auth_service_account: `serviceAccount:k8s-infra-gcr-auditor-invoker@k8s-artifacts-prod.iam.gserviceaccount.com`[<sup>2</sup>](#reference)

### Yaml representation of *CIP Auditor Env Components*[<sup>G1</sup>](../README.md#global-reference),</sup>[<sup>G2</sup>](../README.md#global-reference)

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

## CIP Auditor Deploy

> [**todo(@listx)**]: Do we need the `[PROJECT]` to be a variable? Is CIP Auditor currently deployed for other than `k8s-artifacts-prod` projects too?

### Terraform resources for CIP Auditor Deploy

- Provider: [`Google`](https://www.terraform.io/docs/providers/google/index.html "Provider: Google")
  - [`google_cloud_run_service`](https://www.terraform.io/docs/providers/google/r/cloud_run_service.html "Resource: Google Cloud Run Service")

### CIP Auditor Deploy / Components

- Components for `[PROJECT]` ([currently provided via CLI argument](https://github.com/kubernetes/k8s.io/blob/3ec33f49bf4181f355e30d3871174a79cda3aad2/infra/gcp/cip-auditor/deploy.sh#L75))
  - Cloud Run Service[<sup>1</sup>](#reference "Example of terraform component for cloud run service"):

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

### Yaml representation of *CIP Auditor Deploy Components*[<sup>G1</sup>](../README.md#global-reference)

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

## Reference

- <sup>1</sup> Example of fow to create Google Cloud Run Service component in terraform[<sup>3</sup>](#reference)

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
