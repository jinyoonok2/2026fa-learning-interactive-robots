---
layout: distill
title: "Continual Learning, Multi-task Learning"
description: "Lifelong robot learning, preventing catastrophic forgetting, and positive transfer across diverse robotic manipulation tasks."
date: 2026-11-18
future: true
htmlwidgets: true
ready: false

# Reporter team authors
authors:
  - name: "Jinyoon Kim"
    affiliations:
      - name: "University of Virginia"
  - name: "Austin Song"
    affiliations:
      - name: "University of Virginia"
  - name: "Boyang Yan"
    affiliations:
      - name: "University of Virginia"
  - name: "Sai Tarra"
    affiliations:
      - name: "University of Virginia"
  - name: "Harris Ilyas"
    affiliations:
      - name: "University of Virginia"

bibliography: 2026-11-18-continual-learning-multi-task-learning.bib

toc:
  - name: "Introduction"
  - name: "LIBERO: Benchmarking Knowledge Transfer for Lifelong Robot Learning"
    subsections:
      - name: "LIBERO: Motivation and Problem"
      - name: "LIBERO: Method and Formulation"
      - name: "LIBERO: Experiments and Results"
      - name: "LIBERO: Limitations and Takeaways"
  - name: "PoCo: Policy Composition from and for Heterogeneous Robot Learning"
    subsections:
      - name: "PoCo: Motivation and Problem"
      - name: "PoCo: Method and Formulation"
      - name: "PoCo: Experiments and Results"
      - name: "PoCo: Limitations and Takeaways"
  - name: "SRSA: Skill Retrieval and Adaptation for Robotic Assembly Tasks"
    subsections:
      - name: "Motivation and Problem"
      - name: "Method and Formulation"
      - name: "Experiments and Results"
      - name: "Limitations and Takeaways"
  - name: "Discussion"
---

<!--
Report draft awaiting team review.
Keep ready: false until everyone has reviewed the report.
Add verified BibTeX entries to assets/bibliography/2026-11-18-continual-learning-multi-task-learning.bib.
Store figures in assets/img/2026-11-18-continual-learning-multi-task-learning/.
Store interactive HTML in assets/html/2026-11-18-continual-learning-multi-task-learning/.
-->

## Introduction

### Background and Motivation

Robots encounter new objects and tasks after deployment. Learning to open a drawer might help a robot learn to open a cabinet, but updating the model for the new task can also reduce its performance on earlier ones. This loss of previously learned behavior is called catastrophic forgetting.

### Multi-task Learning

Multi-task learning (MTL) trains on several task datasets jointly so that related tasks can share representations and behaviors. Robotic datasets may differ in their sensors, embodiments, and simulation or real-world conditions. PoCo asks whether policies trained separately on these sources can share information at inference time without joint retraining. <d-cite key="wang2024poco"></d-cite>

### Continual Learning and Catastrophic Forgetting

Continual learning (CL), or lifelong learning, trains on tasks in sequence with limited access to earlier data. It must balance plasticity, the ability to learn new skills, with stability, the ability to retain old ones. Keeping all past data for joint retraining would increase storage and training costs as tasks accumulate.

| Characteristics | Multi-task Learning | Continual Learning |
| :--- | :--- | :--- |
| Task availability | Simultaneous | Sequential |
| Training data access | Multiple task datasets | Current task and potentially limited historical data |
| Training process | Joint training | Incremental learning |
| Primary objective | Knowledge sharing across tasks | Adaptation and retention |
| Major challenges | Task interference and data heterogeneity | Catastrophic forgetting and limited resources |

### Adaptation, Retention, and Knowledge Transfer

This lecture examines what a robot can reuse when learning a new task: training examples and parameters, policy predictions, or specialist skills.

| Paper | Knowledge Reuse Mechanism | Primary Research Objective |
| :--- | :--- | :--- |
| LIBERO | Benchmark methods that reuse demonstrations or protect parameters | Measure transfer and forgetting across tasks |
| PoCo <d-cite key="wang2024poco"></d-cite> | Compose independently trained policy predictions | Use heterogeneous data without joint retraining |
| SRSA <d-cite key="guo2025srsa"></d-cite> | Retrieve and adapt specialist policies | Learn new assembly tasks from existing skills |

## LIBERO: Benchmarking Knowledge Transfer for Lifelong Robot Learning

LIBERO is a simulation benchmark for lifelong robot learning. It provides 130 manipulation tasks, human demonstrations, and baseline algorithms and policy architectures for studying transfer and forgetting. <d-cite key="liu2023libero"></d-cite>

Figures are from the paper and supplement; captions are our summaries.

### LIBERO: Motivation and Problem

After learning new tasks, a robot may fail to retrieve juice because it forgets the object's location or the opening and grasping motions. LIBERO separates these sources of forgetting into declarative and procedural knowledge.

| Type of knowledge | What the robot needs to know | Manipulation example |
|---|---|---|
| Declarative knowledge | Object concepts, locations, and spatial relationships. | Identify the juice and locate it inside the refrigerator. |
| Procedural knowledge | Motions and behaviors needed to accomplish a goal. | Open the door, grasp the juice, and retrieve it. |

To isolate what transfers, LIBERO varies objects, spatial relationships, and task goals. It examines five questions:

1. Knowledge transfer: Which task changes are hardest to learn and retain?
2. Architecture: How should a policy combine images, language, robot state, and observation history?
3. Learning algorithm: How do replay, regularization, and freezing affect learning?
4. Task order: Do results change when tasks arrive in a different order?
5. Pretraining: Does earlier training on a larger task set help?

{% include figure.liquid
   path="assets/img/2026-11-18-continual-learning-multi-task-learning/libero-fig-01-benchmark-overview.png"
   class="img-fluid rounded z-depth-1"
   alt="LIBERO overview showing four task suites and five research topics for lifelong robot learning."
   caption="LIBERO overview: four task suites and five research topics. The source figure reverses the Spatial/Object descriptions; our task-suite table follows Section 4.2. Source: Liu et al. (2023), main paper, Figure 1."
   zoomable=true
   avoid_scaling=true
%}

<!-- Source-artwork note: the two left descriptive subtitles in the original Figure 1 appear exchanged relative to the suite definitions in Section 4.2. Our caption follows those textual definitions; the extracted artwork is unchanged. -->

### LIBERO: Method and Formulation

#### Sequential learning setup

