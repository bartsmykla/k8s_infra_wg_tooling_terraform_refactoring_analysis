# Namespaces <!-- omit in toc -->

Analyzed script: [namespaces/ensure-namespaces.sh](https://github.com/kubernetes/k8s.io/blob/e62c18e79a75615d4868afaf5eebcf36bb265df9/infra/gcp/namespaces/ensure-namespaces.sh)

---

## Table of Content <!-- omit in toc -->

- [Terraform resources](#terraform-resources)
- [Variables used in Components](#variables-used-in-components)
  - [`[PROJECTS]`](#projects)
- [Components](#components)
- [Yaml representation of Components](#yaml-representation-of-componentsg1)

---

## Terraform resources

- Provider: [`Kubernetes`](https://www.terraform.io/docs/providers/kubernetes/index.html "Provider: Kubernetes")
  - [`kubernetes_namespace`](https://www.terraform.io/docs/providers/kubernetes/r/namespace.html "Resource: Kubernetes Namespace")
  - [`kubernetes_role`](https://www.terraform.io/docs/providers/kubernetes/r/role.html "Resource: Kubernetes Role")
  - [`kubernetes_role_binding`](https://www.terraform.io/docs/providers/kubernetes/r/role_binding.html "Resource: Kubernetes Role Binding")

## Variables used in Components

### `[PROJECTS]`

- `gcsweb`
- `publishing-bot`
- `k8s-io-prod`
- `k8s-io-canary`

## Components

- **Components per [[PROJECT]](#projects)**:
  - Kubernetes Namespace:
    - `[PROJECT]`
  - Kubernetes Role:
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
  - Kubernetes Role Binding:
    - `namespace-user`:
      - namespace: `[PROJECT]`
      - subjects:
        - `k8s-infra-rbac-[PROJECT]@kubernetes.io`:
          - kind: `Group`
      - role_ref:
        - name: `namespace-user`
        - kind: `Role`
        - api_group: `rbac.authorization.k8s.io`

## Yaml representation of Components[<sup>G1</sup>](../README.md#global-reference)

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
