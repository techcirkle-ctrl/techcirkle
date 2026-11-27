---
layout: post
title: "CRM as Code: Version-Controlling Your Pipeline, Fields and Data Dictionary"
date: 2026-11-27 00:00:00 +0000
categories: ["Engineering", "Operations"]
tags: ["crm", "startups", "yaml", "devops", "ai"]
description: "Treat CRM pipeline stages, field definitions and the data dictionary as version-controlled config, with a sample YAML schema and review workflow."
image: "https://cdn.sanity.io/images/563mnkns/production/6e1d1eff4232bc3227e85b31ed7e319fadf0fb38-1600x1067.jpg"
author: "TechCirkle"
---

![Startup team reviewing pipeline documents, the kind of sales configuration that can live in version control](https://cdn.sanity.io/images/563mnkns/production/6e1d1eff4232bc3227e85b31ed7e319fadf0fb38-1600x1067.jpg)

Engineers would never accept a production database whose schema anyone could change through a web form, with no review and no history. Yet that is exactly how most startups run their CRM.

Stages get renamed on a Friday afternoon. Someone adds a free-text "Industry" field next to the existing "Industry" picklist. An AI integration starts writing to a field whose meaning nobody wrote down.

The fix is a familiar idea applied to an unfamiliar place: treat your CRM configuration as code.

## What "CRM as code" means

The CRM vendor still stores the records. What moves into a repository is the *definition* of the system:

- Pipeline stages, their order and their written exit criteria.
- Field definitions: name, type, allowed values, whether required, who owns them.
- The data dictionary, in plain language, for humans and AI tools alike.
- Rules about what automation and AI agents may change without human confirmation.

That repository becomes the source of truth. Changes go through pull requests. A script, or a careful human with a checklist, applies approved changes to the live CRM.

## Why it matters early

The data model decisions made in the first month cause most painful CRM migrations later. Separating companies, people and deals, using picklists for anything you report on, and recording stage-change dates are all cheap on day one and expensive in year two.

Version control gives you three things a vendor admin panel does not:

1. **History.** `git log` tells you when "Proposal" started meaning something different.
2. **Review.** A second person sees a new required field before every seller hits it.
3. **A shared contract.** AI agents, integrations and new hires all read the same file.

For the broader question of *when* a startup should set up a CRM at all, and what to include at each stage, see our guide on when a startup should implement CRM: https://techcirkle.com/blog/when-should-a-startup-implement-crm

## A sample data dictionary

Here is a minimal starting point for an early-stage B2B pipeline. It is deliberately small: one pipeline, four required deal fields and a fixed list of loss reasons.

```yaml
# crm/data-dictionary.yml
version: 3
owner: revops@yourcompany.example

objects:
  company:
    external_id: billing_account_id   # matches billing + product systems
  person:
    belongs_to: company
  deal:
    belongs_to: company

pipelines:
  new_business:
    stages:
      - key: identified
        label: Identified
        exit_criteria: "Named contact at a company that fits our target segment."
      - key: contacted
        label: Contacted
        exit_criteria: "Two-way conversation has happened (reply or call)."
      - key: discovery
        label: Discovery
        exit_criteria: "Problem, decision-maker and timeline are written in notes."
      - key: proposal
        label: Proposal
        exit_criteria: "Written proposal sent to the buyer."
      - key: won
        label: Won
        exit_criteria: "Contract signed or first invoice paid."
      - key: lost
        label: Lost
        exit_criteria: "Buyer declined or went silent past the agreed date."
    track_stage_change_dates: true

fields:
  deal.value:
    type: currency
    required: true
    ai_write: never          # human-owned: money
  deal.expected_close_date:
    type: date
    required: true
    ai_write: propose        # AI suggests, human confirms
  deal.source:
    type: picklist
    required: true
    values: [inbound_form, outbound, referral, event, partner]
    ai_write: propose
  deal.next_step:
    type: text
    required: true
    requires_date: true
    ai_write: propose
  deal.loss_reason:
    type: picklist
    values: [price, timing, no_decision, competitor, missing_feature]
    ai_write: propose
  deal.call_summary:
    type: long_text
    ai_write: auto           # descriptive, low risk

governance:
  field_creation: pull_request_required
  reviewers: [founder, revops]
```

A few design notes:

- `ai_write` encodes a simple policy: let AI write freely to descriptive fields, have it *propose* changes to stages and dates, and keep money and forecast fields human-owned.
- `external_id` ties CRM companies to billing and product records, so integrations never depend on names or email addresses.
- Every stage has an `exit_criteria` sentence. That sentence is what your team, your dashboards and any language model all use to interpret the stage.

## A lightweight workflow

You do not need heavy tooling to start:

1. Keep `crm/data-dictionary.yml` in the same repo as your internal tools, or a small dedicated one.
2. Add a CI check that validates the YAML: unique keys, picklists not empty, every required field has an `ai_write` value.
3. Require a pull request for any new field or stage.
4. Apply approved changes to the CRM by script where the vendor API allows it, or by checklist where it does not.
5. Feed the same file to any AI agent that reads or writes CRM data, so its prompts reflect your actual definitions.

That last step is where this pays off. When an agent qualifies a lead, enriches it and routes it to an owner, it should log what it changed and why, and stay within the permissions in the file. If you are building that kind of multi-step automation, this is how we approach it: https://techcirkle.com/agentic-workflow-development

## When the config outgrows the vendor

Sometimes the data dictionary makes a mismatch obvious: your sales motion is tied to product usage, marketplace events or regulated steps that an off-the-shelf tool models badly. At that point the YAML is already a solid spec for custom modules or a purpose-built system around a standard CRM core.

Boring, reviewed, versioned definitions are what let you add AI, reporting and integrations later without starting over.

## Frequently Asked Questions

### Does "CRM as code" mean building our own CRM?

No. The vendor still stores and serves records. Only the definitions, such as stages, fields and rules, live in version control and are applied to the vendor's system.

### What should go into a CRM data dictionary?

Each object and how it relates to others, every pipeline stage with a written exit criterion, every field with its type and allowed values, ownership, and rules about which fields automation may change.

### How do we apply YAML changes to a hosted CRM?

Where the vendor exposes an admin or metadata API, a small script can sync approved definitions. Where it does not, use the merged pull request as a checklist and apply changes by hand.

### Why record AI write permissions per field?

Because AI can quietly corrupt important data. Marking fields as auto, propose or never makes the policy explicit, reviewable and enforceable by any agent or integration that touches the CRM.

### Is this overkill for a three-person startup?

A single YAML file and a pull request rule take an hour to set up. That is cheap insurance against field sprawl, which is one of the main reasons CRMs become unusable as teams grow.

### What should the CI check validate?

Unique keys, non-empty picklists, valid field types, a defined AI write policy for each field and an exit criterion for every stage. Keep it simple enough that it never blocks a legitimate change.