The robot learns $$K$$ tasks in sequence. Each task $$\mathcal T_k=(\mu_0^k,g_k)$$ specifies a starting-state distribution $$\mu_0^k$$ and a success predicate $$g_k$$. Tasks share state and action spaces and environment dynamics.

For task $$k$$, the learner receives demonstrations $$D_k$$. Earlier datasets are not fully available, though replay can retain selected examples. After each training stage, evaluation revisits earlier tasks to measure retention.

#### Generating tasks with explicit goals

The pipeline uses the Robosuite simulator:

1. Generate instructions from Ego4D activity templates and available simulated objects.
2. Specify the scene layout, object placements, and initial states.
3. Define goal predicates in a PDDL-style task file. A drawer task, for example, succeeds when the drawer is open and the bowl is inside.

The predicates define success; the learned policy produces actions. Ego4D supplies language templates, while robot demonstrations are collected separately.

{% include figure.liquid
   path="assets/img/2026-11-18-continual-learning-multi-task-learning/libero-fig-02-task-generation.png"
   class="img-fluid rounded z-depth-1"
   alt="Task-generation pipeline connecting human-activity templates to language instructions, scene specifications, initial states, and goal predicates."
   caption="Task generation: activity templates become language instructions and simulated tasks with defined starting states and goals. Source: Liu et al. (2023), main paper, Figure 2."
   zoomable=true
   avoid_scaling=true
%}

#### Four task suites

The paper uses four fixed suites for reproducible comparisons.

| Task suite | Tasks | Main variation | What it tests |
|---|---:|---|---|
| LIBERO-Spatial | 10 | Locations and relations among the same objects. | Distinguish identical bowls by their spatial relation. |
| LIBERO-Object | 10 | Target objects in pick-and-place tasks. | Recognize and manipulate different object types. |
| LIBERO-Goal | 10 | Goals with fixed objects and layouts. | Learn different motions and behaviors in one scene. |
| LIBERO-100 | 100 | Objects, layouts, goals, and interactions. | Transfer mixed knowledge across diverse tasks. |

LIBERO-100 splits into 90 shorter tasks for pretraining (LIBERO-90) and 10 longer tasks for lifelong-learning evaluation (LIBERO-Long). Long horizon refers to dependent actions within an episode; lifelong learning refers to sequential training across tasks. <d-cite key="liu2023libero"></d-cite>

#### Policy architectures: from observations to actions

All three policies use external-camera and wrist-camera images, joint and gripper state, and BERT instruction embeddings. They differ in their visual encoder and temporal backbone; each has a Gaussian mixture model (GMM) action head. <d-cite key="liu2023libero"></d-cite>

| Policy | Visual encoder | Temporal backbone | Language integration |
|---|---|---|---|
| ResNet-RNN | ResNet | LSTM | FiLM-conditioned visual features and a language input to the LSTM. |
| ResNet-T | ResNet | Transformer decoder | FiLM-conditioned visual features and a language token in the temporal transformer. |
| ViT-T | Vision Transformer | Transformer decoder | Language tokens in both visual and temporal transformers. |

The temporal backbone summarizes the episode's observation history. The GMM head predicts a distribution over continuous actions, from which the robot samples an action to execute.

{% include figure.liquid
   path="assets/img/2026-11-18-continual-learning-multi-task-learning/libero-supp-fig-05-policy-architectures.png"
   class="img-fluid rounded z-depth-1"
   alt="Three policy architectures combining camera images, language embeddings, and robot joint and gripper information with recurrent or transformer temporal processing."
   caption="Three policies tested in LIBERO, left to right: ResNet-RNN, ResNet-T, and ViT-T. Each combines images, language, and robot state to predict actions. Source: Liu et al. (2023), supplement, Figure 5."
   zoomable=true
   avoid_scaling=true
%}

ResNet uses language-conditioned FiLM to rescale and shift visual features, followed by spatial softmax and a linear projection. ViT uses self-attention over image-patch and language tokens. Both representations feed the temporal backbone.

{% include figure.liquid
   path="assets/img/2026-11-18-continual-learning-multi-task-learning/libero-supp-fig-06-image-encoders.png"
   class="img-fluid rounded z-depth-1"
   alt="ResNet and vision-transformer image encoders showing language-conditioned residual features and attention over image and language tokens."
   caption="Encoder details: ResNet and ViT turn images and language into features used by the policies above. Source: Liu et al. (2023), supplement, Figure 6."
   zoomable=true
   avoid_scaling=true
%}

#### Learning algorithms: preserving earlier skills

LIBERO pairs these architectures with three continual-learning algorithms and two reference methods.

| Method | How training works | Resource or constraint |
|---|---|---|
| Sequential fine-tuning (SEQL) | Train on the current task's demonstrations only. | No explicit retention mechanism. |
| Experience Replay (ER) | Mix stored old-task examples with current data. | Requires replay memory. |
| Elastic Weight Consolidation (EWC) | Penalize changes using Fisher-based parameter importance. | Constrains later updates. |
| PackNet | Train, prune, fine-tune, and freeze task subnetworks. | Leaves less capacity for later tasks. |
| Multi-task learning (MTL) | Train jointly on all task datasets. | Requires access to all datasets. |

PackNet's masks protect earlier subnetworks while allowing later tasks to use their frozen weights. MTL has full data access, so it serves as a joint-training reference rather than a guaranteed upper bound. <d-cite key="liu2023libero,mallya2018packnet"></d-cite>

#### Demonstrations and training procedure

Policies are trained by behavioral cloning on 50 human demonstrations per task, collected through teleoperation with a 3Dconnexion SpaceMouse. A per-step negative log-likelihood loss is:

$$
\mathcal L_{\mathrm{BC}}(\theta;D_k)
=-\mathbb E_{(o_{\leq t},a_t)\sim D_k}
\left[\log\pi_\theta(a_t\mid o_{\leq t};\mathcal T_k)\right].
$$

Source: Adapted from LIBERO, Section 2.2, Equation (2). This equation is not shown in the W13 presentation. <d-cite key="liu2023libero"></d-cite>

Here, $$\theta$$ denotes policy parameters, $$o_{\leq t}$$ is observation history, and $$a_t$$ is the demonstrated action. Minimizing the loss increases the probability assigned to demonstrated actions.

The sequential evaluation procedure is:

