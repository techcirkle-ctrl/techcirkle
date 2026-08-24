---
layout: post
title: "A Handover Checklist for Outsourced Software Builds"
date: 2026-08-24 00:00:00 +0000
categories: ["Engineering", "Checklists", "DevOps"]
tags: ["devops", "documentation", "infrastructure", "checklist"]
description: "Lock-in accumulates, it is not designed. A checklist for keeping an outsourced build portable: IaC, docs, accounts, ADRs and a rehearsed handover."
image: "https://cdn.sanity.io/images/563mnkns/production/622d76b634e6e0c9dad1584be07b9a27ed32d4ce-1600x1066.jpg"
author: "TechCirkle Editorial Team"
---

![Team reviewing delivery plans together](https://cdn.sanity.io/images/563mnkns/production/622d76b634e6e0c9dad1584be07b9a27ed32d4ce-1600x1066.jpg)

Vendor lock-in is rarely a trap someone set. It accumulates.

A deployment step that lives in one engineer's shell history. A credential in one person's password manager. A queue created by hand in a console two years ago. A pricing rule that exists only in the head of the developer who wrote it.

Six months later the switching cost is genuinely high, and nobody did anything wrong on purpose. This is a checklist for preventing that, written to be dropped into a contract or a definition of done.

## Repository and build

- [ ] Source control is hosted in **your** organisation from commit one, not the vendor's.
- [ ] A new engineer can go from clone to running locally with one documented command.
- [ ] The command is tested by someone outside the core team at least quarterly.
- [ ] Build and test run in CI on every pull request, and the config lives in the repository.
- [ ] No build step depends on a machine, a personal account, or a manually installed local tool.

## Infrastructure

- [ ] All infrastructure is defined as code and applied through a pipeline, not a console.
- [ ] Environments can be recreated from the repository — and this has been demonstrated at least once.
- [ ] Cloud accounts, domains, DNS and certificates are owned by your legal entity.
- [ ] At least two people in your organisation hold administrative access to every account.
- [ ] Secrets live in a managed secret store, not in CI variables copied between projects by hand.

![Structured planning blocks representing process discipline](https://cdn.sanity.io/images/563mnkns/production/0f794e82a946443490259f40ecea59acb00c378b-1600x914.jpg)

## Documentation that survives people leaving

- [ ] Architecture decision records explaining **why**, not only what — one per irreversible decision.
- [ ] A system context diagram naming every external integration and its owner.
- [ ] A runbook per recurring operational task: deploy, rollback, restore, rotate credentials.
- [ ] An incident log with what happened and what changed afterwards.
- [ ] Data model documentation covering entity lifecycles and legal state transitions.

## Operational readiness

- [ ] Monitoring, alerting and error tracking are in your accounts, with alerts routed to your people.
- [ ] Backups exist **and a restore has been performed**, with the result recorded.
- [ ] Rollback is a documented procedure someone has actually executed, not a theory.
- [ ] On-call responsibilities and escalation paths are written down and agreed.

## AI-specific assets

Increasingly forgotten, and increasingly load-bearing:

- [ ] Prompts and prompt templates are versioned in the repository, not pasted into a console.
- [ ] Evaluation datasets and their expected outputs are committed alongside the code.
- [ ] Model provider accounts and API keys belong to your organisation.
- [ ] Retrieval indexes are rebuildable from source data by a documented pipeline.
- [ ] Cost and latency baselines per feature are recorded so a regression is visible.

## Contract terms that make the rest enforceable

- [ ] IP assigns to you on payment — code, designs, infrastructure definitions, prompts, evaluation data.
- [ ] Named key personnel cannot be substituted silently.
- [ ] A defined exit: notice period, documented handover, scheduled knowledge transfer window.
- [ ] A written change protocol if the engagement is fixed scope.

## The test that matters

Everything above reduces to one rehearsal you can run at any point in the engagement.

**Take an engineer who has never touched the project. Give them the repository and the documentation. Ask them to deploy to a staging environment.**

If they succeed, you are not locked in. If they need to ask the vendor a question, you have found exactly the gap worth closing this month — while it is still a documentation task rather than an emergency.

Run it once a quarter. It costs an afternoon and it is the only version of this checklist that cannot be gamed.

---

*This checklist is drawn from a longer buyer's guide covering engagement models, budgets, architecture decisions and evaluation questions: [Product Development Services: A 2026 Buyer's Guide for CTOs](https://techcirkle.com/blog/product-development-services). More on our [custom software development](https://techcirkle.com/development/custom-software-development) practice.*

## Frequently Asked Questions

### How does vendor lock-in usually happen?

Through accumulation rather than intent: undocumented deployment steps, credentials in personal accounts, infrastructure created by hand in a console, and business rules that exist only in one engineer's head. No single item is deliberate, but together they make switching expensive.

### What is the single best test for whether a project is portable?

Give the repository and documentation to an engineer who has never worked on it and ask them to deploy to staging. Any question they must ask the vendor identifies a documentation gap while it is still cheap to close.

### Who should own the cloud accounts and domains?

Your legal entity, with at least two administrators from your organisation on every account. This includes DNS, certificates, model provider accounts and error tracking — recovering these from a departed vendor is slow and occasionally impossible.

### What AI-specific assets belong in a handover?

Versioned prompts and templates, evaluation datasets with expected outputs, provider accounts under your organisation, a documented pipeline that rebuilds retrieval indexes from source data, and recorded cost and latency baselines per feature so regressions are detectable.

### What contract clauses protect the handover?

IP assignment on payment covering code, designs, infrastructure definitions and AI assets; named key personnel who cannot be swapped silently; and a defined exit with a scheduled knowledge-transfer window rather than a general promise of cooperation.

### How often should the handover rehearsal be run?

Quarterly. It takes an afternoon, and running it during the engagement converts potential emergencies into ordinary documentation work at a point when the people who know the answers are still available.
