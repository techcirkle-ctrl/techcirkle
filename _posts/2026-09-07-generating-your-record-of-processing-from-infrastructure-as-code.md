---
layout: post
title: "Generating Your Record of Processing From Infrastructure as Code"
date: 2026-09-07 00:00:00 +0000
categories: ["DevOps", "Privacy"]
tags: ["devops", "infrastructure", "privacy", "automation"]
description: "Stop maintaining a RoPA spreadsheet by interview. Compile it from service catalogues, IaC, and schema registries so it changes when the code does."
image: "https://cdn.sanity.io/images/563mnkns/production/5d43524d112da719e120b419aaa6193ae5267c76-1600x1066.jpg"
author: "TechCirkle Editorial Team"
---

![Engineer reviewing encrypted data protection controls on a laptop](https://cdn.sanity.io/images/563mnkns/production/5d43524d112da719e120b419aaa6193ae5267c76-1600x1066.jpg)

Article 30 of GDPR requires a record of processing activities. Almost every organisation maintains it as a spreadsheet, refreshed during audit season by someone interviewing engineers about what their services do.

The record is therefore accurate for about two weeks a year, and inaccurate in a specific direction: it under-reports. New pipelines get built; nobody tells the privacy team, because no mechanism exists by which anybody would.

The fix is not better process discipline. It is to stop treating the register as a document and start treating it as a build artefact.

## The inputs you already have

Every processing activity is already described somewhere machine-readable. The register is a compilation of those descriptions.

**Service catalogue.** Ownership, purpose, and data classification per service. If you run Backstage or an equivalent, the metadata largely exists; it needs privacy-relevant fields added to the entity schema.

**Infrastructure as code.** Terraform state tells you which data stores exist, where they are hosted, and which region they sit in. Region is a transfer question, and transfer questions are register questions.

**Schema registries.** Field-level classification of what each store actually holds. This is where personal data becomes concrete rather than asserted.

**Pipeline configuration.** dbt manifests, Airflow DAGs, and stream topologies describe how data moves between stores — which is precisely what a register's data-flow section is meant to capture.

**Vendor management.** Contracted processors and sub-processors, with their data processing agreements and transfer mechanisms.

None of this is exotic. Most organisations have four of the five and have never connected them to a compliance obligation.

## The annotation layer

The one thing code does not carry natively is legal meaning. A Terraform resource knows it is an S3 bucket. It does not know the lawful basis for what is inside it.

Add annotations at the source, where the engineer who understands the system is already working:

```yaml
# service.yaml
privacy:
  processes_personal_data: true
  purposes:
    - id: order_fulfilment
      lawful_basis: contract
      categories: [contact_details, address, order_history]
      retention: P7Y
    - id: product_analytics
      lawful_basis: consent
      categories: [behavioural]
      retention: P26M
  processors:
    - vendor: analytics-provider
      transfer_mechanism: scc
```

Two properties make this work. It lives beside the code, so it changes in the same pull request as the behaviour it describes. And it is reviewable by the person who actually knows the answer, rather than reconstructed months later by someone who does not.

## The compiler

A scheduled job — or better, a CI step — walks the sources, merges them, and emits the register.

Practical design notes:

- **Emit both formats.** A machine-readable JSON artefact for downstream tooling, and a human-readable document for the regulator and the auditor. Same source, two renderers.
- **Version the output in git.** The diff between quarterly registers is itself useful evidence, and it makes drift visible as a review artefact rather than a discovery.
- **Fail the build on unannotated stores.** A new data store with no privacy block should not merge. This is the single highest-value rule in the whole system.
- **Reconcile against discovery.** Your scanner finds what is actually there. The register describes what should be there. The delta is your finding list, and it should be reviewed on a schedule.

That last point matters most. Annotations describe intent; scanners describe reality. A register generated only from annotations is a well-structured claim. Reconciled against a scanner, it becomes evidence.

## The AI-era additions

Two categories routinely missing from registers, and both belong in the same pipeline.

**Model artefacts.** Fine-tuning datasets, embedding stores, and feature stores are processing activities. Annotate the training pipeline and the vector database exactly as you would a Postgres instance.

**Prompt and inference logs.** Teams log full request payloads for debugging. Those payloads contain personal data. The retention policy is usually "they're just logs". Annotate the log sink, give it a retention period, and enforce that period with a job.

The AI Act overlap makes this doubly worth doing. Both regimes want documented data provenance and records of automated decision-making. One annotation layer feeding one compiler can serve both, which is considerably better than maintaining two governance stacks that disagree with each other.

## What you get

The register stops being a periodic project and becomes a property of the system.

Audit preparation drops from weeks to days. More usefully, the register becomes trustworthy between audits, which means people start using it for actual decisions — what can this dataset be used for, do we need an assessment for this feature, is this vendor already covered.

The full engineering context — discovery, consent enforcement, rights-request pipelines, deletion patterns, and build-versus-buy per layer — is in the complete guide: **[GDPR Software in 2026: A CTO's Build vs Buy Playbook](https://techcirkle.com/blog/gdpr-software)**.

![Data protection and privacy controls illustrated across connected systems](https://cdn.sanity.io/images/563mnkns/production/d6f70712aff60f2206f137357aac48ea16f51296-1600x900.jpg)

We build this alongside [cloud application architecture](https://techcirkle.com/blog/cloud-application-development-guide) work and as part of broader [custom software](https://techcirkle.com/development/custom-software-development) engagements.

## Frequently Asked Questions

### What is a record of processing activities?
An Article 30 requirement documenting each processing activity: its purpose, lawful basis, data categories, retention period, recipients, and transfer mechanisms. It is the artefact regulators most often request first, and the one most commonly out of date.

### How do you generate a RoPA automatically?
Compile it from sources that already describe your systems — service catalogues, Terraform state, schema registries, pipeline configuration, and vendor management — with a privacy annotation block living beside the code. A CI step merges these and emits both a machine-readable artefact and a human-readable document.

### Where do lawful basis and retention come from in a generated register?
From annotations added at the source, in the same repository as the service they describe. Code carries structure but not legal meaning, so the annotation supplies purpose, lawful basis, data categories, and retention — reviewed in the same pull request as the change it accompanies.

### How do you stop the generated register from drifting?
Fail the build when a new data store has no privacy annotation, and reconcile the register against automated discovery on a schedule. Annotations describe intent, scanners describe reality, and the delta between them is your finding list.

### Should AI systems appear in the record of processing?
Yes. Fine-tuning datasets, embedding stores, feature stores, and prompt or inference logs are all processing activities. Prompt logs are the most commonly omitted, because teams categorise them as debugging output rather than as a personal data store.

### Does a generated register also help with the EU AI Act?
Substantially. Both regimes require documented data provenance and records of automated decision-making. One annotation layer feeding one compiler can serve both obligations, which is considerably cheaper than maintaining parallel governance stacks that disagree.
