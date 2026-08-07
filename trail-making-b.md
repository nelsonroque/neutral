---
schema: neutral-spec/v0.1
id: trail-making-b-digital
title: Trail Making Test, Part B — Digital Adaptation
version: 0.1.0
description: Connect alternating numbers and letters in ascending sequence.
language: en-US
tags: [executive-function, set-shifting, visuomotor, completion-time]
notices:
  - This is an engine-neutral digital adaptation, not a claim of equivalence to any specific normed or proprietary form.
  - Layout, input device, screen size, and error handling must be held constant before normative interpretation.

parameters:
  practice_time_limit:
    type: duration
    default: 60s
  test_time_limit:
    type: duration
    default: 300s
  node_diameter:
    type: viewport_fraction
    default: 0.065
  hit_slop:
    type: viewport_fraction
    default: 0.012
  path_width:
    type: viewport_fraction
    default: 0.006
  incorrect_feedback_duration:
    type: duration
    default: 350ms
  require_fullscreen:
    type: boolean
    default: true

requirements:
  input: [pointer, touch]
  orientation: landscape
  minimum_viewport: { width: 900px, height: 600px }
  fullscreen: required
---

# Instructions

On the screen are circles containing numbers and letters. Connect them in
alternating order:

**1 → A → 2 → B → 3 → C**, and so on.

Begin at **1** and continue until the final number. Work as quickly as you can
without making mistakes. If you select the wrong circle, it will be marked as
an error. Correct the mistake and continue; the timer will keep running.

```yaml task
coordinate_system:
  type: normalized
  origin: top_left
  x_range: [0, 1]
  y_range: [0, 1]
  safe_area: { left: 0.05, right: 0.05, top: 0.07, bottom: 0.07 }

stimuli:
  practice_nodes:
    - { id: p1, label: "1", sequence: 1, x: 0.12, y: 0.72 }
    - { id: pA, label: "A", sequence: 2, x: 0.32, y: 0.25 }
    - { id: p2, label: "2", sequence: 3, x: 0.53, y: 0.68 }
    - { id: pB, label: "B", sequence: 4, x: 0.73, y: 0.24 }
    - { id: p3, label: "3", sequence: 5, x: 0.88, y: 0.70 }

  test_nodes:
    - { id: n1,  label: "1",  sequence: 1,  x: 0.08, y: 0.78 }
    - { id: A,   label: "A",  sequence: 2,  x: 0.18, y: 0.19 }
    - { id: n2,  label: "2",  sequence: 3,  x: 0.31, y: 0.69 }
    - { id: B,   label: "B",  sequence: 4,  x: 0.47, y: 0.13 }
    - { id: n3,  label: "3",  sequence: 5,  x: 0.64, y: 0.66 }
    - { id: C,   label: "C",  sequence: 6,  x: 0.86, y: 0.22 }
    - { id: n4,  label: "4",  sequence: 7,  x: 0.76, y: 0.83 }
    - { id: D,   label: "D",  sequence: 8,  x: 0.55, y: 0.43 }
    - { id: n5,  label: "5",  sequence: 9,  x: 0.91, y: 0.55 }
    - { id: E,   label: "E",  sequence: 10, x: 0.68, y: 0.08 }
    - { id: n6,  label: "6",  sequence: 11, x: 0.39, y: 0.34 }
    - { id: F,   label: "F",  sequence: 12, x: 0.13, y: 0.48 }
    - { id: n7,  label: "7",  sequence: 13, x: 0.36, y: 0.88 }
    - { id: G,   label: "G",  sequence: 14, x: 0.59, y: 0.91 }
    - { id: n8,  label: "8",  sequence: 15, x: 0.24, y: 0.56 }
    - { id: H,   label: "H",  sequence: 16, x: 0.82, y: 0.41 }
    - { id: n9,  label: "9",  sequence: 17, x: 0.49, y: 0.76 }
    - { id: I,   label: "I",  sequence: 18, x: 0.07, y: 0.30 }
    - { id: n10, label: "10", sequence: 19, x: 0.72, y: 0.56 }
    - { id: J,   label: "J",  sequence: 20, x: 0.28, y: 0.10 }
    - { id: n11, label: "11", sequence: 21, x: 0.89, y: 0.73 }
    - { id: K,   label: "K",  sequence: 22, x: 0.43, y: 0.57 }
    - { id: n12, label: "12", sequence: 23, x: 0.17, y: 0.88 }
    - { id: L,   label: "L",  sequence: 24, x: 0.62, y: 0.26 }
    - { id: n13, label: "13", sequence: 25, x: 0.92, y: 0.09 }

flow:
  - screen: instructions
    continue: { type: button, label: Begin practice }

  - scene: practice
    use: trail_scene
    nodes: $stimuli.practice_nodes
    time_limit: $parameters.practice_time_limit
    feedback: corrective_feedback

  - screen: practice_complete
    content: "Practice is complete. Remember to alternate numbers and letters."
    continue: { type: button, label: Begin test }

  - scene: test
    use: trail_scene
    nodes: $stimuli.test_nodes
    time_limit: $parameters.test_time_limit
    feedback: corrective_feedback

  - screen: completion
    content: "The task is complete."
    continue: { type: button, label: Continue }
```

