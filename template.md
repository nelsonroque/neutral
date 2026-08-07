---
schema: neutral-spec/v0.1
id: <assessment-id>
title: <Assessment Title>
version: 0.1.0
description: <Brief description of what the assessment measures or asks the participant to do.>
language: en-US
tags: []

notices:
  - <Optional implementation, interpretation, licensing, or equivalence notice.>

parameters:
  <parameter_name>:
    type: <integer|number|boolean|string|duration|proportion|viewport_fraction>
    default: <value>

requirements:
  input: [<keyboard|pointer|mouse|touch>]
  minimum_viewport: { width: <px>, height: <px> }
  fullscreen: <required|preferred|optional>
---

# Instructions

<Participant-facing instructions go here. Keep these instructions independent of any
specific implementation provider or software framework.>

```yaml task
stimuli:
  # Define stimuli or stimulus sets here.
  <stimulus_set>:
    - { id: <id>, value: <value> }

flow:
  - screen: instructions
    continue: { type: button, label: Begin }

  - block: practice
    use: <trial_or_scene_name>
    source: <stimulus_source>

  - block: test
    use: <trial_or_scene_name>
    source: <stimulus_source>

  - screen: completion
    content: "The task is complete."
    continue: { type: button, label: Continue }
```

```yaml trial <trial_or_scene_name>
# Use a `trial` block for trial-based tasks.
# For continuous/spatial tasks, this may instead be:
# ```yaml scene <scene_name>

variables:
  # Define trial-level derived variables here.

timeline:
  - id: stimulus
    show:
      type: <text|image|shape|custom>
      # Define the displayed stimulus.
    duration: $parameters.<duration_parameter>
    response:
      type: <key|pointer|touch|choice>
      # Define valid responses and correctness.
      timestamp_from: stimulus.actual_onset

record:
  raw:
    - session_id
    - participant_id
    - phase
    - block_index
    - trial_index
    - response.value
    - response.correct
    - response.rt
    - response.timestamp
    - stimulus.actual_onset

  calculated:
    <derived_variable>: <expression>
```

```yaml feedback <feedback_name>
# Optional. Remove this block if the assessment does not use feedback.

scope: practice
rules:
  - when: <condition>
    show:
      type: text
      text: <Feedback text>
    duration: $parameters.<feedback_duration_parameter>
```

```yaml evaluate <evaluation_name>
# Optional. Use for practice criteria, branching, stopping rules, etc.

<metric>: <expression>
<decision>: <expression>
```

```yaml score
population: "$trials where phase == 'test'"

exclusions:
  # Optional scoring exclusions.
  <rule_name>: <expression>

outputs:
  trial_count: "count($population)"
  accuracy: "mean($population.response.correct)"
  mean_correct_rt: "mean($population where response.correct, response.rt)"

  # Add assessment-specific outputs here.

quality:
  flags:
    - when: <quality_condition>
      code: <quality_flag_code>

return:
  format: neutral-results/v0.1
  include:
    - metadata
    - parameters
    - trials
    - outputs
    - quality
    - provenance
```