1. Train on the current task for 50 epochs using the selected algorithm.
2. Evaluate every five epochs with 20 rollouts, each capped at 600 steps.
3. Keep the earliest checkpoint with the best current-task success rate, then retest earlier tasks with 20 rollouts each.
4. Continue through the task sequence and repeat with three random seeds.

PackNet adds 50 epochs of fine-tuning after pruning; the supplement reports metrics before this stage. MTL trains jointly. Separate studies vary task order and test LIBERO-90 pretraining before learning LIBERO-Long. <d-cite key="liu2023libero"></d-cite>

#### Evaluation equations from the W13 presentation

The slides use three success-rate metrics: forward transfer (FWT) for new-task learning, negative backward transfer (NBT) for forgetting, and area under the curve (AUC) for their combination. <d-cite key="liu2023libero"></d-cite>

The equations use the following notation:

| Symbol | Meaning |
|---|---|
| $$K$$ | Number of tasks in the sequence. |
| $$k$$ | Task being measured. |
| $$\tau$$ | A training stage after task $$k$$. |
| $$e\in\mathcal E=\{0,5,10,\ldots,50\}$$ | The 11 evaluation checkpoints. |
| $$c_{i,j,e}$$ | Success rate on task $$j$$ after $$e$$ training epochs on task $$i$$. |
| $$c_{i,j}$$ | Success rate on task $$j$$ at task $$i$$'s selected best checkpoint. |

Thus, $$c_{k,k}$$ is task $$k$$'s best acquisition success rate, and $$c_{\tau,k}$$ is its later performance. For these metrics, the learning curve is held constant after the earliest best checkpoint.

{% include figure.liquid
   path="assets/img/2026-11-18-continual-learning-multi-task-learning/libero-fig-03-evaluation-metrics.png"
   class="img-fluid rounded z-depth-1"
   alt="Schematic success-rate curve for one task during acquisition and after learning later tasks, with areas illustrating FWT, NBT, and AUC."
   caption="Schematic of LIBERO's evaluation metrics: FWT (new-task learning), NBT (forgetting), and AUC (learning and retention). Source: Liu et al. (2023), main paper, Figure 3."
   zoomable=true
   avoid_scaling=true
%}

##### Forward transfer: learning a new task

$$
\begin{aligned}
\mathrm{FWT}
&=\frac{1}{K}\sum_{k=1}^{K}\mathrm{FWT}_k,\\
\mathrm{FWT}_k
&=\frac{1}{11}\sum_{e\in\mathcal E}c_{k,k,e}.
\end{aligned}
$$

Source: W13 presentation, "LIBERO - Evaluation Metrics," slide 29 (PDF page 38); LIBERO, Section 5.1, Equation (3).

FWT averages success across the 11 learning checkpoints, then across tasks. Higher values indicate faster or stronger learning. This definition does not subtract a from-scratch baseline, so it does not isolate the benefit of prior learning.

##### Negative backward transfer: forgetting an earlier task

$$
\begin{aligned}
\mathrm{NBT}
&=\frac{1}{K}\sum_{k=1}^{K}\mathrm{NBT}_k,\\
\mathrm{NBT}_k
&=\frac{1}{K-k}\sum_{\tau=k+1}^{K}
\left(c_{k,k}-c_{\tau,k}\right).
\end{aligned}
$$

Source: W13 presentation, "LIBERO - Evaluation Metrics," slide 29 (PDF page 38); LIBERO, Section 5.1, Equation (3).

NBT averages the drop from task $$k$$'s best learned performance to its performance after later tasks. Lower values indicate better retention; negative values indicate improvement. A drop from 0.8 to 0.5 contributes 0.3.

*Notation note:* The inner expression applies to $$k<K$$. For the final task, $$K-k=0$$; computing the printed average requires a final-task convention.

##### AUC: combining learning and retention

$$
\begin{aligned}
\mathrm{AUC}
&=\frac{1}{K}\sum_{k=1}^{K}\mathrm{AUC}_k,\\
\mathrm{AUC}_k
&=\frac{1}{K-k+1}
\left(\mathrm{FWT}_k+\sum_{\tau=k+1}^{K}c_{\tau,k}\right).
\end{aligned}
$$

Source: W13 presentation, "LIBERO - Evaluation Metrics," slide 29 (PDF page 38); LIBERO, Section 5.1, Equation (3).

AUC averages a task's acquisition score, $$\mathrm{FWT}_k$$, with its success rates after later tasks, then averages across tasks. The denominator counts one acquisition stage and $$K-k$$ later stages. Higher AUC is better.

### LIBERO: Experiments and Results

#### Comparing continual-learning algorithms

The first experiment compares how effectively different continual-learning algorithms acquire new tasks while retaining previously learned skills. The presentation highlights results from LIBERO-Long and LIBERO-Spatial, which represent two different types of knowledge transfer. LIBERO-Long requires transferring both declarative and procedural knowledge across diverse, long-horizon tasks, while LIBERO-Spatial primarily evaluates transfer across different spatial relationships.

The following table summarizes the experimental results presented in the lecture. Results are reported as the mean and standard error across three random seeds. <d-cite key="liu2023libero"></d-cite>

| Method | Long FWT ↑ | Long NBT ↓ | Long AUC ↑ | Spatial FWT ↑ | Spatial NBT ↓ | Spatial AUC ↑ |
|---|---:|---:|---:|---:|---:|---:|
| SEQL | **0.54 ± 0.01** | 0.63 ± 0.01 | 0.15 ± 0.00 | **0.72 ± 0.01** | 0.81 ± 0.01 | 0.20 ± 0.01 |
| ER | 0.48 ± 0.02 | 0.32 ± 0.04 | **0.32 ± 0.01** | 0.65 ± 0.03 | 0.27 ± 0.03 | 0.56 ± 0.01 |
| EWC | 0.13 ± 0.02 | 0.22 ± 0.03 | 0.02 ± 0.00 | 0.23 ± 0.01 | 0.33 ± 0.01 | 0.06 ± 0.01 |
| PackNet | 0.22 ± 0.01 | **0.08 ± 0.01** | 0.25 ± 0.00 | 0.55 ± 0.01 | **0.07 ± 0.02** | **0.63 ± 0.00** |
| MTL | — | — | 0.48 ± 0.01 | — | — | 0.83 ± 0.00 |

*Bold values indicate the best performance among sequential learning methods for each metric. MTL is included as a joint-training reference.*

