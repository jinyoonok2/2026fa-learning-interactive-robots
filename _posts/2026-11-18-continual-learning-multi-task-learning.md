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
  - name: "Comparison and Synthesis"
  - name: "Discussion"
---

<!--
Report in progress. The introduction, LIBERO motivation and methods, figure captions, and classroom discussion have been drafted.
LIBERO results and limitations, the PoCo sections, and the comparison remain to be written.
Keep ready: false until the report is complete and reviewed.
Add verified BibTeX entries to assets/bibliography/2026-11-18-continual-learning-multi-task-learning.bib.
Store figures in assets/img/2026-11-18-continual-learning-multi-task-learning/.
Store interactive HTML in assets/html/2026-11-18-continual-learning-multi-task-learning/.
-->

## Introduction

### Background and Motivation

A robot working in a changing environment needs to acquire new skills while retaining the ones it already uses. However, traditional machine learning approaches typically assume that training data are available before deployment and that models are trained offline on relatively stable data distributions. These assumptions are often unrealistic in robotics, where new objects, environments, and tasks may be encountered continuously.

Consider a household robot that has learned to open a drawer. Ideally, the knowledge acquired from this task should help the robot learn to open cabinets or other objects with similar mechanisms, rather than requiring it to learn each task from scratch. However, learning to manipulate a new object can update the same model parameters that supported an earlier task, reducing performance on that task. This loss of previously learned behavior is called **catastrophic forgetting**.

These challenges motivate two closely related research areas: **multi-task learning** and **continual learning**. Both aim to improve the efficiency, adaptability, and generalization of robot learning through knowledge sharing and reuse. <d-cite key="liu2023libero"></d-cite>

### Multi-task Learning

**Multi-task learning (MTL)** aims to learn multiple tasks simultaneously, with access to their datasets, by exploiting shared information across related tasks. A conventional multi-task learning architecture consists of a shared backbone, such as a convolutional neural network (CNN) or a Transformer, and multiple task-specific prediction heads. The shared backbone extracts common representations, while each prediction head specializes in a particular task.

In robotics, multi-task learning extends beyond simultaneous predictions. Robots must acquire reusable manipulation skills that can potentially be recombined to accomplish new tasks. For example, grasping, object alignment, and insertion skills may be shared across different manipulation and assembly tasks.

However, collecting robotic demonstrations and training policies from scratch for every new task can be expensive and time-consuming. Moreover, training datasets may originate from heterogeneous sources, including simulations, human demonstrations, and physical robots. These datasets may also contain different sensing modalities, such as RGB images, point clouds, and tactile observations.

Therefore, an important research question is whether independently learned robot policies can be effectively combined without requiring joint retraining. <d-cite key="wang2024poco"></d-cite>

### Continual Learning and Catastrophic Forgetting

**Continual learning (CL)**, also called lifelong learning, studies how an agent learns from a sequence of tasks when earlier training data are only partly available or no longer accessible. Unlike multi-task learning, which trains on several tasks together, continual learning requires a model to continuously acquire new knowledge while preserving previously learned capabilities.

One of the fundamental challenges in continual learning is catastrophic forgetting, where updating a model on new tasks significantly degrades its performance on previously learned tasks.

For example, consider an autonomous vehicle initially trained to operate under snowy conditions. If the vehicle subsequently learns to drive at night and under sunny conditions, sequential fine-tuning may cause it to forget how to operate safely in snow. Although retraining the model using all historical data could mitigate this problem, the computational and storage requirements would increase as more data become available.

Consequently, continual learning must balance two competing objectives:

- **Plasticity:** The ability to acquire new knowledge and adapt to unfamiliar tasks.
- **Stability:** The ability to preserve previously acquired knowledge and capabilities.

This challenge is commonly referred to as the **stability–plasticity dilemma**. The central challenge is to reuse earlier knowledge to learn new tasks efficiently while preserving earlier skills. <d-cite key="liu2023libero"></d-cite>

