# neutral

A syntax for cognitive assessments/activities/tasks that is neutral of implementation provider.

## Specifications and results

Neutral separates an assessment into two related, implementation-independent contracts:

### `neutral-spec`

`neutral-spec` describes **what an assessment is and how it should behave**, without tying that description to a particular experiment framework, application, programming language, or vendor.

A spec can define things such as:

- assessment metadata and versioning
- configurable parameters
- device or display requirements
- instructions and stimuli
- task, block, trial, and scene flow
- interaction and response behavior
- what raw and calculated data should be recorded
- scoring rules and quality flags

For example, the included Trail Making B and Stroop definitions declare `schema: neutral-spec/v0.1`. The implementation consuming the file is responsible for translating that neutral description into its own runtime behavior.

### `neutral-results`

`neutral-results` describes **the shape and meaning of the data returned after an assessment runs**. A spec can explicitly declare `format: neutral-results/v0.1` and identify the result sections that should be returned, such as metadata, parameters, trials or events, derived outputs, quality information, and provenance.

This provides a predictable boundary between running an assessment and consuming its data. An assessment engine can have its own internal event model, storage system, or UI implementation while still producing a common results representation for downstream analysis.

## Why separate the two?

The goal is portability at both ends of an assessment.

`neutral-spec` makes the **assessment definition portable**. The same conceptual task can be interpreted by different compatible engines instead of being rewritten around a specific library or provider.

`neutral-results` makes the **assessment output portable**. Analysis tools, databases, pipelines, and research applications can consume a consistent result contract without needing to understand how a particular engine implemented the task internally.

Keeping these contracts separate is useful because task execution and data analysis evolve independently. A renderer can change without requiring an analysis pipeline to change, and a results consumer does not need to know whether the assessment was implemented in JavaScript, a native mobile application, or another environment.

The separation also makes reproducibility more explicit. A result can retain the assessment metadata, parameters, quality information, and provenance needed to understand how it was produced, while the corresponding spec documents the intended task behavior and scoring rules.

---

In short:

**`neutral-spec` = portable description of what to run.**  
**`neutral-results` = portable description of what came back.**

The aim is not to claim that implementations are automatically psychometrically equivalent. Implementation details such as layout, timing, input device, screen size, and error handling can affect measurement and should be controlled when equivalence or normative interpretation matters.

See [template.md](template.md) and the [`activities`](activities/) folder for examples.