Sequential fine-tuning achieves the highest FWT on both task suites, suggesting that a policy without an explicit retention mechanism can acquire new tasks relatively effectively. However, it also experiences the most substantial forgetting. On LIBERO-Long, SEQL achieves an FWT of 0.54 but an NBT of 0.63, resulting in an AUC of only 0.15. Similarly, on LIBERO-Spatial, its NBT reaches 0.81.

These results demonstrate why evaluating only the success rate of the most recently learned task is insufficient. A robot may appear to improve as it encounters new tasks while simultaneously losing its ability to perform earlier ones.

Experience Replay and PackNet demonstrate different approaches to the stability–plasticity dilemma:

- **Experience Replay** achieves the highest AUC among sequential methods on LIBERO-Long (0.32). Rehearsing earlier demonstrations allows the policy to retain some previously learned behaviors while continuing to update its parameters.
- **PackNet** achieves the lowest NBT on both task suites, indicating stronger retention of previously learned tasks. However, its lower FWT on LIBERO-Long suggests that protecting earlier parameters can restrict the acquisition of new skills.
- **EWC** achieves relatively low FWT and AUC on both task suites. Penalizing changes to important parameters does not necessarily produce an effective balance between adaptation and retention.

Interestingly, the relative performance of ER and PackNet changes depending on the task suite. PackNet achieves a higher AUC than ER on LIBERO-Spatial (0.63 compared to 0.56), whereas ER performs better on LIBERO-Long (0.32 compared to 0.25).

This suggests that the effectiveness of a continual-learning algorithm depends on the type of knowledge being transferred. An algorithm that effectively preserves knowledge across changes in spatial relationships may not be equally effective when both declarative and procedural knowledge vary simultaneously.

MTL achieves higher AUC than the sequential methods in both comparisons. However, because MTL has simultaneous access to all task datasets, its results should be interpreted as a reference for joint training rather than a directly equivalent continual-learning method. <d-cite key="liu2023libero"></d-cite>

#### Interaction between policy architecture and learning algorithm

Another important research question is whether certain policy architectures are inherently better suited for lifelong robot learning. The authors investigate this by combining ResNet-RNN, ResNet-T, and ViT-T with different continual-learning algorithms.

The following table presents the LIBERO-Long results for Experience Replay and PackNet, as shown in Table 1 of the paper and discussed during the lecture.

| Policy architecture | ER FWT ↑ | ER NBT ↓ | ER AUC ↑ | PackNet FWT ↑ | PackNet NBT ↓ | PackNet AUC ↑ |
|---|---:|---:|---:|---:|---:|---:|
| ResNet-RNN | 0.16 ± 0.02 | **0.16 ± 0.02** | 0.08 ± 0.01 | 0.13 ± 0.00 | 0.21 ± 0.01 | 0.03 ± 0.00 |
| ResNet-T | **0.48 ± 0.02** | 0.32 ± 0.04 | **0.32 ± 0.01** | 0.22 ± 0.01 | **0.08 ± 0.01** | 0.25 ± 0.00 |
| ViT-T | 0.38 ± 0.05 | 0.29 ± 0.06 | 0.25 ± 0.02 | **0.36 ± 0.01** | 0.14 ± 0.01 | **0.34 ± 0.01** |

The results show that the relative ranking of policy architectures changes depending on the continual-learning algorithm:

- With Experience Replay, ResNet-T achieves the highest AUC of 0.32, outperforming ViT-T at 0.25.
- With PackNet, ViT-T achieves the highest AUC of 0.34, outperforming ResNet-T at 0.25.
- ResNet-RNN achieves the lowest NBT under Experience Replay, but its low FWT results in poor overall performance.

The comparison between ResNet-T and ViT-T is particularly interesting because neither architecture consistently outperforms the other. Under Experience Replay, ResNet-T learns new tasks more effectively but also experiences more forgetting. Under PackNet, ViT-T achieves better new-task learning and overall performance despite having higher NBT than ResNet-T.

Furthermore, the ResNet-RNN results illustrate why minimizing forgetting alone is insufficient. A policy that acquires relatively little knowledge may have less knowledge to lose, resulting in a low NBT without necessarily being an effective lifelong learner.

Therefore, policy architectures and continual-learning algorithms should be evaluated jointly rather than treated as independent design choices. A stronger architecture under one learning algorithm does not necessarily remain stronger under another. <d-cite key="liu2023libero"></d-cite>

#### Does pretraining improve lifelong learning?

The authors also investigate whether pretraining on a diverse set of manipulation tasks improves subsequent knowledge transfer.

The motivation is that a robot exposed to many shorter manipulation tasks should acquire reusable representations and behaviors before encountering more complex tasks. For example, prior experience with grasping objects, opening drawers, and placing objects into containers could potentially help a robot learn longer tasks that require combinations of these behaviors.

The experimental procedure consists of two stages:

1. **Pretraining:** Jointly train a policy using behavioral cloning on the 90 short-horizon tasks in LIBERO-90 for 50 epochs, selecting the best saved checkpoint.
2. **Downstream continual learning:** Initialize the policy with the pretrained parameters and sequentially train on the 10 long-horizon tasks in LIBERO-Long.

The pretrained policies are compared against policies initialized with random weights. A separate MTL reference jointly trains on the downstream task datasets.

| Learning algorithm | Policy architecture | With pretraining | Without pretraining | MTL reference |
|---|---|---:|---:|---:|
| ER | ResNet-RNN | 0.04 | 0.08 | 0.20 |
| ER | ResNet-T | 0.22 | 0.32 | 0.48 |
| ER | ViT-T | 0.10 | 0.25 | 0.46 |
| EWC | ResNet-T | 0.01 | 0.02 | 0.48 |
| PackNet | ResNet-T | 0.17 | 0.25 | 0.48 |

*Downstream performance on LIBERO-Long, comparing behavioral-cloning pretraining on LIBERO-90 with training from scratch.*

Contrary to the initial motivation, the pretrained policies perform worse than their randomly initialized counterparts in every configuration shown in the presentation. For example, the AUC of ER with ResNet-T decreases from 0.32 to 0.22 after pretraining, while PackNet with ResNet-T decreases from 0.25 to 0.17.

One possible interpretation is that representations acquired during supervised pretraining are not necessarily well suited for subsequent adaptation under sequential learning constraints. Knowledge that helps solve shorter manipulation tasks may not automatically transfer to more complex tasks requiring different combinations of behaviors. However, these results alone do not establish the exact cause of the performance degradation.