The following table summarizes the differences between multi-task learning and continual learning.

| Characteristics | Multi-task Learning | Continual Learning |
| :--- | :--- | :--- |
| Task availability | Simultaneous | Sequential |
| Training data access | Multiple task datasets | Current task and potentially limited historical data |
| Training process | Joint training | Incremental learning |
| Primary objective | Knowledge sharing across tasks | Adaptation and retention |
| Major challenges | Task interference and data heterogeneity | Catastrophic forgetting and limited resources |

### Adaptation, Retention, and Knowledge Transfer

An effective robot learning system should not simply accumulate task-specific behaviors. Instead, it should acquire transferable knowledge that can be reused across different objects, spatial configurations, and task objectives.

Three important concepts characterize knowledge reuse in robot learning:

- **Adaptation:** The ability to adjust existing behaviors to new tasks or environmental conditions.
- **Retention:** The ability to preserve previously learned capabilities after acquiring new knowledge.
- **Knowledge transfer:** The ability to leverage previously acquired knowledge to improve learning or execution on new tasks.

However, an important research question remains:

> **What should a robot reuse when learning a new task?**

Possible approaches include reusing historical training demonstrations, preserving model parameters, composing existing policy predictions, or retrieving and adapting previously learned specialist policies.

This lecture examines three complementary approaches to knowledge reuse:

- **LIBERO: Benchmarking Knowledge Transfer for Lifelong Robot Learning (2023)** provides a benchmark for studying knowledge transfer and catastrophic forgetting across sequential robot manipulation tasks. <d-cite key="liu2023libero"></d-cite>

- **PoCo: Policy Composition from and for Heterogeneous Robot Learning (2024)** investigates how separately trained diffusion policies can be composed at inference time to use information from heterogeneous data sources without joint retraining. <d-cite key="wang2024poco"></d-cite>

- **SRSA: Skill Retrieval and Adaptation for Robotic Assembly Tasks (2025)** explores how pretrained specialist policies can be retrieved from a skill library and adapted to previously unseen robotic assembly tasks.

These three papers investigate knowledge reuse at different levels, as summarized below.

| Paper | Knowledge Reuse Mechanism | Primary Research Objective |
| :--- | :--- | :--- |
| LIBERO | Historical demonstrations and model parameters | Evaluate knowledge transfer, adaptation, and retention |
| PoCo | Independently learned policy predictions | Compose policies without joint retraining |
| SRSA | Pretrained specialist policies | Retrieve and adapt existing skills to new tasks |

While LIBERO evaluates knowledge transfer through continual learning, PoCo focuses on policy composition at inference time, and SRSA investigates skill retrieval and adaptation. Together, these papers provide complementary perspectives on how robots can efficiently acquire new skills while preserving previously learned capabilities.

This report begins with LIBERO's problem setting and benchmark design, followed by PoCo's policy composition framework and SRSA's skill retrieval and adaptation approach. Finally, we compare their methodologies, experimental results, and limitations, summarize the classroom discussions, and explore potential future research directions in continual robot learning.

## LIBERO: Benchmarking Knowledge Transfer for Lifelong Robot Learning

Liu et al. introduce LIBERO as a simulation benchmark for **lifelong learning in decision making**. It combines a procedure for generating manipulation tasks, 130 fixed benchmark tasks, human demonstrations, and implementations of several learning algorithms and policy architectures. Its contribution is a common framework for investigating what transfers between tasks, what is forgotten, and how the learning setup affects both. <d-cite key="liu2023libero"></d-cite>

The figures below are reproduced from LIBERO and its supplementary material; the captions are our explanations of the original artwork.

### LIBERO: Motivation and Problem

Consider a robot that has learned to retrieve juice from a refrigerator. After learning other tasks, it might fail because it no longer identifies the correct object or location, or because it no longer executes the opening and grasping motions successfully. The final failure alone does not identify which knowledge was lost. LIBERO motivates its design through this distinction between two kinds of knowledge. <d-cite key="liu2023libero"></d-cite>