```yaml scene trail_scene
render:
  background: "#FFFFFF"
  nodes:
    source: $scene.nodes
    shape: circle
    diameter: $parameters.node_diameter
    fill: "#FFFFFF"
    stroke: "#111111"
    stroke_width: 2px
    label:
      color: "#111111"
      font_size: "clamp(18px, 2.2vw, 32px)"
      font_weight: 600
    hit_slop: $parameters.hit_slop
  path:
    mode: connect_node_centers
    retain_correct_segments: true
    retain_incorrect_segments: false
    color: "#1565C0"
    width: $parameters.path_width
  start_indicator:
    node_sequence: 1
    label: Begin
  end_indicator:
    node_sequence: "max($scene.nodes.sequence)"
    label: End

interaction:
  type: ordered_node_selection
  pointer_events: [pointerdown, pointermove, pointerup]
  activation: pointerdown
  first_target: 1
  expected_next: "$state.last_correct_sequence + 1"
  correctness: "$selected.sequence == $expected_next"
  on_correct:
    - append_path_segment: true
    - mark_node: completed
    - advance_sequence: true
  on_incorrect:
    - record_error: true
    - keep_expected_target: true
    - continue_timer: true
  complete_when: "$state.last_correct_sequence == max($scene.nodes.sequence)"
  timeout: $scene.time_limit
  timer_starts: scene.actual_onset
  timer_stops: final_correct_selection.timestamp

record:
  events:
    pointer_sample:
      throttle: animation_frame
      fields: [timestamp, x, y, pressure, pointer_type]
    node_selection:
      fields:
        - timestamp
        - selected.id
        - selected.label
        - selected.sequence
        - expected.sequence
        - correct
        - elapsed_time
        - x
        - y
    error:
      fields:
        - timestamp
        - selected.id
        - expected.id
        - elapsed_time
    lifecycle:
      fields: [scene.actual_onset, document.visibility_changes, viewport, device_pixel_ratio]
  calculated:
    inter_node_latency: "$selection.timestamp - $previous_correct_selection.timestamp"
    path_distance: "sum(distance($pointer_samples))"
    ideal_path_distance: "sum(distance(center($correct_node_pairs)))"
    path_efficiency: "$ideal_path_distance / $path_distance"
```

```yaml feedback corrective_feedback
rules:
  - when: "$selection.correct == false"
    show:
      type: node_highlight
      target: $selection.selected.id
      color: "#B71C1C"
      style: pulse
    duration: $parameters.incorrect_feedback_duration
    timer: continue
    response_lock: false
```

```yaml score
population: "$scenes where phase == 'test'"
outputs:
  completed: "$test.complete"
  timed_out: "$test.timed_out"
  completion_time: "if($completed, $test.final_correct_selection.timestamp - $test.actual_onset, null)"
  elapsed_time_at_stop: "$test.stop_timestamp - $test.actual_onset"
  error_count: "count($test.events where type == 'error')"
  corrected_error_count: "count(distinct($test.errors.expected.id subsequently selected correctly))"
  correct_selection_count: "count($test.node_selections where correct)"
  total_selection_count: "count($test.node_selections)"
  mean_inter_node_latency: "mean($test.correct_selections.inter_node_latency)"
  path_distance: "$test.path_distance"
  ideal_path_distance: "$test.ideal_path_distance"
  path_efficiency: "$test.path_efficiency"

quality:
  flags:
    - when: "$test.document.visibility_changes > 0"
      code: focus_changed
    - when: "$test.viewport.width < 900 or $test.viewport.height < 600"
      code: viewport_below_minimum
    - when: "$test.timed_out"
      code: time_limit_reached

return:
  format: neurotask-results/v0.1
  include: [metadata, parameters, events, outputs, quality, provenance]
```
