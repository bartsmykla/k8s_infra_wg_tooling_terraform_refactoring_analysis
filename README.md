# Infrastructure tools refactoring <!-- omit in toc -->

- [Preface](#preface)
  - [tl;dr](#tldr)
  - [How to read this document](#how-to-read-this-document)
  - [Why I created this document](#why-i-created-this-document)
  - [What problem are we trying to solve](#what-problem-are-we-trying-to-solve)
  - [Analysis of current state](#analysis-of-current-state)
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

### Analysis of current state

One could ask why this much effort to analyse all this components by hand and not to just use [audit](https://github.com/kubernetes/k8s.io/tree/9e3204a10978dde2f4114940842e0546edfe32ce/audit) and take them drom there. This is a great question and the answer is I wanted to compare if all components from [audit](https://github.com/kubernetes/k8s.io/tree/9e3204a10978dde2f4114940842e0546edfe32ce/audit) are being created by [our tooling](https://github.com/kubernetes/k8s.io/tree/9e3204a10978dde2f4114940842e0546edfe32ce/infra/gcp).

What I don't like is there are places like CIP Auditor which need some scripts being run in a specified sequence for it to be able to correctly provision resources. We need to make it more atomic.

- [CIP Auditor](./cip_auditor)
  - [Preface](./cip_auditor#preface)
  - [CIP Auditor Env](./cip_auditor#cip-auditor-env)
    - [Terraform resources for CIP Auditor Env](./cip_auditor#terraform-resources-for-cip-auditor-env)
    - [CIP Auditor Env / Components](./cip_auditor#cip-auditor-env--components)
    - [Yaml representation of *CIP Auditor Env Components*](./cip_auditor#yaml-representation-of-cip-auditor-env-componentsg1g2)
  - [CIP Auditor Deploy](./cip_auditor#cip-auditor-deploy)
    - [Terraform resources for CIP Auditor Deploy](./cip_auditor#terraform-resources-for-cip-auditor-deploy)
    - [CIP Auditor Deploy / Components](./cip_auditor#cip-auditor-deploy--components)
    - [Yaml representation of *CIP Auditor Deploy Components*](./cip_auditor#yaml-representation-of-cip-auditor-deploy-componentsg1)
  - [Reference](./cip_auditor#reference)
- [Conformance Storage](./conformance_storage)
  - [Terraform resources](./conformance_storage#terraform-resources)
  - [Variables used in Components](./conformance_storage#variables-used-in-components)
    - [`[BUCKETS]`](./conformance_storage#buckets)
  - [Components](./conformance_storage#components)
  - [Yaml representation of Components](./conformance_storage#yaml-representation-of-componentsg1g2)

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