Importantly, this experiment does not demonstrate that pretraining is generally ineffective for robot learning. Rather, it demonstrates that the particular behavioral-cloning pretraining procedure evaluated in LIBERO does not guarantee improved downstream continual-learning performance. This motivates investigating pretraining objectives that explicitly encourage knowledge reuse and adaptability. <d-cite key="liu2023libero"></d-cite>

### LIBERO: Limitations and Takeaways

#### Limitations

Although LIBERO provides a structured framework for evaluating knowledge transfer in lifelong robot learning, several limitations should be considered when interpreting its results.

- **Simulation-based evaluation:** The experiments are conducted in simulated manipulation environments. Physical robots encounter additional challenges, including sensor noise, disturbances, calibration errors, and uncertainty during contact-rich manipulation. Consequently, successful retention in LIBERO does not necessarily establish reliable retention on physical robots.
- **Dependence on demonstrations:** The evaluated policies primarily acquire new tasks through behavioral cloning using human demonstrations. The results therefore characterize demonstration-based continual learning rather than more general settings where robots acquire knowledge through autonomous exploration or interaction.
- **Fixed benchmark task collection:** Although LIBERO provides a procedural task-generation pipeline, the reported experiments use predefined task suites. These controlled suites are useful for reproducible comparisons, but they do not fully capture the unpredictability or scale of an open-ended lifelong-learning environment.
- **Best-checkpoint evaluation:** The evaluation procedure selects the checkpoint with the highest evaluated success rate on the current task before proceeding. This assumes access to task-specific evaluation rollouts and an explicit checkpoint-selection procedure. A continuously deployed robot may not have the same ability to identify the best checkpoint before adapting to subsequent tasks.

The success-rate metrics also provide limited information about the underlying cause of failure. For example, a reduction in success rate may result from forgetting an object's identity, incorrectly interpreting its spatial relationship, or failing to execute a previously learned manipulation behavior. LIBERO's controlled task suites help distinguish these types of knowledge transfer, but further analysis would be required to identify the precise knowledge that was lost within a policy.

#### Key takeaways

Below are the most important takeaways:

1. **Adaptation does not imply retention.** Sequential fine-tuning can acquire new tasks effectively while experiencing substantial catastrophic forgetting. Therefore, evaluating new-task performance alone is insufficient.

2. **Retention does not imply effective learning.** Protecting previously learned parameters or minimizing NBT does not necessarily produce a useful lifelong learner if the policy struggles to acquire new skills. FWT, NBT, and AUC should be interpreted together.

3. **The type of knowledge transfer matters.** Experience Replay and PackNet perform differently across LIBERO-Long and LIBERO-Spatial, suggesting that the effectiveness of a retention mechanism depends on how tasks differ from one another.

4. **Policy architecture and learning algorithm interact.** ResNet-T performs better with Experience Replay, whereas ViT-T performs better with PackNet in the presented LIBERO-Long comparison. Architecture selection should therefore account for the continual-learning mechanism.

5. **Pretraining does not guarantee positive transfer.** The evaluated behavioral-cloning pretraining procedure reduces downstream performance across the configurations presented in the lecture. Acquiring more knowledge before continual learning does not necessarily make that knowledge easier to reuse.

The central contribution of LIBERO is not a new algorithm that eliminates catastrophic forgetting, but rather a benchmark that makes the tradeoffs between adaptation, retention, and knowledge transfer measurable. By controlling how tasks vary and evaluating performance throughout the learning sequence, LIBERO provides a foundation for investigating which forms of knowledge are transferable and which learning mechanisms are most effective at preserving them. <d-cite key="liu2023libero"></d-cite>

These findings also motivate exploring alternative approaches to knowledge reuse. Instead of continually updating a shared policy and risking interference with earlier skills, another possibility is to preserve independently learned policies and combine their knowledge when needed. This is the central idea explored by PoCo in the following section.

## PoCo: Policy Composition from and for Heterogeneous Robot Learning

### PoCo: Motivation and Problem

#### Why heterogeneous robot learning is difficult

Robots can learn manipulation skills from several useful sources: simulated demonstrations, human videos, and real robot teleoperation. Each source offers different strengths. Simulation provides large amounts of relatively inexpensive data, human videos are easy to collect and teach high level motions, and real robot demonstrations capture the actual robot's contact rich interactions. However, these sources also differ in their data distributions, embodiments, control frequencies, and available sensors. Observations may include RGB images, point clouds, tactile images, or robot proprioception.

This creates a data heterogeneity problem: simply pooling all demonstrations into one training dataset is difficult, especially when the observation modalities and domains do not match. Training a separate specialist for every task or data source is possible, but those specialists do not automatically share their useful information. Meanwhile, collecting enough real robot data to cover every task and situation can be expensive.

PoCo (Policy Composition) asks: Can we train policies separately on heterogeneous data and combine their knowledge at inference time, without retraining a single model on all the data?

The paper focuses on generalizable robotic tool use, including hammering, scooping with a spatula, cutting, and using a wrench. The goal is to produce robust manipulation trajectories by combining information from different tasks, behaviors, sensing modalities, and data domains.

#### Core idea

Rather than forcing all data into one model, PoCo learns modular probabilistic policies using diffusion models. At test time, it composes these policies by combining their predictions over the same action trajectory space. This allows, for example, a policy trained with point clouds to be combined with one trained using RGB or tactile observations, or simulation knowledge to be combined with real robot demonstrations.

The important compatibility requirement is that the policies being composed must output trajectories with the same action dimension and horizon. Their observation inputs do not need to be identical. PoCo therefore offers a way to reuse independently learned skills and add new information without discarding previously trained policies. <d-cite key="wang2024poco"></d-cite>

{% include figure.liquid
path="assets/img/2026-11-18-continual-learning-multi-task-learning/fig2poco.png"
class="img-fluid rounded z-depth-1"
alt="Overview of PoCo combining behaviors, data domains, robotic tasks, and sensing modalities through probabilistic policy composition."
caption="PoCo overview: policy composition combines information across behaviors, tasks, modalities, and domains. Source: Wang et al. (2024), Figure 2."
zoomable=true
avoid_scaling=true
%}

### PoCo: Method and Formulation

#### 1. Represent each policy as a trajectory distribution

PoCo represents a robot policy as a probability distribution over action trajectories:

$$
\pi(\tau \mid o)
$$

