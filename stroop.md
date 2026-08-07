---
schema: neutral-spec/v0.1
id: stroop-color-word
title: Stroop Color-Word Test
version: 0.1.0
description: Identify the ink color while ignoring the color word.
language: en-US
tags: [executive-function, inhibition, reaction-time]

parameters:
  practice_repetitions:
    type: integer
    default: 1
    minimum: 1
  test_repetitions:
    type: integer
    default: 4
    minimum: 1
  fixation_duration:
    type: duration
    default: 500ms
  stimulus_duration:
    type: duration
    default: 2000ms
  inter_trial_interval:
    type: duration
    default: 500ms
  feedback_duration:
    type: duration
    default: 750ms
  minimum_practice_accuracy:
    type: proportion
    default: 0.75
  maximum_practice_attempts:
    type: integer
    default: 2
  random_seed:
    type: string
    required: false

requirements:
  input: keyboard
  color_vision: required
  minimum_viewport: { width: 640px, height: 480px }
  fullscreen: preferred
---

# Instructions

You will see a color word displayed in colored ink. Respond to the **ink
color** and ignore the word itself.

- Press **R** for red.
- Press **G** for green.
- Press **B** for blue.
- Press **Y** for yellow.

Respond as quickly and accurately as you can. If you are unsure, make your
best response before the stimulus disappears.

```yaml task
stimuli:
  colors:
    red:    { label: Red,    value: "#D32F2F", response: r }
    green:  { label: Green,  value: "#2E7D32", response: g }
    blue:   { label: Blue,   value: "#1565C0", response: b }
    yellow: { label: Yellow, value: "#F9A825", response: y }

  design:
    method: full_factorial
    factors:
      word: [red, green, blue, yellow]
      ink: [red, green, blue, yellow]
    derive:
      congruency: "if(word == ink, 'congruent', 'incongruent')"

flow:
  - screen: instructions
    continue: { type: key, choices: [space] }

  - block: practice
    use: stroop_trial
    source: stimuli.design
    repetitions: $parameters.practice_repetitions
    order:
      method: constrained_shuffle
      seed: $parameters.random_seed
      constraints:
        - no_immediate_repeat: [word, ink, correct_response]
    feedback: trial_feedback
    evaluate: practice_gate

  - repeat: practice
    while: "$practice_gate.retry and $practice_gate.attempt < $parameters.maximum_practice_attempts"

  - screen: test_intro
    content: "Practice is complete. The test will now begin. You will no longer receive correctness feedback."
    continue: { type: key, choices: [space] }

  - block: test
    use: stroop_trial
    source: stimuli.design
    repetitions: $parameters.test_repetitions
    order:
      method: constrained_shuffle
      seed: $parameters.random_seed
      constraints:
        - no_immediate_repeat: [word, ink, correct_response]
        - maximum_run: { field: congruency, length: 3 }

  - screen: completion
    content: "The task is complete."
    continue: { type: key, choices: [space] }
```

```yaml trial stroop_trial
variables:
  correct_response: "$stimuli.colors[$row.ink].response"

timeline:
  - id: fixation
    show:
      type: text
      text: "+"
      style: { font_size: 48px, color: "#111111" }
    duration: $parameters.fixation_duration
    response: none

  - id: stimulus
    show:
      type: text
      text: "$stimuli.colors[$row.word].label"
      style:
        color: "$stimuli.colors[$row.ink].value"
        font_size: 64px
        font_weight: 700
    duration: $parameters.stimulus_duration
    response:
      type: key
      choices: [r, g, b, y]
      correct: "$response.value == $correct_response"
      timestamp_from: stimulus.actual_onset
      ends_display: true
      ends_trial: true
      timeout: $parameters.stimulus_duration

  - id: inter_trial_interval
    show: { type: blank }
    duration: $parameters.inter_trial_interval
    response: none

record:
  raw:
    - session_id
    - participant_id
    - phase
    - block_index
    - trial_index
    - word
    - ink
    - correct_response
    - response.value
    - response.correct
    - response.rt
    - response.timestamp
    - response.timed_out
    - stimulus.requested_onset
    - stimulus.actual_onset
    - stimulus.requested_duration
    - stimulus.actual_duration
    - document.visibility_changes
  calculated:
    congruency: "$row.congruency"
```

```yaml feedback trial_feedback
scope: practice
rules:
  - when: $response.timed_out
    show: { type: text, text: "Too slow", color: "#B71C1C" }
    duration: $parameters.feedback_duration
  - when: $response.correct
    show: { type: text, text: "Correct", color: "#1B5E20" }
    duration: $parameters.feedback_duration
  - otherwise:
    show: { type: text, text: "Incorrect", color: "#B71C1C" }
    duration: $parameters.feedback_duration
```

```yaml evaluate practice_gate
attempt: "$block.iteration"
accuracy: "mean($block.trials.response.correct)"
retry: "$accuracy < $parameters.minimum_practice_accuracy"
on_final_failure: continue_with_quality_flag
quality_flag: practice_criterion_not_met
```

```yaml score
population: "$trials where phase == 'test'"
exclusions:
  rt_valid: "not response.timed_out and response.rt >= 100ms"

outputs:
  trial_count: "count($population)"
  response_count: "count($population where not response.timed_out)"
  correct_count: "count($population where response.correct)"
  accuracy: "mean($population.response.correct)"
  omission_rate: "mean($population.response.timed_out)"
  mean_correct_rt: "mean($population where response.correct and rt_valid, response.rt)"
  median_correct_rt: "median($population where response.correct and rt_valid, response.rt)"
  congruent_accuracy: "mean($population where congruency == 'congruent', response.correct)"
  incongruent_accuracy: "mean($population where congruency == 'incongruent', response.correct)"
  congruent_median_correct_rt: "median($population where congruency == 'congruent' and response.correct and rt_valid, response.rt)"
  incongruent_median_correct_rt: "median($population where congruency == 'incongruent' and response.correct and rt_valid, response.rt)"
  interference_rt: "$incongruent_median_correct_rt - $congruent_median_correct_rt"
  interference_accuracy: "$congruent_accuracy - $incongruent_accuracy"

return:
  format: neutral-results/v0.1
  include: [metadata, parameters, trials, outputs, quality, provenance]
```