| Type of knowledge | What the robot needs to know | Manipulation example |
|---|---|---|
| Declarative knowledge | Object concepts, locations, and spatial relationships. | Identify the juice and locate it inside the refrigerator. |
| Procedural knowledge | Motions and behaviors needed to accomplish a goal. | Open the door, grasp the juice, and retrieve it. |

Robot manipulation requires these forms of knowledge to work together across a sequence of decisions. Recognizing the correct object is useful only if the robot can act on it, and a small execution error can prevent a later step from succeeding. This makes it difficult to diagnose transfer and forgetting using a final success rate alone.

LIBERO addresses this problem by controlling which aspects of tasks change. Some task suites emphasize spatial relationships, object identities, or goals separately; another combines several kinds of change. This supports five research questions:

1. **Knowledge transfer:** How does learning change when tasks vary in objects, spatial relationships, goals, or a mixture of these?
2. **Policy architecture:** How should a policy combine images, language, robot state, and observation history?
3. **Learning algorithm:** Which mechanisms preserve earlier skills while allowing new learning?
4. **Task ordering:** How sensitive is learning to the order in which tasks arrive?
5. **Pretraining:** Does training on a larger set of tasks help subsequent lifelong learning?

The benchmark provides a way to investigate these questions by varying task distributions, architectures, and learning algorithms within a shared evaluation framework. <d-cite key="liu2023libero"></d-cite>

{% include figure.liquid
   path="assets/img/2026-11-18-continual-learning-multi-task-learning/libero-fig-01-benchmark-overview.png"
   class="img-fluid rounded z-depth-1"
   alt="LIBERO overview showing four task suites and five research topics for lifelong robot learning."
   caption="LIBERO overview: four task suites and five research topics for lifelong robot learning. Source: Liu et al. (2023), main paper, Figure 1."
   zoomable=true
   avoid_scaling=true
%}

<!-- Source-artwork note: the two left descriptive subtitles in the original Figure 1 appear exchanged relative to the suite definitions in Section 4.2. Our caption follows those textual definitions; the extracted artwork is unchanged. -->

### LIBERO: Method and Formulation

#### Sequential learning setup

The robot encounters tasks $$\mathcal T_1,\ldots,\mathcal T_K$$ in sequence. Each task $$\mathcal T_k=(\mu_0^k,g_k)$$ specifies an initial-state distribution $$\mu_0^k$$ and a goal predicate $$g_k$$ that indicates whether a state satisfies the task. The tasks share the robot's state and action spaces and underlying environment dynamics.

For each task, the learner receives a demonstration dataset $$D_k$$. Earlier datasets are not fully available during later training, although a method such as replay can retain selected samples. The objective is to perform well across the tasks encountered so far. Evaluation therefore revisits earlier tasks to measure retention even though access to earlier training data is restricted. <d-cite key="liu2023libero"></d-cite>

#### Generating tasks with explicit goals

LIBERO builds its task-generation pipeline on the Robosuite manipulation simulator:

1. **Generate an instruction.** Extract activity templates from Ego4D language annotations and combine them with objects available in the simulator. For example, an opening-and-placement instruction can ask the robot to open a drawer and put a bowl inside.
2. **Specify the initial scene.** Select a suitable layout, object categories, placements, and initial object states. These define the distribution of starting configurations.
3. **Specify success conditions.** Use a PDDL-style task file to describe goal predicates, such as an open drawer and a bowl inside it. Execution succeeds when all required predicates are true.

The task file defines the environment and its success conditions; the learned policy produces the robot's actions. Ego4D supplies language templates for task creation, while robot-action demonstrations are collected separately. <d-cite key="liu2023libero"></d-cite>

{% include figure.liquid
   path="assets/img/2026-11-18-continual-learning-multi-task-learning/libero-fig-02-task-generation.png"
   class="img-fluid rounded z-depth-1"
   alt="Task-generation pipeline connecting human-activity templates to language instructions, scene specifications, initial states, and goal predicates."
   caption="Task generation: activity templates become language instructions and simulated tasks with defined starting states and goals. Source: Liu et al. (2023), main paper, Figure 2."
   zoomable=true
   avoid_scaling=true
%}

