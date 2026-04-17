<img width="1024" height="572" alt="image" src="https://github.com/user-attachments/assets/080a0d1c-8f69-43b5-bc00-b67b3e8612cb" />

# CortexPilot Executive Control Benchmark

### Project Name

CortexPilot Executive Control Benchmark

### Your Team

Ajay Bhargava Jashwanth Reddy

### Problem Statement

Modern language models can produce fluent text and solve many reasoning tasks, but those strengths do not always reflect robust executive control. In practical decision settings, models are often required to do more than generate plausible answers: they must follow constraints, resist intuitive but incorrect shortcuts, and adapt when rules change. These control processes are central to human executive function and are strongly tied to real-world reliability [Miyake et al., 2000; Diamond, 2013].

The core problem this benchmark addresses is that many existing evaluations emphasize final-answer accuracy without isolating how the model arrives there under cognitive pressure. A model may appear strong overall yet still fail in important ways, such as:

- producing plans that violate explicitly stated rules,
- selecting a common but wrong trap response,
- continuing to apply an outdated rule after a switch instruction.

These failure modes are not minor formatting issues; they indicate weakness in the model's internal control of reasoning steps. For safety-sensitive and workflow-heavy applications, this gap can lead to brittle behavior even when general benchmark scores are high.

To make this measurable, CortexPilot evaluates executive control as three distinct but related dimensions:

- Planning/Re-planning: Can the model build and update valid action sequences under constraints?
- Inhibitory Control: Can the model suppress a salient incorrect response and choose the correct one?
- Cognitive Flexibility: Can the model switch rules and maintain performance after the switch?

The benchmark also explicitly probes two additional executive-function behaviors within these same tasks:

- conflict resolution between multiple plausible actions under competing constraints,
- intermediate mental state tracking (working-memory-like maintenance) across multi-step prompts.

By separating these dimensions and reporting both composite and facet-level scores, the benchmark provides a clearer diagnostic profile than a single aggregate accuracy number. In short, the goal is to test whether the model can control reasoning, not just complete it [Diamond, 2013; Hendrycks et al., 2021].

### Task & benchmark construction

The benchmark is implemented as three separate Kaggle tasks in a single evaluation pipeline. Each task targets one executive-function component and enforces a structured JSON response so outputs can be reliably parsed and graded.

The end-to-end execution flow is:

1. Send item prompt to the model.
2. Clean output and parse into a strict schema (Pydantic).
3. Apply task-specific correctness checks.
4. Record per-item outcomes and aggregate into facet metrics.

Task design by benchmark component:

- Task 1 (`ExecPlan_Benchmark`): measures planning and re-planning under constraints.
  - Items require producing valid action orders, schedules, or resource-feasible sets.
  - Grading supports normalized equivalence (alternate valid sequences) to avoid brittle string-only scoring.
  - Several items force conflict resolution among plausible alternatives and require maintaining intermediate constraints across steps.
- Task 2 (`ExecInhibit_Benchmark`): measures inhibitory control under salient traps.
  - Items are designed so an intuitive answer is often incorrect.
  - Grading checks both final-answer correctness and explicit trap recognition, aligned with interference-control framing [Stroop, 1935].
- Task 3 (`ExecSwitch_Benchmark`): measures cognitive flexibility after rule updates.
  - Items separate pre-switch and post-switch phases.
  - Grading compares adherence to the currently active rule and captures switch-sensitive behavior [Rogers & Monsell, 1995].
  - Rule-update prompts require the model to retain prior context while correctly applying a new rule, testing working-memory-like control under interference.

Implementation choices made for robustness and reproducibility:

- Strict schema validation via Pydantic for all task outputs.
- JSON fence cleanup and retry logic to recover from malformed outputs.
- Task-return compatibility with Kaggle SDK plus notebook-side record buffers for deeper analytics.
- Repeated full-benchmark runs with mean/std reporting to evaluate stability, not only peak score.

### Dataset

The benchmark uses three curated internal item banks:

- Planning bank: 12 items
- Inhibition bank: 13 items
- Switching bank: 12 items

Design characteristics:

- Planning items include constraint ordering, budget/resource constraints, and re-planning after rule changes.
- Inhibition items are trap-heavy (salient heuristic lures with non-intuitive correct answers), consistent with inhibitory-control stress tests [Stroop, 1935].
- Switching items explicitly separate pre-switch and post-switch phases to capture switch cost [Rogers & Monsell, 1995].

### Technical details

Core runtime and scoring setup:

- Platform: Kaggle Benchmarks SDK (`kaggle-benchmarks`)
- Prompting: strict JSON-only response requirement
- Validation: Pydantic schema parse + assertion-based grading
- Composite metric:

`ExecScore = (Planning Score + Inhibition Score + Post-switch Accuracy) / 3`

- Additional diagnostics:
  - Trap Recognition Rate
  - Pre-switch Accuracy
  - Post-switch Accuracy
  - Switch Cost

Robustness upgrades applied during implementation:

- Planning: normalization + equivalence matching + retry on invalid/misaligned outputs
- Inhibition: two prompt variants (A/B) tested with repeated evaluation
- Stability-first selection: model/prompt configuration chosen using highest mean ExecScore with low std

### Results, insights, and conclusions

Latest repeated full-run result (5 repeats) for `google/gemini-3-flash-preview`:

- Variant A (`inhib=A`):
  - Mean ExecScore: 87.26%
  - Std ExecScore: 6.76%
  - High volatility from inhibition collapses in some runs
- Variant B (`inhib=B`):
  - Mean ExecScore: 92.39%
  - Std ExecScore: 1.02%
  - Clearly better and substantially more stable

Final recommended configuration:

- `google/gemini-3-flash-preview - inhib=B`
- Primary selection basis: highest mean ExecScore with lowest observed run variance.

Facet-level behavior:

- Switching is consistently saturated at 100% with near-zero switch cost.
- Planning is stable but capped at about 83.33%, making it the current headroom bottleneck.
- Inhibition is the main variance driver; prompt variant B materially improves robustness.

Image placeholders (paste your images below these lines):

<img width="862" height="714" alt="exec_radar (1)" src="https://github.com/user-attachments/assets/0c21f46a-446b-403c-a498-302c7615f63d" />


<img width="1021" height="733" alt="switch_cost" src="https://github.com/user-attachments/assets/9ada1a60-84e6-4f18-8e5c-ada39d2a4e94" />


<img width="1175" height="733" alt="exec_scores" src="https://github.com/user-attachments/assets/8f0452c0-2e00-465f-9f65-885de380d0ad" />


Conclusion:

The benchmark is functioning as intended and produces meaningful executive-control diagnostics. For final submission settings, use the inhibition-B configuration and prioritize next iteration work on planning-task ceiling improvements. This complements broad capability benchmarks by adding targeted control-process measurement beyond general knowledge and reasoning scores [Hendrycks et al., 2021].

### Organizational affiliations

Kaggle, Google

### References & citations

Miyake, A., Friedman, N. P., Emerson, M. J., Witzki, A. H., Howerter, A., & Wager, T. D. (2000). The unity and diversity of executive functions and their contributions to complex "Frontal Lobe" tasks: A latent variable analysis. Cognitive Psychology, 41(1), 49-100. https://doi.org/10.1006/cogp.1999.0734

Diamond, A. (2013). Executive functions. Annual Review of Psychology, 64, 135-168. https://doi.org/10.1146/annurev-psych-113011-143750

Rogers, R. D., & Monsell, S. (1995). Costs of a predictable switch between simple cognitive tasks. Journal of Experimental Psychology: General, 124(2), 207-231. https://doi.org/10.1037/0096-3445.124.2.207

Stroop, J. R. (1935). Studies of interference in serial verbal reactions. Journal of Experimental Psychology, 18(6), 643-662. https://doi.org/10.1037/h0054651

Hendrycks, D., Burns, C., Basart, S., Zou, A., Mazeika, M., Song, D., & Steinhardt, J. (2021). Measuring Massive Multitask Language Understanding. arXiv. https://arxiv.org/abs/2009.03300

