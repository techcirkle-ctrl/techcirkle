---
layout: post
title: "An Artifact Checklist for Engineering Vendor Due Diligence"
date: 2026-08-04 00:00:00 +0000
categories: ["Engineering", "DevOps"]
tags: ["devops", "ci", "due-diligence", "architecture", "engineering-management"]
description: "A concrete list of artifacts to request from a prospective engineering partner, what each one reveals, and what a bad version looks like."
author: "TechCirkle Editorial Team"
---

Technical due diligence on a vendor fails for a structural reason: the interview format rewards answers, and every answer in this domain is free to give. "Do you test?" has one possible response. So does "is your team senior?" You can spend an hour and acquire nothing.

Artifacts do not have this problem. They exist or they do not, and they are hard to produce retroactively.

This is the checklist, with what each item reveals and what a weak version looks like.

---

## 1. CI pipeline configuration

**Request:** the actual pipeline file from a recent project, redacted for secrets and client identifiers.

**Read for:**

- Tests as a **merge gate**, not a job permitted to fail
- Lint and type-check stages that block
- Dependency audit, secret scanning, or SAST present at all
- Migrations exercised somewhere before production
- Total pipeline duration

**Weak version:** a description of their pipeline rather than the file. Or a file where the test job has `continue-on-error` or its equivalent, which is a red pipeline wearing a green badge.

**Note on duration:** anything over roughly fifteen minutes changes behaviour. Engineers batch changes into larger commits and merge on optimism. Read the number as a proxy for daily working style.

---

## 2. An architecture decision record

**Request:** one real ADR, ideally where they chose the less obvious option.

**Read for:** a consequences section containing something the author is not happy about. Real decisions have costs; documented decisions that only list benefits were written to satisfy a process, not to think.

**Weak version:** no ADRs at all, or a template with every field filled optimistically. If they cannot produce one, the reasoning behind your future system will exist only in the heads of people who will eventually leave.

---

## 3. A production incident timeline

**Request:** what broke in production on your last engagement — detection, timeline, resolution, follow-up change.

**Read for:** an alert as the detection mechanism rather than a customer email. Real numbers rather than adjectives. A concrete change to the system or process afterwards. Ideally something they admit getting wrong during the response.

**Weak version:** "nothing significant." Everything breaks. That answer means either no users or nobody watching, and both are worth knowing.

This is the single highest-signal question in the list. Engineering culture is visible in how a team describes a bad day.

---

## 4. AI-assisted code review policy

**Request:** what proportion of production code is AI-assisted, and what is the review gate.

**Read for:** mechanisms. Diff size limits. Mandatory human review on authentication, authorisation, payments, personal data, and migrations. Coverage gates before merge. A requirement that tests for generated implementation be authored independently rather than in the same generation context.

**Weak version:** enthusiasm without specifics.

**Why it discriminates:** the industry has not converged here, so answers are genuinely varied. The underlying problem is real — code volume arriving for review rose while review capacity did not, and generated code fails differently from human-written code of equivalent size. A team that has not adjusted its process is accumulating a review deficit whether or not it can feel it yet.

---

## 5. Production AI system, if AI features are in scope

**Request:** a live system with a model in the request path serving real users, with usage numbers.

**Follow-ups:**

- What regression did your evaluation harness catch?
- What is cost per user session, and how did you get it there?
- What happens when the provider degrades — not fails, degrades?

**Read for:** immediate, specific answers. All three of these hurt to learn, so anyone who has learned them answers without pausing.

**Weak version:** a proof of concept, a demo, or a pivot to describing internal tooling. Using AI to build software and building software with AI in it are unrelated competencies; the conflation is deliberate and commercially useful.

---

## 6. Named delivery team

**Request:** the specific engineers who will be assigned, what they shipped most recently, and that list written into the statement of work.

**Read for:** willingness to commit contractually, plus a clause requiring written approval for changes and a minimum commitment period for the technical lead.

**Weak version:** roles rather than people. "Two senior engineers, three mid-level" is a staffing plan, not a team, and it permits substitution by construction.

Team substitution after signature is the most frequently reported problem in this market and it is entirely preventable at contract stage.

---

## 7. Infrastructure and repository arrangement

**Request:** confirmation, in writing, of where each of these lives during the engagement:

- Source repositories
- Cloud accounts and root credentials
- CI/CD platform and secret storage
- Monitoring and log aggregation
- Domain registrar
- Third-party service accounts — payments, email, error tracking, feature flags

**Correct answer:** all in your organisation, from the first commit, with vendor engineers holding scoped access.

**Weak version:** "we manage it during development and transfer at handover." That transfer loses issue history, pull request discussion, branch protection rules, and CI configuration, and it converts a handover into a negotiation.

The third-party account list is the one that gets skipped. An error tracker or payment processor account registered to a vendor email domain is a small problem right up until it is the only problem.

---

## 8. Transition and exit terms

**Request:** the termination section, rewritten as a technical specification.

**Should contain:** notice period, defined transition window, a concrete artifact list (architecture documentation, runbooks, ADRs, deployment procedures, credential inventory, known-issue register), knowledge transfer at pre-agreed rates, and explicit confirmation that no production dependency runs on vendor-controlled infrastructure.

**Verify rather than accept** that last point. It is common for a scheduled job, a build agent, or a monitoring integration to be running in the vendor's environment because it was convenient at the time. It is discovered when it stops.

---

## Using this

You do not need to be the decision maker. Request the eight items, hand the folder to whoever is deciding, and let the gaps speak for themselves. Every serious firm has been asked before and can produce most of this within a few days. A vendor that treats routine technical due diligence as an affront has told you something useful about how scrutiny will be received during delivery.

---

Full buyer-side guide with delivery models, current US rate bands, compliance expectations and twelve-month engagement economics: **[Product Engineering Services Companies in USA](https://techcirkle.com/blog/product-engineering-services-companies-usa)**.

## Frequently Asked Questions

### What if the vendor will not share a CI configuration?

A redacted pipeline file contains no client-identifying information, so refusal usually means it does not exist or it is embarrassing. Offer to accept it with all names, URLs and secrets stripped, then note the response either way.

### How many artifacts is reasonable to request?

All of them. Every established firm has faced this before. The request itself is a test: how a vendor responds to routine scrutiny during sales predicts how they respond to it during delivery.

### What does a good architecture decision record look like?

It states the decision, the alternatives considered, and — critically — consequences that include real costs the team accepted. Uniformly positive ADRs indicate documentation produced for process compliance rather than for thinking.

### Why is the incident question so revealing?

Because "nothing broke" is never true, so the answer is a direct measure of both observability and candour. A good answer contains an alert, a timeline with numbers, and a change made afterwards.

### Do repositories really need to be in our organisation from day one?

Yes. It costs one conversation at kickoff. Doing it later loses issue history and pull request discussion, requires rebuilding CI configuration, and gives the handover a negotiating position it should not have.

### Which third-party accounts get forgotten most often?

Error tracking, email delivery, feature flags, and payment processors. Each tends to be created quickly by whoever needed it, under whatever email address was convenient, and each becomes a single point of control nobody audited.