Here, $$\tau$$ is an action trajectory with horizon $$H$$ and action dimension $$d$$, and $$o$$ is the robot's observation (such as an image, point cloud, or tactile input).

Each policy is learned as a conditional diffusion model. During training, noise is added to expert action trajectories, and a neural network learns to predict that noise. At inference, the model starts with a noisy trajectory and repeatedly denoises it to generate an action sequence. The paper uses a mean squared error noise prediction objective:

$$
\mathcal{L}_{\mathrm{MSE}}
=
\left\|\epsilon_t-\epsilon_\theta(\tau_0+\epsilon_t,t\mid o)\right\|^2
$$

In this expression, $$\tau_0$$ is the demonstrated trajectory, $$\epsilon_t$$ is the noise added at diffusion step $$t$$, and $$\epsilon_\theta$$ is the learned noise prediction network. The key point is that the diffusion model represents a distribution of plausible robot trajectories, rather than outputting only one fixed action.

PoCo trains separate policies for different combinations of:

- **Modality $$M$$:** RGB image, point cloud, tactile image, or proprioception.
- **Domain $$D$$:** simulation, human demonstrations, or real robot data.
- **Task $$T$$:** the requested manipulation, such as hammering or scooping.
- **Behavior cost $$c$$:** an additional objective, such as smoothness or workspace safety.

#### 2. Compose policies using a product of distributions

The central mathematical idea is probabilistic composition. Given two trajectory distributions, PoCo defines their composition as

$$
p_{\mathrm{product}}(\tau)\propto p_1(\tau)\,p_2(\tau).
$$

This product assigns high probability to trajectories supported by both policies. In other words, the composed policy favors trajectories that satisfy the information or requirements represented by each component.

The paper applies this idea in three main ways:

- **Task level composition:** combines a general policy with task conditioned information, encouraging trajectories that accomplish a specified task. The paper uses a guidance weight to control how strongly the task condition influences generation.
- **Behavior level composition:** combines a task policy with a cost function, so generated trajectories both perform the task and satisfy desired properties.
- **Domain level composition:** combines policies trained on different domains or modalities, allowing their learned trajectory information to complement one another.

For behavior composition, a cost $$c(\tau)$$ is converted into a probability like preference using a Boltzmann factor:

$$
p_{\mathrm{cost}}(\tau)\propto \exp[-c(\tau)],
\qquad
p_{\mathrm{product}}(\tau)\propto
p_{\mathrm{cost}}(\tau)\,p(\tau\mid T).
$$

A lower cost therefore gives a trajectory higher probability. The paper gives smoothness and workspace bound constraints as examples.

#### 3. Combine predictions during diffusion sampling

A product of distributions can be implemented by adding their corresponding score or energy gradient predictions. Intuitively, each component policy contributes a direction that guides the noisy trajectory toward regions it considers likely. The combined direction guides sampling toward trajectories supported by the composed policies.

For example, domain composition adds the predictions from two independently trained diffusion policies:

$$
\tau^{t-1}
=
\tau^t-\gamma_D
\left(\epsilon_{\theta_1}(\tau^t\mid t)
+\epsilon_{\theta_2}(\tau^t\mid t)\right)+\xi.
$$

Here, $$\tau^t$$ is the current noisy trajectory, $$\gamma_D$$ is a composition step size/weight, and $$\xi$$ is Gaussian sampling noise. The same general procedure can include task guidance and gradients from behavior costs. The process repeats over denoising steps until a final trajectory is produced.

In short, PoCo does not merge the component networks' parameters or retrain them jointly. It combines their trajectory predictions at inference time. This is what makes the approach modular: new policies or objectives can be added to the composition, provided their outputs use a compatible action trajectory representation. <d-cite key="wang2024poco"></d-cite>

{% include figure.liquid
path="assets/img/2026-11-18-continual-learning-multi-task-learning/fig3poco.png"
class="img-fluid rounded z-depth-1"
alt="Diagram showing diffusion-policy composition across sensing modalities and domains, task-conditioned policies, and behavior costs."
caption="Policy composition at inference time: domain, task, and behavior composition combine diffusion-model predictions. Source: Wang et al. (2024), Figure 3."
zoomable=true
avoid_scaling=true
%}

### PoCo: Experiments and Results

PoCo is evaluated on four tool-use tasks: scooping with a spatula, cutting with a knife, hammering, and turning a wrench. Simulation experiments use the Fleet-Tools benchmark with point-cloud observations, averaging 10 runs over 50 held-out scenes per task. Real-world experiments use a Franka Panda with wrist and overhead cameras and a GelSight tactile finger. The metric is task success rate. <d-cite key="wang2024poco"></d-cite>

#### Behavior and task composition

Adding smoothness or workspace costs at test time lowered each cost (smoothness from 0.027 to 0.016, workspace violation from 0.030 to 0.022) while success fell slightly, from 0.70 to 0.67, without retraining. Task composition, implemented as classifier-free guidance between conditioned and unconditioned multitask policies, outperformed both multitask baselines on all four tasks in simulation. On the real robot the results were more mixed:

| Policy | Spatula | Knife | Hammer | Wrench | Average |
|---|---:|---:|---:|---:|---:|
| Single-task | 8/10 | 8/10 | 5/10 | 5/10 | 65% ± 17% |
| Multitask, unconditioned | 6/10 | 5/10 | 5/10 | 4/10 | 50% ± 8% |
| Multitask, conditioned | 8/10 | 5/10 | 6/10 | 2/10 | 53% ± 25% |
| Multitask composition ($$\alpha = 2$$) | 8/10 | 4/10 | 7/10 | 5/10 | 60% ± 18% |

Composition was the best multitask approach, but separate single-task policies still had the highest average, and with 10 trials per task the differences fall within the reported standard deviations.

#### Domain composition

Domain composition produced the largest gains. On a real-world hammer-reaching task tested under four scene changes (5 trials each), composed policies outperformed each of their components:

| Scene change | Human | Simulation | Real robot | Human + Real | Sim + Real |
|---|---:|---:|---:|---:|---:|
| Vary object pose | 1/5 | 5/5 | 2/5 | 4/5 | 5/5 |
| Vary robot and tool pose | 1/5 | 5/5 | 4/5 | 5/5 | 5/5 |
| Distractor objects | 0/5 | 5/5 | 2/5 | 3/5 | 5/5 |
| Novel object instance | 1/5 | 3/5 | 2/5 | 1/5 | 5/5 |
| Total | 15% | 90% | 50% | 65% | 100% |

