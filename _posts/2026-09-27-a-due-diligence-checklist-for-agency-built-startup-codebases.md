---
layout: post
title: "A Due Diligence Checklist for Agency-Built Startup Codebases"
date: 2026-09-27 00:00:00 +0000
categories: ["Engineering", "Startups"]
tags: ["duediligence", "codequality", "security", "startups"]
description: "A runnable checklist for auditing a startup codebase built by an external agency, covering reproducibility, tests, secrets, dependencies and architecture."
image: "https://cdn.sanity.io/images/563mnkns/production/b5b921f3b144af016337b6671b67b9e757b22a1e-1600x1066.jpg"
author: "TechCirkle"
---

![Engineers reviewing a codebase together](https://cdn.sanity.io/images/563mnkns/production/b5b921f3b144af016337b6671b67b9e757b22a1e-1600x1066.jpg)

Someone hands you a repository built by an agency over the last eighteen months and asks whether it is any good. You have a week. This is the checklist I work through, roughly in order of how much signal each item gives per hour spent.

## Reproducibility first

Before reading any code, try to run it.

Clone the repository fresh, follow only the README, and time how long until you have it running locally with a working database and can make a visible change.

Under an hour is excellent. Half a day with a couple of questions is acceptable. Two days with multiple conversations means the project has real portability debt, and that debt shows up as a four-month onboarding for every future hire rather than a one-month one.

This check is disproportionately informative because environment reproducibility correlates tightly with documentation quality, dependency hygiene, and whether the team has ever onboarded anyone at all.

## Secrets and configuration

Scan the history, not just the working tree. Credentials committed once and removed later remain in the history and remain compromised.

Check for API keys, database URLs with embedded passwords, private keys, and cloud credentials. Then check where secrets legitimately live — a managed secrets store is good, environment variables injected at deploy are acceptable, a file passed around in chat is not.

Also check that production and development configuration are genuinely separate and that a developer cannot accidentally point local tooling at production data. This sounds obvious and is violated regularly.

## Dependencies

Three things to measure.

Freshness: how far behind are the major dependencies, and is anything on a version that no longer receives security patches. A framework two major versions behind is a multi-week upgrade project sitting quietly in the backlog.

Known vulnerabilities: run whatever audit tooling the ecosystem provides and separate the noise from the genuinely reachable issues.

Surface area: count the direct dependencies. A small project with two hundred direct dependencies has an upgrade and supply-chain problem that will only grow.

## Tests, weighted by consequence

Total coverage percentage is a weak signal and easy to game. Coverage on specific paths is a strong one.

Look at the code that handles payment, authentication, authorisation checks, and personal data. Ask whether a wrong result on each path would be annoying or expensive. Expensive paths without tests mean nobody can safely change anything near them, which is how a system calcifies — future engineers route around the untested core rather than touching it.

In 2026 there is a specific reason to weight this more heavily. Generated implementations of permission checks and boundary conditions are frequently plausible and wrong in exactly one branch. Happy-path tests do not catch that. Boundary and property-based tests do.

## Commit history

Two things: author distribution and commit granularity.

If one author wrote ninety percent of the code, the model of the system lives in one person's head and their departure is an event rather than an inconvenience.

If commits are feature-sized with messages like `updates`, nobody expected this history to be read, which usually means it cannot be used for the thing history is actually for — understanding why something is the way it is.

## Written reasoning

Identify the three most expensive decisions to reverse. Typically the data model for the core entity, the authentication and authorisation approach, and the hosting and deployment topology.

Now look for the written reasoning behind each: alternatives considered, constraints that drove the choice, conditions under which it should be revisited.

Where that writing is absent, future engineers will either preserve a decision nobody can justify or relitigate it from first principles. Architecture decision records take about twenty minutes each at the time and are effectively unreconstructable a year later.

## Architecture against stage

Check the architecture against the company's actual scale, not its ambitions.

A pre-product-market-fit company with microservices, event sourcing and multi-region deployment is carrying operational complexity it cannot afford and did not need. The appropriate shape at that stage is one well-structured application, one database, a background job runner, and clear internal module boundaries that permit splitting later.

Conversely, check for the things that genuinely should be there early and often are not: structured logging, error tracking, reversible migrations, and product event instrumentation.

## Operational readiness

Break something small in staging and watch what happens.

Point the database at a bad credential, or remove a required environment variable. Does the failure surface clearly somewhere a human will see it, or does it fail silently and get reported by a customer three days later?

Then check the deployment path. Is there a pipeline running tests on every change, or a deploy button? Are migrations automated and reversible? Is there a rollback that anyone has actually used?

## Ownership

Not code, but it belongs in the same report because it affects every option the company has.

Repository organisation, cloud account and billing, domain and DNS, app store developer accounts, payment provider, error tracking and analytics. Anything in the agency's name rather than the company's is both leverage during a disagreement and a migration project during a transition.

## Write two pages

Score each area from one to five, weight reproducibility and consequence-weighted test coverage double, and give a blunt bottom line: could a new team of two be productive here within a month, and what would it cost to make that true.

That last question is the one the founder actually needs answered.

Longer, buyer-side version: [How to Choose a Software Development Company for Startups in 2026](https://techcirkle.com/blog/software-development-company-for-startups). Related: our overview of [custom software development](https://techcirkle.com/development/custom-software-development).

## Frequently Asked Questions

### What is the fastest high-signal check on an unfamiliar codebase?

Clone it fresh and time how long until it runs locally from the README alone. Reproducibility correlates strongly with documentation quality, dependency hygiene and onboarding cost.

### Why scan git history for secrets rather than just the working tree?

Credentials committed once and removed in a later commit remain in the history and remain compromised. Removal from the current tree does not rotate the key.

### Is total test coverage a useful metric?

Not on its own. Coverage weighted by consequence is far more informative — specifically the paths handling payment, authentication, authorisation and personal data, where boundary and property-based tests catch failures that happy-path tests miss.

### What architecture should a pre-PMF startup have?

One well-structured application, one database, a job runner, and clear internal module boundaries. Microservices and multi-region deployment at that stage impose operational cost with no corresponding benefit.

### What belongs in the final report?

Scores per area with reproducibility and consequence-weighted coverage double-weighted, plus a direct answer to whether a new team of two could be productive within a month and what it would cost to make that true.
