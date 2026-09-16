---
layout: post
title: "Policy as Code Around a CNAPP: Encoding Your Own Architecture Rules"
date: 2026-09-16 00:00:00 +0000
categories: ["Architecture", "Cloud Security"]
tags: ["policy-as-code", "opa", "terraform", "devsecops"]
description: "How to enforce your own cloud architecture rules and ownership tags as code alongside a CNAPP, with illustrative Terraform, Rego and CI snippets."
image: "https://cdn.sanity.io/images/563mnkns/production/008529a9adbb3c3c70b73f0a947b815c4766913d-1600x1200.jpg"
author: "TechCirkle Editorial Team"
---

![Padlock over a cloud network representing codified cloud security policy](https://cdn.sanity.io/images/563mnkns/production/008529a9adbb3c3c70b73f0a947b815c4766913d-1600x1200.jpg)

A cloud-native application protection platform ships with hundreds of built-in checks: public buckets, unencrypted volumes, open security groups. Those are useful, and they are generic by design.

What no vendor ships is *your* architecture. The rule that production databases never live in a public subnet. The rule that AI inference services are only reachable through the internal gateway. The rule that every resource has an owning team, so a finding can be routed to someone who can fix it.

Those rules belong in code, versioned alongside the infrastructure they govern and evaluated before a change merges. The CNAPP then becomes the runtime backstop that catches drift and anything created outside the pipeline.

This post walks through a practical layout. **All snippets are illustrative** — simplified to show the pattern, not drop-in production policy.

## The division of labour

Keep responsibilities clear:

- **Pull-request time (policy as code):** your organisation-specific rules, evaluated against the Terraform plan. Fast feedback, owned by platform engineering.
- **Post-deploy (CNAPP):** vendor rules plus your custom rules mirrored into the platform, evaluated continuously against the live estate, correlated with identity, workload and data context.
- **Exceptions:** one auditable place, with expiry dates, consumed by both layers.

If the same rule exists in both layers, the pull-request version prevents the problem and the CNAPP version detects anything that bypassed the pipeline.

## Step 1: Make ownership non-optional

Routing is where most rollouts stall. Findings without an owner land in a shared queue and age there.

Start by enforcing tags at the provider level so they exist on everything Terraform creates:

```hcl
# Illustrative only
provider "aws" {
  region = var.region

  default_tags {
    tags = {
      team        = var.team
      service     = var.service
      environment = var.environment
      repo        = var.repo_url
    }
  }
}

variable "team" {
  type = string
  validation {
    condition     = length(var.team) > 0
    error_message = "Every stack must declare an owning team."
  }
}
```

Default tags help, but they do not stop someone overriding them or using a module that ignores them, so also check the plan.

## Step 2: Evaluate the plan with OPA

Render the plan to JSON and evaluate it with Open Policy Agent. A tag check might look like this:

```rego
# Illustrative only
package terraform.ownership

import rego.v1

required_tags := {"team", "service", "environment"}

deny contains msg if {
  some rc in input.resource_changes
  rc.change.actions[_] in {"create", "update"}
  tags := object.get(rc.change.after, "tags_all", {})
  missing := required_tags - {k | tags[k]}
  count(missing) > 0
  msg := sprintf("%s is missing ownership tags: %v", [rc.address, missing])
}
```

![Cloud infrastructure diagram with policy checks between code and deployment](https://cdn.sanity.io/images/563mnkns/production/1a6d63ba1aa374ded8418595cad9e60729270d6e-1600x640.jpg)

## Step 3: Encode architecture rules

Now the rules only your organisation knows. The simplest version of "no production databases in public subnets" checks the database itself; a fuller version also resolves its subnet group against subnets tagged `tier = public`:

```rego
# Illustrative only
package terraform.architecture

import rego.v1

deny contains msg if {
  some rc in input.resource_changes
  rc.type == "aws_db_instance"
  rc.change.after.tags_all.environment == "prod"
  rc.change.after.publicly_accessible == true
  msg := sprintf("%s: production databases must not be publicly accessible", [rc.address])
}
```

The inference rule follows the same shape: any load balancer or endpoint tagged `workload = ai-inference` must be internal, and its security group may only accept traffic from the gateway's security group. Encoding this matters more each quarter, because model endpoints are often stood up quickly by product teams outside the usual review path. When we deliver [LLM integration](https://techcirkle.com/llm-integration) work, this is one of the first guardrails we add.

## Step 4: Wire it into CI with warn and block tiers

Not every rule should block. Split policies into a small blocking set and a broader warning set, and promote rules only once teams trust them:

```yaml
# Illustrative only
name: policy-check
on: pull_request

jobs:
  opa:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: terraform init -input=false
      - run: terraform plan -out=tfplan -input=false
      - run: terraform show -json tfplan > plan.json
      - name: Blocking policies
        run: conftest test plan.json --policy policy/block --namespace terraform.ownership --namespace terraform.architecture
      - name: Advisory policies
        run: conftest test plan.json --policy policy/warn || true
```

Blocking a hotfix over a low-risk rule is the quickest way to get the whole check bypassed. Keep the blocking set short and boring.

## Step 5: Handle exceptions explicitly

Engineers need a legitimate way to accept a risk. Store exceptions as data in the policy repository:

```json
{
  "address": "aws_db_instance.legacy_reporting",
  "rule": "prod_db_not_public",
  "reason": "Vendor integration, migration tracked in PLAT-412",
  "approved_by": "platform-security",
  "expires": "2026-12-31"
}
```

Policies skip matching addresses until the expiry date, after which the check fails again. Mirror the same exception list into the CNAPP's suppression mechanism through its API so both layers agree.

## Step 6: Close the loop with the CNAPP

Three integrations finish the picture:

1. **Mirror custom rules.** Most platforms support custom policies over their inventory. Recreate your blocking rules there so drift and console-created resources are caught.
2. **Route by tag.** Use the `team` and `repo` tags to send prioritised findings into each team's tracker.
3. **Report on bypasses.** A resource that violates a rule in production but never failed a pull request was created outside the pipeline. That list is often more revealing than the findings themselves.

## What this buys you

Generic checks tell you the cloud is configured badly in ordinary ways. Encoded architecture rules tell you it has drifted from the design you intended — and ownership tags make sure the right person hears about it. That combination is what turns a CNAPP from a dashboard into part of delivery.

For platform comparison, AI prioritisation, pricing and rollout sequencing, see the full guide to [CNAPP tools](https://techcirkle.com/blog/cnapp-tools).

## Frequently Asked Questions

### Why not rely on the CNAPP's custom policies alone?

Because they typically evaluate resources after deployment. Checking the Terraform plan stops the problem before it exists and gives the author feedback in the pull request.

### Is OPA the only option for this?

No. Sentinel, Checkov custom checks and cloud-native policy services can express similar rules. OPA is shown here because it is open and widely supported.

### How many rules should block merges?

Very few at first — ownership tags and a handful of high-impact architecture rules. Everything else starts as a warning until teams trust it.

### What about resources created outside Terraform?

The pipeline cannot see them. That is exactly the gap the CNAPP covers, and reporting on those bypasses is worth doing.

### How do we stop exceptions becoming permanent?

Require an expiry date and a tracked reason on every exception, and let policies fail again automatically once the date passes.

### Do these snippets work as written?

They are illustrative and simplified. Adapt schema paths, provider details and tool versions to your own environment and test before enforcing.
