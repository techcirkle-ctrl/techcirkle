---
layout: post
title: "Engineering Ambient AI Documentation for Telehealth Visits"
date: 2026-11-05 00:00:00 +0000
categories: ["Engineering", "AI"]
tags: ["llm", "speech-to-text", "healthtech", "evaluation", "privacy"]
description: "Engineering notes on ambient AI scribes for telehealth: audio capture, BAA chains, prompt and log retention, human review and evaluation metrics."
image: "https://cdn.sanity.io/images/563mnkns/production/42b08fc33ef6be5c1ee38a74dd1d84b4495379ee-1600x1067.jpg"
author: "TechCirkle"
---

Ambient documentation sounds like a prompt and a text box. In a telehealth product it is really a distributed pipeline carrying some of your most sensitive data through several vendors, ending in a legal record signed by a human. Here are design notes for building it, engineer to engineer. Configs and code are illustrative, and anything touching consent or retention needs sign-off from healthcare counsel.

![Older doctor at an office desk speaking with a patient through a laptop video consultation](https://cdn.sanity.io/images/563mnkns/production/42b08fc33ef6be5c1ee38a74dd1d84b4495379ee-1600x1067.jpg)

## The pipeline at a glance

```text
consent gate -> per-track audio -> speech-to-text -> transcript store
    -> context assembly -> LLM via gateway -> draft note
    -> clinician review -> sign -> EHR write-back -> eval sampling
```

Each arrow is a place where data crosses a boundary, and each boundary needs an owner, a retention rule and a failure mode.

## Audio: use your telehealth advantage

In-room scribes have to separate voices from one microphone. You usually don't, since a video session can give you a track per participant, which beats diarization for speaker attribution. Check what your provider exposes, as some offer only composite recordings.

Some design choices to make early:

- **Consent is a precondition, not a banner.** Capture must not start until consent is recorded for this encounter, including any requirement in all-party consent states. Store the consent template version alongside it.
- **Participant roles travel with tracks.** Label each track `clinician`, `patient`, `caregiver` or `interpreter`. Interpreter visits matter here, because the English rendering and the patient's original speech may differ. Decide explicitly which one the note uses, and test the case.
- **Handle degraded calls.** Packet loss and audio-only fallback produce gaps. Mark low-confidence segments and carry those markers to the draft instead of allowing the model to smooth over missing speech.

## The BAA chain is a dependency graph

Treat vendor coverage as something you can query. Convenience SDKs are the classic hazard, because one error tracker or analytics library can capture a transcript fragment and ship it to a party with no agreement in place.

```yaml
# illustrative data-flow register, reviewed on every new dependency
- service: video_provider
  data: [audio, participant_metadata]
  baa: signed
- service: speech_to_text
  data: [audio, transcript]
  baa: signed
  training_on_customer_data: contractually_excluded
- service: llm_provider
  data: [transcript, problem_list]
  baa: signed_for_this_endpoint
- service: error_tracker
  data: [scrubbed_stack_traces]
  baa: signed
  scrub_rule: deny_request_bodies
```

Some practical checks:

- Confirm that the BAA covers the *specific* endpoint and feature you call, since a provider's coverage may not extend to every product it sells.
- Fail CI when a new outbound domain or SDK appears without a register entry.

## Prompts and logs are PHI too

Once a transcript goes into a prompt, your prompt log becomes a copy of the patient record, and it deserves the same controls. The common mistake is piping full LLM traces into a general-purpose observability tool during development and never removing that path.

A workable separation:

| Artifact | Sensitivity | Suggested handling |
|---|---|---|
| Raw audio | Highest | Do not keep by default. Short TTL only if there is a defined reason. |
| Transcript | PHI | Encrypted store, retention set with counsel |
| Prompt template | Not PHI by itself | Versioned in Git, referenced by ID |
| Rendered prompt and response | PHI | Encrypted, access-logged, short retention or hash only |
| Draft and final note | Clinical record | Retained under medical records policy |
| Eval scores, token counts, latency | Ideally de-identified | Normal observability |

Reference prompts and models by version in a small per-encounter record, so you can reconstruct what happened without keeping every rendered payload forever.

Retention periods should be a real decision reached with your compliance team and counsel, because every retained artifact can add breach exposure and discovery risk. Route everything through a single [LLM gateway](https://techcirkle.com/llm-integration) so these rules live in one place, not in each feature.

![Stethoscope, open notepad and smartphone arranged on a blue background](https://cdn.sanity.io/images/563mnkns/production/f644f5714f27ecaada8b850c6251f715fe45ea68-1600x900.jpg)

## Human review as an enforced state

Make "no signature, no record" a system invariant. Drafts live in their own state and never reach the EHR by themselves.

- `draft_generated -> under_review -> signed -> written_back`
- Only a signed note triggers write-back.
- Store the draft and the signed version, so diffs can be calculated later.
- Keep the review UI inside the clinician's workspace. A draft requiring copy and paste into another system throws away much of its value, so test write-back early.
- Any patient-facing generated text needs its own review and disclosure logic. California's AB 3030 requires disclosure on certain generative AI clinical communications, so look it up through your policy layer rather than hard-coding it.
- If the model or speech service is down, the clinician falls back to a manual template, and the visit carries on unaffected.

## Evaluation metrics that mean something

Offline checks and production sampling answer different questions, so you need both. Build a gold set from properly governed data, with clinicians labeling errors against source audio, and run it for every prompt or model change.

Metrics worth tracking:

- **Invention rate:** statements with no basis in the conversation, per note
- **Omission rate:** clinically relevant items that should appear and do not
- **Misattribution rate:** statements assigned to the wrong speaker
- **Severity-weighted errors:** a wrong dose counts for more than awkward phrasing
- **Edit ratio:** normalized difference between draft and signed note
- **Time to sign,** and the amount of after-hours charting
- **Subgroup slices:** by language, accent, interpreter use, audio quality and specialty

A falling edit ratio may mean better drafts, or may mean clinicians are skimming. So correlate it with independent audits, and consider planting checks in staging environments to test how reviewers behave. Set your release thresholds with clinical leadership. There is no universal acceptable error rate to copy from a blog post.

Also watch for silent drift. Pin model versions where the provider allows it, and treat any provider update like a dependency upgrade that has to pass the suite.

## Further reading

This piece looks only at one subsystem. For how it fits into scope, regulation and planning-level budget, see this [guide to telehealth app development services](https://techcirkle.com/blog/telehealth-app-development-services). If you expect the system to act as well as draft, such as filing orders or chasing follow-ups, the permissions and rollback questions covered under [agentic workflow development](https://techcirkle.com/agentic-workflow-development) become relevant.

## Frequently Asked Questions

### Should we run speech-to-text ourselves or use a vendor?

A BAA-covered vendor is usually the sensible start, because accuracy tuning and scaling are significant work. Self-hosting may suit you if you have strict data residency needs and the staff to run and evaluate models properly.

### Do we need to keep the audio after the note is signed?

Often not, and keeping it increases exposure. There can be valid reasons for short retention, such as evaluation under a defined program. Set the rule with counsel and your compliance team, and enforce it automatically.

### How can we evaluate without storing lots of PHI?

Use a smaller governed sample with strict access controls and time limits, and rely on de-identified metrics for routine monitoring. De-identification should be validated, since transcripts can hold identifying details in free speech.

### What should block a prompt or model release?

Regressions on severity-weighted errors, invention or omission rates, or on any tracked subgroup should block it, using thresholds agreed with clinical leadership. Treat prompts as versioned code that has to pass the same gate as any other change.

### How do we manage interpreter-assisted visits in the pipeline?

Label the interpreter as a separate participant, decide which language stream feeds the note, and evaluate those visits as their own group. Mixed-language audio may behave quite differently from your general test set.