#### Four task suites

The procedural pipeline supports creating new tasks, while the paper uses fixed suites for reproducible comparisons.

| Task suite | Tasks | Main variation | What it tests |
|---|---:|---|---|
| LIBERO-Spatial | 10 | Locations and spatial relationships among the same objects. | Selecting the correct object through its spatial relation, such as distinguishing identical bowls. |
| LIBERO-Object | 10 | Target object types in pick-and-place tasks. | Recognizing and manipulating different objects while retaining a similar task structure. |
| LIBERO-Goal | 10 | Requested goals with fixed objects and spatial relationships. | Learning different motions and behaviors in a shared scene. |
| LIBERO-100 | 100 | Objects, layouts, goals, and interactions vary together. | Transferring mixed knowledge across more diverse tasks. |

LIBERO-100 is divided into **LIBERO-90**, containing 90 shorter tasks used for pretraining, and **LIBERO-Long**, containing 10 longer tasks used for downstream lifelong-learning evaluation. These are subdivisions of the 100-task suite. A long-horizon task requires dependent actions within an episode; continual learning concerns the sequence of tasks used for training. LIBERO-Long combines both challenges. <d-cite key="liu2023libero"></d-cite>

#### Policy architectures: from observations to actions

LIBERO compares three policies that process external-camera and wrist-camera images, a task instruction, and information about the robot's joints and gripper. Instructions are encoded with pretrained BERT embeddings. Each policy has a visual encoder, a temporal backbone for observation history, and a **Gaussian mixture model (GMM) action head**. <d-cite key="liu2023libero"></d-cite>

| Policy | Visual encoder | Temporal backbone | Language integration |
|---|---|---|---|
| ResNet-RNN | ResNet | LSTM | Language conditions visual features through FiLM and is included in the temporal input. |
| ResNet-T | ResNet | Transformer decoder | Language conditions the visual encoder and is supplied as a token to the temporal transformer. |
| ViT-T | Vision Transformer | Transformer decoder | Language is supplied as a token to both the visual and temporal transformers. |

The visual encoder extracts information about the scene. The temporal backbone combines information across observation steps. The GMM head predicts a distribution over continuous manipulation actions, from which an action is sampled for execution. This separates the choice of policy architecture from the choice of algorithm used to preserve knowledge across tasks.

{% include figure.liquid
   path="assets/img/2026-11-18-continual-learning-multi-task-learning/libero-supp-fig-05-policy-architectures.png"
   class="img-fluid rounded z-depth-1"
   alt="Three policy architectures combining camera images, language embeddings, and robot joint and gripper information with recurrent or transformer temporal processing."
   caption="Three policies tested in LIBERO: ResNet-RNN, ResNet-T, and ViT-T. Each combines images, language, and robot state to predict actions. Source: Liu et al. (2023), supplement, Figure 5."
   zoomable=true
   avoid_scaling=true
%}

The encoder diagram below shows how language enters visual processing. In the ResNet encoder, FiLM uses language-dependent scaling and shifting of intermediate features, followed by spatial softmax and a linear projection. The ViT encoder processes image-patch tokens together with a language token through self-attention. Both produce representations for the policy's temporal backbone. The temporal backbone summarizes observations within an episode; the continual-learning algorithm governs how the policy changes across training tasks.

{% include figure.liquid
   path="assets/img/2026-11-18-continual-learning-multi-task-learning/libero-supp-fig-06-image-encoders.png"
   class="img-fluid rounded z-depth-1"
   alt="ResNet and vision-transformer image encoders showing language-conditioned residual features and attention over image and language tokens."
   caption="Encoder details: ResNet and ViT turn images and language into features used by the policies above. Source: Liu et al. (2023), supplement, Figure 6."
   zoomable=true
   avoid_scaling=true
%}

