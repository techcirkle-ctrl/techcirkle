---
layout: post
title: "Integration Contracts for Manufacturing ERP: MES, PLM, WMS and Quality"
date: 2026-09-14 00:00:00 +0000
categories: ["Architecture", "Integration"]
tags: ["erp", "integration", "manufacturing", "architecture"]
description: "Define which system owns which record before you select an ERP. A reference set of integration contracts for MES, PLM, WMS and quality systems."
image: "https://cdn.sanity.io/images/563mnkns/production/7b7a40690516f1d0fe68c810952e737076bca2ab-1600x848.jpg"
author: "TechCirkle Editorial Team"
---

![Digital factory production line](https://cdn.sanity.io/images/563mnkns/production/7b7a40690516f1d0fe68c810952e737076bca2ab-1600x848.jpg)

Almost every difficult manufacturing integration I have worked on was difficult for the same reason, and it was not the protocol.

It was that two systems both believed they owned the same record, nobody had written down which one was right, and the disagreement only surfaced under conditions — a partial receipt, a mid-build revision change, a lot split — that nobody had modelled during selection.

Protocols are solvable. Ownership ambiguity is not, at least not cheaply, because resolving it after go-live means changing behaviour in systems that are now load-bearing.

This post sets out a reference set of integration contracts worth agreeing before an ERP is chosen, not after.

## The general form

For each boundary, write down four things:

1. **Owner** — which system holds the authoritative version of this record.
2. **Direction** — which way data flows, and whether the reverse direction exists at all.
3. **Trigger** — what event causes the flow, and whether it is event-driven or scheduled.
4. **Conflict rule** — what happens when both sides changed, because eventually both sides will.

If you cannot fill in the fourth column, the contract is not finished.

## ERP and PLM

**Owner:** PLM owns part definition, revision and engineering BOM. ERP owns the manufacturing BOM derived from it.

**Direction:** PLM to ERP on release. The reverse direction should not exist. If production is editing BOM structure in ERP, engineering has lost control of the product definition and you have two sources of truth.

**Trigger:** Engineering change order reaching released state.

**Conflict rule:** ERP-side structural changes are rejected, not merged. Production-specific additions — consumables, packaging, scrap allowances — live in a designated ERP-only segment of the structure that PLM never overwrites.

The common failure is treating revision as a string rather than as a state machine. Define explicitly what happens to work in progress, to existing stock and to open purchase orders when a revision releases mid-build. That single scenario belongs in your evaluation pack.

## ERP and MES

**Owner:** ERP owns the work order, the material commitment and the cost. MES owns execution state, operator interaction and machine data.

**Direction:** Bidirectional, and this is the boundary where discipline matters most.

- ERP to MES: work order release, routing, quantities, required materials, specification references.
- MES to ERP: operation confirmations, quantities produced and scrapped, labour and machine time, material consumption.

**Trigger:** Release on the outbound side. Confirmation events on the inbound side, buffered at the edge so a connectivity failure does not lose production data.

**Conflict rule:** MES wins on what physically happened. ERP wins on what was authorised. When a confirmation exceeds the authorised quantity, the ERP records the reality and raises an exception rather than rejecting the confirmation — the parts exist regardless of what the order said.

Requirements worth writing into the contract:

- Confirmations must be idempotent. Network retries are guaranteed, duplicate postings are not acceptable.
- Confirmations must queue during ERP downtime and replay in order.
- Reconciliation must run continuously, comparing MES-recorded output against ERP-posted quantities, with alerting on drift rather than discovery at month end.

![Colleagues reviewing operational data](https://cdn.sanity.io/images/563mnkns/production/e5cccbe31492e9db78fcc82da6265c90c5b9ec10-1600x1068.jpg)

## ERP and WMS

**Owner:** WMS owns location-level movement and task management. ERP owns the inventory balance and its valuation.

**Direction:** ERP to WMS for expected receipts and shipment requirements. WMS to ERP for confirmed movements.

**Trigger:** Purchase order and delivery events outbound; putaway, pick and shipment confirmation inbound.

**Conflict rule:** On cycle count discrepancy, WMS is authoritative for quantity at location and ERP records the adjustment with its financial consequence. Never allow both systems to independently adjust the same balance.

The frequent mistake is running both systems with full inventory models and reconciling nightly. That works until a partial pick or a cross-dock, at which point the reconciliation becomes a permanent manual process that somebody performs every morning forever.

## ERP and quality

**Owner:** Quality system owns inspection results, non-conformance workflow and disposition decisions. ERP owns the inventory hold, the scrap posting and the cost consequence.

**Direction:** ERP to quality for inspection requirements on receipt and production. Quality to ERP for disposition outcomes.

**Trigger:** Goods receipt, operation completion, and customer return.

**Conflict rule:** A quality hold cannot be released in ERP. If the only way to release stock is through the quality system, your audit trail is coherent by construction, which is the entire point of separating them.

## Machine data

Deliberately not on the enterprise integration backbone.

Sample rates differ from business events by three orders of magnitude. Terminate telemetry at an edge tier, aggregate, and publish only derived business-meaningful events into the integration layer. Sizing an enterprise bus for sensor traffic is a way of making both workloads expensive. Our [cloud application development guide](https://techcirkle.com/blog/cloud-application-development-guide) covers that split in more depth.

## Testing the contracts before selection

Convert each contract into a scenario and run it against candidate systems in a configured environment with a sample of your data.

- Release a revision mid-build and observe what happens to WIP, stock and open orders.
- Confirm an operation twice and check for duplicate posting.
- Disconnect the MES link during a shift, continue confirming, reconnect and verify order and completeness.
- Post a partial receipt against a short shipment and follow the reallocation.
- Place a lot on quality hold and attempt to ship it.

Vendors handle these very differently, and the differences almost never appear on a requirements matrix.

## Where this leaves selection

If you are architecting this way, weight API completeness, event availability, idempotency support and environment tooling far above functional module breadth. A narrower ERP with excellent interfaces serves this structure better than a broad suite with a closed one — you are buying a well-behaved core, not a perimeter.

---

## Frequently Asked Questions

**What belongs in an integration contract?**
Record owner, flow direction, triggering event, and the conflict rule for when both sides changed. The fourth item is the one most often omitted and most often needed.

**Who owns the BOM, PLM or ERP?**
PLM owns the engineering definition and revision state; ERP owns the manufacturing BOM derived from it. Structural edits should not flow back from ERP.

**What happens when MES reports more output than the ERP order authorised?**
Record the physical reality and raise an exception. The parts exist regardless of what the order permitted; rejecting the confirmation loses production data.

**Should machine telemetry use the enterprise event bus?**
No. Aggregate at an edge tier and publish only derived business events, or you will size an enterprise backbone for sensor traffic.

**How do we test these contracts before buying?**
Turn each one into a scenario — mid-build revision release, duplicate confirmation, link failure and replay, partial receipt, quality hold — and run them in a configured environment with your own data.

---

*Full buyer-side guide: [Best ERP Software for Manufacturing in 2026](https://techcirkle.com/blog/best-erp-software-for-manufacturing).*
