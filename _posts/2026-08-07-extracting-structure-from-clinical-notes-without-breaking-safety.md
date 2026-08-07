---
layout: post
title: "Extracting Structure From Clinical Notes Without Breaking Safety"
date: 2026-08-07 00:00:00 +0000
categories: ["Engineering", "Machine Learning"]
tags: ["nlp", "healthcare", "engineering", "llm"]
description: "LLMs made coding a decade of clinical notes affordable. The provisional-layer pattern that keeps extraction safe and useful at scale."
image: "https://cdn.sanity.io/images/563mnkns/production/a5e80ca8bcbd9e4ff55ad2b999934dee21032094-1600x1001.jpg"
author: "TechCirkle Editorial Team"
---

Most healthcare organisations have a decade or more of clinical narrative sitting in their database as text. Diagnoses, medication changes, findings, plans — all of it clinically meaningful and none of it queryable as data.

![hero](https://cdn.sanity.io/images/563mnkns/production/a5e80ca8bcbd9e4ff55ad2b999934dee21032094-1600x1001.jpg)

Historically this stayed that way for a simple reason: converting it meant clinical coders reading records by hand. Expensive enough that almost nobody did it at scale, so the archive stayed inert and everyone accepted that historical data was searchable only as free text.

Language models changed that economically, and the change is genuinely large. They also made it trivially easy to poison a medical record. This post is about the gap between those two facts.

## Why this is not a normal extraction task

If you extract entities from support tickets and get 3% wrong, you have a slightly noisy dataset.

If you extract clinical facts from notes and get 3% wrong, you have introduced errors into a medical record — and the errors have a specific, unhelpful character. Language models do not fail by producing obvious garbage. They produce fluent, plausible, well-formed clinical assertions that happen to be false. A hallucinated medication at a plausible dose looks exactly like a real one.

Worse, the failure is silent at the point of use. A clinician reading a problem list has no signal distinguishing a coded fact extracted by a model from one a colleague entered.

So the engineering question is not "how do we maximise extraction accuracy." Accuracy will not reach a level where unreviewed output is safe to treat as authoritative. The question is how to make extracted data useful **while keeping it structurally distinguishable from clinical truth**, permanently.

## The provisional layer

The pattern that works keeps extracted facts in a separate layer that is never silently merged into the authoritative record.

```
narrative note
      │
      ▼
  extraction  ──►  provisional_assertions
                          │
              confidence-gated review
                          │
                    ┌─────┴─────┐
                 confirmed   rejected
                    │
                    ▼
        surfaced as clinician-confirmed,
        source still recorded permanently
```

Every provisional assertion carries:

```json
{
  "assertion_id": "01J9F2K7QX8N3M4P5R6S7T8V9W",
  "patient_id": "...",
  "source_document_id": "note-2019-04-17-8821",
  "source_span": { "start": 1420, "end": 1476 },
  "concept": { "system": "SNOMED", "code": "44054006" },
  "asserted_at": "2019-04-17T00:00:00Z",
  "extracted_at": "2026-08-08T11:02:00Z",
  "extractor_version": "clinical-extract-2.3.0",
  "confidence": 0.82,
  "confirmation_status": "unconfirmed"
}
```

Three fields are doing the safety work.

**`source_span`** — the character offsets in the original note. This is what makes review tractable. A reviewer does not read the whole note; they see the highlighted span that produced the assertion and judge whether it supports the claim. It is also your only defence against a hallucinated fact with no textual basis at all — if the span does not contain supporting text, the assertion is fabricated, and that is mechanically checkable.

**`extractor_version`** — you will improve the extractor and re-run it. Without version attribution you cannot tell which assertions came from which model, cannot compare their accuracy, and cannot selectively invalidate a bad batch.

**`confirmation_status`** — never inferred. A clinician viewing a screen is not confirmation; only an affirmative action is.

![alt](https://cdn.sanity.io/images/563mnkns/production/d76b8cab0e5f5bd0958b7066084d4a3bc4fcd0c8-1600x1065.jpg)

## Confidence gating and its limits

Confidence scores route work: high-confidence assertions to spot-check sampling, mid-range to full review queues, low-confidence to discard or to a coder.

One caveat worth being blunt about. Model confidence is not calibrated probability of correctness, and treating it as such will hurt you. It correlates with correctness well enough to prioritise a queue and not well enough to justify auto-confirming anything.

Calibrate empirically against a human-reviewed sample from *your* notes, and re-calibrate whenever the extractor version changes. A threshold tuned for one model version is not valid for the next one.

## Rules that are not negotiable

**Nothing becomes authoritative through inaction.** Auto-confirm timeouts are exactly the wrong pattern. An unreviewed assertion stays unreviewed indefinitely — that is a backlog problem, and a backlog is much better than silently promoting unverified clinical claims.

**Unconfirmed renders differently everywhere.** Not just in the review queue. Every view — problem list, medication list, summary, API response — must distinguish provisional from confirmed. If they render identically anywhere, you have built the unsafe version with extra steps.

**Provenance survives confirmation.** Once confirmed, the assertion is clinically usable, but the record must still show it originated from machine extraction of a specific note. That matters for audit, for later quality review, and for the day you discover an extractor version had a systematic bias.

**Extraction never modifies existing records.** It only adds assertions. If extraction suggests a medication dose differing from the current record, that is a conflict to surface, not a value to update.

## What it unlocks

With this in place the economics genuinely work.

A decade of notes becomes a queryable longitudinal dataset — which is precisely what clinical AI needs and precisely what most organisations lack. Cohort identification for research stops requiring manual chart review. Quality measures can be computed rather than sampled. And the retrospective data needed to train or validate a model exists in structured form.

The reason to build the provisional layer first is that it is the same infrastructure you need for two other things: reconciling records arriving from other organisations, and accepting writes from ambient documentation tools. All three require the same primitive — a clinical fact as an attributed assertion rather than a bare value.

Build it once for extraction and the other two become mostly free. Build extraction as a bulk import into the primary record and you have a liability that surfaces at the worst possible moment, plus two future migrations.

Full guide: [EHR vs EMR: What the Difference Means for Your Build](https://techcirkle.com/blog/ehr-vs-emr).

## Frequently Asked Questions

### Why can extracted clinical facts not go straight into the record?

Because language models produce fluent, plausible, confidently wrong output, and a hallucinated medication at a realistic dose is indistinguishable from a genuine entry once written. Extracted facts must remain structurally separable from clinician assertions.

### What does storing the source span achieve?

It makes review tractable by showing the reviewer exactly which text produced the assertion, and it provides a mechanical check against fabrication — if the referenced span contains no supporting text, the assertion has no basis in the note.

### Can confidence scores be used to auto-confirm assertions?

No. Model confidence correlates with correctness well enough to prioritise a review queue but is not calibrated probability of correctness. Calibrate empirically against human-reviewed samples from your own notes, and re-calibrate whenever the extractor version changes.

### Why version the extractor on every assertion?

Because extractors get improved and re-run. Without version attribution you cannot compare accuracy between versions, cannot identify which assertions a given model produced, and cannot selectively invalidate a batch found to be systematically wrong.

### What should happen when extraction conflicts with the existing record?

The conflict should be surfaced as an additional attributed assertion, never applied as an update. Extraction adds claims about what a note said; it does not have the authority to modify what the record currently holds.

### Does the provenance record matter after a clinician confirms a fact?

Yes. The origin must remain visible for audit purposes, for later quality review, and for the case where a specific extractor version is found to have a systematic bias and its outputs need re-examination.