#### Learning algorithms: preserving earlier skills

LIBERO implements three continual-learning approaches and two reference methods. A policy architecture can be paired with different learning algorithms, allowing their effects to be studied separately. <d-cite key="liu2023libero"></d-cite>

| Method | How training works | Resource or constraint |
|---|---|---|
| Sequential fine-tuning (SEQL) | Update the policy using only the current task's demonstrations. | No explicit mechanism protects earlier skills. |
| Experience Replay (ER) | Mix stored demonstrations from earlier tasks with current-task data. | Requires memory for replay data. |
| Elastic Weight Consolidation (EWC) | Use Fisher-based importance estimates to penalize changes to parameters important for earlier tasks. | Protection constrains later parameter updates. |
| PackNet | Train, prune, fine-tune, and freeze a task subnetwork; use the remaining unallocated parameters for later tasks. | Available capacity decreases as tasks are added. |
| Multi-task learning (MTL) | Jointly train using all task datasets. | Has broader data access than the sequential setting. |

ER preserves behavior through rehearsal, EWC through a soft penalty, and PackNet through parameter allocation and freezing. PackNet's task masks retain access to previously allocated weights while protecting the computation used for earlier tasks. MTL is a useful joint-training reference, but its access to all datasets differs from the lifelong-learning restriction; it is not a guaranteed performance ceiling. <d-cite key="liu2023libero,mallya2018packnet"></d-cite>

#### Demonstrations and training procedure

The policies are trained with **behavioral cloning**: supervised learning to reproduce human-demonstrated actions. The paper provides 50 demonstrations per task, collected through teleoperation with a 3Dconnexion SpaceMouse. Observations include images and robot state, while the task instruction conditions the policy. <d-cite key="liu2023libero"></d-cite>

A per-step negative log-likelihood form of the paper's behavioral-cloning objective is:

$$
\mathcal L_{\mathrm{BC}}(\theta;D_k)
=-\mathbb E_{(o_{\leq t},a_t)\sim D_k}
\left[\log\pi_\theta(a_t\mid o_{\leq t};\mathcal T_k)\right].
$$

Here, $$\theta$$ denotes policy parameters, $$o_{\leq t}$$ is observation history, and $$a_t$$ is the demonstrated action. Minimizing this loss increases the probability assigned to demonstrated actions. This expresses the supervised action-matching step; replay, regularization, or parameter freezing changes how that step is applied across tasks. The paper's Equation (2) gives the broader trajectory- and task-averaged formulation.

The sequential evaluation procedure is:

1. Train on the current task's 50 demonstrations for 50 epochs, applying the selected continual-learning mechanism.
2. Evaluate the current task every five epochs using 20 rollouts, each limited to 600 steps.
3. Select the earliest checkpoint attaining the best current-task success rate, then evaluate previously learned tasks with 20 rollouts each.
4. Continue to the next task and repeat the sequence across three random seeds.

PackNet additionally prunes, fine-tunes, and freezes its allocated parameters. The supplement specifies an extra 50-epoch fine-tuning stage and notes that its reported evaluation metrics are calculated before that stage. MTL follows joint training across task datasets. Separate studies vary task ordering or compare training from scratch with pretraining on LIBERO-90 before learning LIBERO-Long. <d-cite key="liu2023libero"></d-cite>

#### Evaluation equations from the W13 presentation

The presentation's evaluation slide (printed slide 29, PDF page 38) uses the three metrics from Equation (3) of the paper: **forward transfer (FWT)**, **negative backward transfer (NBT)**, and **area under the success-rate curve (AUC)**. They measure new-task learning, retention of earlier tasks, and their combination. All are based on task success rates. <d-cite key="liu2023libero"></d-cite>