{% include figure.liquid
path="assets/img/2026-11-18-continual-learning-multi-task-learning/poco-fig-09-domain-composition-real.png"
class="img-fluid rounded z-depth-1"
alt="Grid of real-robot photos comparing Real, Real plus Human, Simulation, and Simulation plus Real policies under different distractors, object instances, and object, pose, and tool changes."
caption="Real-world domain composition. The Simulation + Real composition succeeds in all three scene variations shown. Source: Wang et al. (2024), Figure 9."
zoomable=true
avoid_scaling=true
%}

In an ablation, pooling simulation point clouds and real images into a single model reduced success by 75% relative to composition, despite the larger model. Removing tactile input reduced success by 38%. Composition weights mattered throughout: performance peaked at intermediate values of $$\alpha$$ and $$\gamma_D$$. <d-cite key="wang2024poco"></d-cite>

### PoCo: Limitations and Takeaways

PoCo combines policies at a single moment rather than sequencing them, so it was evaluated only on short-horizon tool use. All composed policies must share an action space and be roughly aligned in horizon, scale, and control frequency. Each denoising step evaluates every component, so inference slows as policies are added; two policies already limit control to 5 Hz. Composition weights are tuned by hand within a narrow stable range, policies still fail in delicate contact-rich tasks, and the evaluation is small: four tasks with 5 to 10 real-world trials per condition. <d-cite key="wang2024poco"></d-cite>

The main lesson is that heterogeneous data can be used by training separate policies and combining their predictions, which worked better than pooling the data into one network. For continual learning, this suggests a modular form of retention: existing policies are never updated, so new data or tasks cannot overwrite earlier skills, although the paper does not test this in a sequential benchmark. The costs are growing inference time and the need for compatible action spaces. SRSA also keeps a library of specialist policies but, instead of combining their predictions, retrieves the most relevant one and adapts its parameters to the new task.

## SRSA: Skill Retrieval and Adaptation for Robotic Assembly Tasks

### Motivation and Problem

While LIBERO investigates how robots acquire new knowledge while retaining previously learned capabilities, and PoCo explores how independently trained policies can be composed at inference time, SRSA introduces an alternative approach based on skill retrieval and adaptation.

Robotic assembly tasks, such as peg insertion, stacking, and mechanical component assembly, often require precise manipulation and contact-rich interactions. Training a new policy from scratch for every assembly task can be computationally expensive and sample-inefficient. However, previously learned assembly skills may contain transferable knowledge that can facilitate learning new tasks.

**SRSA (Skill Retrieval and Adaptation)** addresses this problem by maintaining a library of pretrained specialist policies. When encountering a new assembly task, the system retrieves a relevant existing policy and adapts it to the new task instead of learning entirely from scratch. <d-cite key="guo2025srsa"></d-cite>

The central research question is:

> How can a robot identify and reuse previously learned skills to efficiently solve unfamiliar assembly tasks?

### Method and Formulation

The SRSA framework consists of three main components:

- **Skill Library:** Stores pretrained specialist policies for previously learned assembly tasks.
- **Skill Retriever:** Identifies existing policies that are likely to transfer successfully to a new task.
- **Skill Adapter:** Fine-tunes a selected policy using reinforcement learning in a simulation environment.

#### Pretrained Skill Library

SRSA assumes that a collection of pretrained specialist policies is available before learning a new task. Each specialist is trained to perform a particular assembly task.

Conceptually, the skill library can be represented as

$$
\mathcal{L} =
\{(\mathcal{T}_i,\pi_i)\}_{i=1}^{N},
$$

where $\mathcal{T}_i$ represents a previously learned assembly task, $\pi_i$ represents its corresponding specialist policy, and $N$ is the number of available policies.

Given a new target task $\mathcal{T}_{\mathrm{new}}$, the objective is to identify an existing policy that provides a useful starting point for learning the target task.

Unlike conventional continual learning methods that repeatedly update a shared model, SRSA maintains separate specialist policies. This allows previously learned behaviors to remain available without directly modifying the original policies during adaptation.

#### Skill Retrieval

The skill retriever determines which pretrained policy is most suitable for a new assembly task.

Rather than selecting a policy solely based on the visual similarity between objects, the retriever estimates how successfully an existing policy can perform the target task.

Let $f_{\phi}$ denote a learned transfer success predictor parameterized by $\phi$. The predicted transfer performance of a source policy can be conceptually expressed as

$$
\hat{s}_i =
f_{\phi}(\mathcal{T}_i,\mathcal{T}_{\mathrm{new}}),
$$

where $\hat{s}_i$ represents the estimated transfer success of the specialist policy associated with task $\mathcal{T}_i$.

The retrieval objective can be simplified as

$$
i^* =
\arg\max_{i \in \{1,\ldots,N\}}
\hat{s}_i.
$$

The system identifies promising candidate policies and evaluates them in simulation before selecting a policy for adaptation. <d-cite key="guo2025srsa"></d-cite>

An important challenge is that geometric similarity does not necessarily imply behavioral compatibility. For example, inserting a peg into a hole and placing a ring around a shaft may involve similar insertion motions but require different alignment and contact strategies. Consequently, an effective retriever must identify policies that provide transferable manipulation behaviors rather than simply recognizing similar objects.

#### Policy Adaptation

After retrieving a suitable specialist policy, SRSA adapts it to the target assembly task using reinforcement learning.

Instead of initializing the target policy randomly, the retrieved policy provides the initial parameters:

$$
\theta_{\mathrm{new}}^{(0)}
=
\theta_{i^*},
$$

where $\theta_{i^*}$ represents the parameters of the selected specialist policy.

The retrieved policy is subsequently fine-tuned using Proximal Policy Optimization (PPO). The general reinforcement learning objective is to maximize the expected discounted cumulative reward:

$$
J(\theta)
=
\mathbb{E}_{\tau \sim \pi_{\theta}}
\left[
\sum_{t=0}^{H-1}
\gamma^t r_t
\right],
$$

where

- $\pi_{\theta}$ represents the parameterized robot policy.
- $\tau$ represents a trajectory generated by the policy.
- $H$ represents the trajectory horizon.
- $\gamma$ represents the discount factor.
- $r_t$ represents the reward received at time step $t$.

Through fine-tuning, the retrieved policy gradually adapts its behavior to the requirements of the new assembly task.

