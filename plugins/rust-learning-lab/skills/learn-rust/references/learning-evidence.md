# Programming learning research and its scope

English | [日本語](learning-evidence_ja.md)

Sources checked: 2026-09-26. Read when explaining evidence for learning methods or their effects.
Separate reported findings from the applications adopted by this skill. Classroom or curriculum comparisons do not
establish universal effects of individual techniques or long-term Rust mastery. Where only an abstract or summary
was checked, stay within it; read the paper before discussing detailed effect sizes or causal claims.

## Reading and understanding code

### Explicit tracing

[Xie, Nelson & Ko, 2018](https://faculty.washington.edu/ajko/papers/Xie2018TracingStrategies.pdf) compared instruction
in step-by-step Java tracing with external state tables among 24 university novices. Tracing performance improved.
This small, short-task study does not establish general implementation ability. This skill uses tables when state
is difficult to follow, locating the point where predictions diverge.

### Learning through prediction

[Prediction versus production for teaching computer programming, 2024](https://www.sciencedirect.com/science/article/pii/S0959475223001408)
randomly assigned 121 university students without programming experience to predicting R output or writing code after
explanation. The prediction group improved on learning assessments and other measures. Introductory-task results do not
imply that writing practice is unnecessary. This skill treats prediction as an interactive-practice option, not a barrier
to answering ordinary questions.

### From reading to modification and creation

PRIMM in [Sentance, Waite & Kallia, 2019](https://eprints.gla.ac.uk/229013/) combines prediction, running, investigation,
modification, and making. A comparison involving 493 learners aged 11–14 across 13 schools over 8–12 weeks reported
better post-test performance. This evaluates a classroom approach, not an individual adult learner or each step in isolation.
This skill uses it as an option for connecting reading to changes and independent creation, not a compulsory five-step routine.

### Explaining Rust ownership

[Crichton, Gray & Krishnamurthi, 2023](https://arxiv.org/abs/2309.04134) incorporated a permissions model and
visualizations into learning material to connect runtime state with static constraints through permitted reading,
writing, and ownership-related operations. An initial deployment evaluation with 342 readers reported improved
ownership assessment performance; it did not measure long-term development ability in general. This skill connects
values and references to permitted operations at each point, without requiring a visualization tool just for teaching.

## Supporting writing and debugging

### Worked examples with subgoal labels

[Margulieux, Morrison & Decker, 2020](https://link.springer.com/article/10.1186/s40594-020-00222-7) evaluated
subgoal-labeled worked examples over a university introductory programming course. Quiz performance and failure or
withdrawal indicators improved, but average exam performance did not improve significantly. This skill explains both
what a line does and what a group achieves, then selects practice applying that group to another problem.

### Reordering as writing support

[Hou, Ericson & Wang, 2023](https://arxiv.org/abs/2311.18115) randomly assigned 89 university students to conditions
examining Parsons problems as support for code writing. Practice performance and efficiency improved for participants
with lower self-efficacy; prior knowledge also affected how accessible the support was. This does not establish superiority
for everyone or long-term independent writing ability. This skill uses reordering selectively, followed by a small task without choices.

### Explicit debugging instruction

[Decoding Debugging Instruction, 2024](https://doi.org/10.1145/3690652) reviewed 43 intervention studies from 2010–2022.
Results for accuracy and learning were promising, but learners did not consistently adopt the systematic strategies they
were taught, and replications were limited. This skill makes observation, hypothesis, and checking visible and looks for
actual use during a repair. Explaining a procedure alone does not establish debugging ability.

### Limits of automated feedback

[Messer et al., 2024 (author manuscript)](https://arxiv.org/abs/2306.11722) reviewed 121 studies from 2017–2021.
Many tools emphasized correctness, with feedback often limited to tests and differences from expected results.
This was not a comparison establishing the superiority of a particular hint format or AI. This skill provides a next
operation toward the cause as well as the result, separating task success from learner understanding.

## AI-assisted work and independent understanding

### Novices using code generation

[Kazemitabaar et al., 2023](https://arxiv.org/abs/2302.07427) compared Python authoring tasks followed by manual
modification tasks among 69 novices aged 10–17. Generation assistance improved authoring performance without a decrease
in modification performance. The overall group difference on a one-week follow-up was not statistically significant.
The setting, ages, materials, and model were specific; do not generalize this into a claim that generation improves
long-term learning. This skill does not ban AI categorically; it separates getting working code from changing and
explaining it under other conditions.

### Assistance while learning a new library

[Shen & Tamkin, 2026](https://www.anthropic.com/research/AI-assistance-coding-skills) reported that developers randomly
assigned to AI assistance while learning an unfamiliar Python library scored lower on an immediate understanding test;
the time difference was not significant. Usage patterns such as conceptual questions were associated with higher scores,
but those patterns were not randomized and their causal effects were not established. The small, short-term study does
not directly establish effects on Rust, every current product, or long-term skill decline. This skill does not infer
understanding from code completed or time spent reading AI output; it checks unaided prediction, explanation, and repair separately.

These studies differ in participants, tasks, and assessments. Do not select only the convenient result or infer learning
effects from AI use alone. See [Engineering practice](engineering-practice.md) for applications to exercises.

## Supporting evidence on retention

General memory research informs retrieval and spacing, separately from direct programming-skill evidence.
Participants, limitations, and applications are described in [Learning design](learning-design.md#retrieval-and-spaced-revision).
Assess reading, changing, creating, and debugging on another task with reduced support.