{% include figure.liquid
   path="assets/img/2026-11-18-continual-learning-multi-task-learning/libero-fig-03-evaluation-metrics.png"
   class="img-fluid rounded z-depth-1"
   alt="Schematic success-rate curve for one task during acquisition and after learning later tasks, with areas illustrating FWT, NBT, and AUC."
   caption="Schematic of LIBERO's evaluation metrics: FWT (new-task learning), NBT (forgetting), and AUC (learning and retention). Source: Liu et al. (2023), main paper, Figure 3."
   zoomable=true
   avoid_scaling=true
%}

The equations use the following notation:

| Symbol | Meaning |
|---|---|
| $$K$$ | Total number of tasks in the training sequence. |
| $$k$$ | Task whose learning and retention are being measured. |
| $$\tau$$ | A later training stage, after task $$k$$. |
| $$e\in\mathcal E=\{0,5,10,\ldots,50\}$$ | The 11 evaluation checkpoints during current-task learning. |
| $$c_{i,j,e}$$ | Success rate on task $$j$$ after training task $$i$$ for $$e$$ epochs, following tasks $$1,\ldots,i-1$$. |
| $$c_{i,j}$$ | Success rate on task $$j$$ at the selected best checkpoint for training task $$i$$. |

Thus, $$c_{k,k}$$ is the best evaluated success rate when learning task $$k$$, while $$c_{\tau,k}$$ is its success rate after a later task $$\tau$$ has been learned. For metric calculation, the paper holds the current task's learning curve constant after its earliest best checkpoint.

##### Forward transfer: learning a new task

$$
\begin{aligned}
\mathrm{FWT}
&=\frac{1}{K}\sum_{k=1}^{K}\mathrm{FWT}_k,\\
\mathrm{FWT}_k
&=\frac{1}{11}\sum_{e\in\mathcal E}c_{k,k,e}.
\end{aligned}
$$

For each task, FWT averages success across the 11 learning checkpoints. A policy that reaches a high success rate earlier receives a higher score. The outer average summarizes this behavior across tasks. **Higher FWT is better.** LIBERO's definition measures new-task learning performance without subtracting a separately trained from-scratch baseline.

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

For task $$k$$, NBT compares its best learned performance with performance after each later task, then averages those differences. For example, a decline from 0.8 to 0.5 contributes a loss of 0.3 at that later stage. **Lower NBT is better.** Zero indicates no average change from the reference performance; a negative value indicates average improvement.

*Notation note:* The per-task expression applies when $$k<K$$. For the final task, there is no later training stage and the denominator is zero. The displayed aggregate preserves the paper and slide notation; numerical evaluation requires an explicit final-task convention.

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

For each task, AUC combines its acquisition score, $$\mathrm{FWT}_k$$, with success rates after later tasks. Dividing by $$K-k+1$$ averages the learning contribution and the subsequent retention measurements. The outer average combines all tasks. **Higher AUC is better**, reflecting stronger performance across learning and retention. This is the paper's specific normalized definition of AUC.

These three metrics make the learning-retention tradeoff visible: a policy can learn new tasks quickly while forgetting earlier ones, or retain earlier tasks while struggling to acquire new skills. They should therefore be interpreted together.

### LIBERO: Experiments and Results

<!-- TODO: Add the empirical comparisons, results, and supporting tables. The evaluation procedure and metric definitions are described above. -->

### LIBERO: Limitations and Takeaways

<!-- TODO: Add limitations and evidence-supported takeaways. -->

## PoCo: Policy Composition from and for Heterogeneous Robot Learning

### PoCo: Motivation and Problem

<!-- TODO: Add the motivation, problem setting, and research question. -->

### PoCo: Method and Formulation

<!-- TODO: Add the method, notation, and mathematical formulation. -->
<!-- TODO: Add method figures and an interactive walkthrough if useful. -->

### PoCo: Experiments and Results

<!-- TODO: Add the evaluation setup, baselines, metrics, and results with supporting figures or tables. -->

### PoCo: Limitations and Takeaways

<!-- TODO: Add limitations and evidence-supported takeaways. -->

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
## Comparison and Synthesis

<!-- TODO: Compare the papers and connect their lessons to the lecture topic. -->

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