This approach allows the robot to begin learning with previously acquired manipulation knowledge rather than discovering the entire assembly strategy from scratch.

#### Adaptation and Retention

An important characteristic of SRSA is that it separates adaptation from retention.

- **Adaptation** is achieved by fine-tuning the retrieved specialist policy for the new task.
- **Retention** is achieved by preserving the original specialist policies in the skill library.

In contrast to sequential fine-tuning, where learning a new task may overwrite previously acquired knowledge, SRSA preserves existing policies while adapting a selected policy to the target task.

This separation provides an alternative approach to addressing the stability–plasticity dilemma in continual robot learning.

### Experiments and Results

The authors evaluate SRSA on robotic assembly tasks and compare its performance with alternative learning approaches, including AutoMate, which learns specialist policies without SRSA's proposed skill retrieval mechanism.

The experiments investigate whether retrieving a previously learned policy provides a better initialization for learning an unfamiliar assembly task.

The reported results demonstrate that SRSA improves learning performance compared with training policies from scratch. Retrieving a relevant specialist allows the robot to reuse existing manipulation knowledge, reducing the need to rediscover previously learned behaviors.

The main experimental findings can be summarized as follows:

| Evaluation Aspect | Main Finding |
| :--- | :--- |
| Skill retrieval | Identifies previously learned policies that are promising candidates for transfer to new assembly tasks. |
| Policy adaptation | Fine-tuning a retrieved policy can improve learning efficiency compared with training from scratch. |
| Task performance | SRSA achieves better assembly performance than the evaluated AutoMate baseline. |
| Knowledge retention | Previously learned specialist policies remain available while the selected policy is adapted. |

These results suggest that explicit skill retrieval can facilitate knowledge transfer in robotic assembly tasks. <d-cite key="guo2025srsa"></d-cite>

However, the effectiveness of this approach depends on the availability of relevant pretrained policies and an appropriate simulation environment for adaptation.

### Limitations and Takeaways

Although SRSA demonstrates the benefits of retrieving and adapting previously learned skills, several limitations were identified during the lecture and classroom discussion.

**Dependence on the skill library.** SRSA assumes that a collection of relevant pretrained specialist policies is available. If a new task differs significantly from previously learned tasks, the skill library may not contain a suitable policy for adaptation.

**Skill retrieval accuracy.** Identifying transferable skills is challenging because assembly tasks with similar geometric structures may require different manipulation strategies. An inaccurate retrieval decision may provide an unsuitable initialization for subsequent learning.

**Dependence on simulation.** SRSA assumes that a simulation environment is available in which candidate policies can be evaluated and adapted. This assumption may limit its applicability to unstructured real-world environments where accurate simulation models are difficult to obtain.

**Scalability.** Maintaining separate specialist policies avoids directly overwriting previously learned behaviors. However, as the number of tasks increases, maintaining and searching an expanding skill library may introduce additional storage and computational costs.

**Retrieval objective.** A policy with high zero-shot success may not necessarily be the best policy for subsequent adaptation. A policy with lower initial performance could potentially adapt faster or achieve better final performance. This raises the question of whether retrieval should optimize initial success, adaptation efficiency, or final task performance.

Overall, SRSA provides an alternative perspective on knowledge reuse by retrieving and adapting pretrained specialist policies. While PoCo reuses policy predictions through composition and LIBERO evaluates knowledge transfer through continual learning, SRSA reuses previously learned policy parameters through retrieval and fine-tuning.

The key takeaway is that separating knowledge retention from adaptation can help robots preserve existing skills while efficiently learning new tasks. However, the effectiveness of this approach depends on the quality of the skill library, retrieval accuracy, and the availability of suitable environments for policy adaptation.

## Discussion

The following summarizes questions from the audience and the discussion with the presenters. The exchanges are paraphrased from the class notes, with supporting paper details identified separately.

### Attention maps and explainability

**Audience question:** Does LIBERO include attention maps, and can they explain how the robot interacts with objects?

**Discussion with the presenters:** Attention maps were not included in the presentation, but they appear in the paper's supplementary material. They can highlight important objects or image regions, yet do not fully explain the details of interactions or the sequence of actions.

**Paper context:** These are perturbation-based saliency maps. The paper also reports similar-looking maps for ER and EWC despite different performance, illustrating the limits of using attention to explain behavior. <d-cite key="liu2023libero"></d-cite>

### Overlapping object and spatial changes

**Audience question:** What happens when object identity and spatial arrangement change together, instead of being varied separately?

**Discussion with the presenters:** LIBERO-100 partially addresses this situation through tasks involving mixed changes. However, the discussion did not establish that it systematically covers every combination of object and spatial changes.

**Paper context:** LIBERO-Spatial and LIBERO-Object emphasize individual factors, while LIBERO-100 includes mixed changes in objects, layouts, and goals. <d-cite key="liu2023libero"></d-cite>

### Identifying important parameters

**Audience question:** How can freezing methods determine which parameters are important, when apparently minor weights may still contribute through less obvious interactions?

**Discussion with the presenters:** The concern was that parameter importance is difficult to interpret completely. An importance estimate may miss contributions that are not obvious from individual weights. This remained a question about the reliability of those estimates.

**Paper context:** PackNet has a concrete selection rule based on weight magnitude, followed by fine-tuning and freezing. Its masks protect retained old-task computation from later updates; the concern here is whether the selection preserves useful behavior before freezing. <d-cite key="mallya2018packnet"></d-cite>

### Adding new tasks and reusing old data

**Audience question:** When a new task or new data is added, does learning also involve training on earlier data again?

**Discussion with the presenters:** Reusing old data is already covered by an approach evaluated in LIBERO. The discussion clarified that this is an existing mechanism for retaining earlier skills while learning a new task.

**Paper context:** Experience Replay (ER) mixes stored samples from previous tasks with demonstrations for the current task. It does not require retraining on the entire accumulated dataset from scratch. <d-cite key="liu2023libero"></d-cite>

### Expert modules as a future direction

**Audience question:** Could the model reuse or add expert modules to learn new tasks while protecting earlier skills?

**Discussion with the presenters:** No specific expert-module approach was established in the discussion. Expert modules were raised as a possible direction for future continual learning research.

**Paper context:** LIBERO does not report a separate expert-module or mixture-of-experts baseline. PackNet already partitions a fixed network across tasks, but the benefits of adding separate experts are not established by these experiments. <d-cite key="liu2023libero"></d-cite>
