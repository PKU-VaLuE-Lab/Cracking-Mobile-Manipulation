<div align="center">

# Cracking Mobile Manipulation

### From Failure Diagnosis to Agentic Robot System

**Why do strong local manipulation policies fail when deployed in long-horizon mobile manipulation, and which interventions help most?**

<p>
  <a href="./paper.pdf">
    <img alt="Preprint PDF" src="https://img.shields.io/badge/Paper-Preprint%20PDF-b31b1b?style=for-the-badge&logo=adobeacrobatreader&logoColor=white">
  </a>
  <a href="#-release-status">
    <img alt="Code coming soon" src="https://img.shields.io/badge/Code-Coming%20Soon-f59e0b?style=for-the-badge&logo=github&logoColor=white">
  </a>
</p>

</div>

## 🧭 Release

🚧 **Coming soon.** We plan to release the following components:

- [ ] **Subtask VLA training code** — Training scripts and configurations for subtask vla.
- [ ] **Subtask VLA evaluation** — Evaluation scripts and benchmark configurations for subtask vla.
- [ ] **Subtask VLA checkpoint** 
- [ ] **Agentic robot system** — High-level planner, skill library, adaptive visual memory, and geometric adjustment tools.
- [ ] **System evaluation** — Task definitions, evaluation scripts, and episode seeds for long-horizon mobile manipulation.
- [ ] **Setup and reproduction guide** — Environment setup and instructions for running the released components.


## Overview

Vision-language-action (VLA) models can execute local manipulation task, but coordinating those skills across changing viewpoints, robot base poses, and multiple workspaces remains challenging. In our evaluation, a pretrained subtask VLA reaches **80%** success on atomic RoboCasa365 tasks, while an agentic system built on that VLA succeeds on only **19%** of long-horizon mobile-manipulation episodes. We study **where** failures occur and use that diagnosis to decide **which capabilities to add**. Our failure-guided agentic framework augments a fixed subtask VLA with in-context planning guidance, adaptive visual recall, and geometric adjustment.

<p align="center">
  <img src="./assets/overview.png" alt="Mobile manipulation success gap, failure attribution, and targeted interventions" width="94%">
</p>

**Main contributions**

- 🔎 **A systematic failure diagnosis of mobile manipulation.** We analyze the gap between strong subtask-level manipulation and long-horizon mobile manipulation, identifying three dominant failure regimes: task planning, navigation and base positioning, and local manipulation. The analysis turns overall success gap into a concrete, stage-wise failure structure.
- 🧩 **A failure-guided agentic intervention framework.** Based on the diagnosed bottlenecks, we introduce targeted interventions around a fixed subtask VLA, including demonstration-based planning guidance, adaptive visual memory, accessibility-aware base adjustment, and local end-effector adjustment. 
- 📈 **Agentic intervention consistently improves the performance of mobile manipulation.** The proposed interventions improve the success rate from 19.0\% to 47.0\% on RoboCasa365. Our experiments further demonstrate consistent gains on held-out RoboCasa365 episodes, BEHAVIOR-1K tasks, and a real dual-arm mobile manipulator. 

---

## 🔍 Where Does Mobile Manipulation Fail?

We study **20 mobile-manipulation tasks** selected from the **32 composite tasks** in RoboCasa365. The agentic baseline achieves **19/100** successful episodes on the development seeds, leaving **81 baseline failures**. The breakdown below is **the share of those 81 failures**, *not* the percentage of all episodes.

| Primary failure type | Baseline failed episodes | Share of baseline failures | Typical symptoms |
|:--|--:|--:|:--|
| **Task planning** | 17 | **21.0%** | Wrong target, wrong receptacle, omitted or misordered subtasks |
| **Navigation & base positioning** | 21 | **25.9%** | Ineffective approach, poor manipulation stance, or navigation issues |
| **Local manipulation** | 43 | **53.1%** | Placement errors, empty grasps, object drops, or collisions |
| **Total** | **81** | **100%** | |

Our key observation is that **reaching the correct area does not imply being ready to manipulate**. The planner must preserve task progress, the base must reach a usable stance, and the arm must align with the intended interaction.

---

## 🛠️ Failure-Guided Agentic Framework

<p align="center">
  <img src="./assets/framework.png" alt="Failure-guided agentic framework with skill library, visual memory, and geometric skills" width="96%">
</p>

The framework has four complementary intervention groups:

1. **Skill library from demonstrations.** Demonstration videos from the training split are distilled into concise skill cards with a task goal, ordered subtasks, representative keyframes, and completion criteria. The high-level planner consults the relevant card to guide its next decision.
2. **Adaptive visual memory.** When the planner needs missing historical context, a visual retrieval model finds relevant earlier observations from the *current episode*. The method retrieves the **top 16 frames**, without access to future observations.
3. **Accessibility check and base adjustment.** RGB-D visual grounding estimates the target location. An accessibility check categorizes the current stance as *reachable*, *marginal*, or *out of reach*. The system performs a small base correction or plans a new stance as appropriate; this is a coarse accessibility check, not a full inverse-kinematics or collision-feasibility guarantee.
4. **End-effector adjustment.** Local target grounding and 3D geometric estimation guide bounded corrections for grasping and placement, complementing rather than replacing the subtask VLA.

The resulting system targets the failure stage that currently prevents task completion, instead of uniformly scaling or retraining the underlying RoboCasa365 policy.

---

## 🧪 Benchmarks & Evaluation

| Setting | Evaluation protocol | Purpose |
|:--|:--|:--|
| **RoboCasa365 — development** | 20 mobile-manipulation tasks × 5 matched episodes = **100 trials** | Failure diagnosis and progressive integration |
| **RoboCasa365 — held-out** | Same 20 tasks × 5 **separate seeds** = **100 trials** | Evaluate the finalized design on different initial conditions |
| **BEHAVIOR-1K** | 5 long-horizon tasks, **100 trials** | Evaluate transfer of the intervention design |
| **Real robot** | Astribot-S1, 2 tasks × 10 trials = **20 trials** | Test closed-loop task completion on hardware |

**Important evaluation distinction:** For RoboCasa365, the agentic configurations share a fixed subtask VLA; the high-level planner and agentic intervention design are varied. For BEHAVIOR-1K and real-robot evaluation, the respective low-level policies are trained or fine-tuned for those domains. These studies test **transfer of the intervention design**, not zero-shot transfer of a single VLA checkpoint.

---

## 📊 Headline Results

### Progressive integration on RoboCasa365

Each row **adds one component** to the configuration directly above it; all configurations are evaluated on the same 100 development episodes.

| System configuration | Success / 100 | Success rate |
|:--|--:|--:|
| Agentic baseline | 19 | 19% |
| + Demonstration-based skill library | 25 | 25% |
| + Adaptive visual memory | 29 | 29% |
| + Accessibility-aware base adjustment | 40 | 40% |
| **+ Local end-effector adjustment (full system)** | **47** | **47%** |

<p align="center">
  <img src="./assets/progressive_integration.png" alt="Progressive integration improves success from 19 to 47 percent" width="78%">
</p>

The largest incremental improvement occurs after **base adjustment (29% → 40%)**, emphasizing the importance of positioning the base for manipulation.

### Development and held-out performance

| Method | Development success (%) | Held-out success (%) |
|:--|--:|--:|
| Agentic baseline (adapted from Harness VLA) | 19 | 18 |
| **Ours — full system** | **47** | **49** |

The held-out evaluation is conducted only after fixing the system design, prompts, and tool parameters. Its matched comparison shows **18% → 49%** success on different scene and object initializations.

### Transfer beyond RoboCasa365

| Evaluation | Agentic baseline | Full system |
|:--|--:|--:|
| **BEHAVIOR-1K** (5 tasks, 100 trials) | 14% | **37%** |
| **Real mobile manipulator** (2 tasks, 20 trials) | 5/20 | **13/20** |

On the real robot, the two tasks are **Storage Box Transport** (2/10 → 6/10) and **Table Service** (3/10 → 7/10). A representative Table Service rollout is shown below.

<p align="center">
  <img src="./assets/real_robot.png" alt="Representative real-robot Table Service rollout on Astribot-S1" width="94%">
</p>

### The remaining failure frontier shifts

As planning and base-positioning failures are addressed, more episodes reach later manipulation stages. The remaining failures increasingly involve placement errors, object drops during transport, and collision-induced scene disruption. This does **not** mean manipulation necessarily deteriorated; the set of episodes reaching manipulation has changed.

<p align="center">
  <img src="./assets/failure_distribution.png" alt="Failure counts for navigation and base positioning, planning, manipulation, and overall failures across the progressive system" width="83%">
</p>


## 📚 Citation

If you find this project useful, please cite:

```bibtex
@misc{wang2026crackingmobilemanipulation,
  title  = {Cracking Mobile Manipulation: From Failure Diagnosis to Agentic Robot System},
  author = {Zixuan Wang and Jia-Ning Li and Yinzhou Tang and Yanzhao Yu and Biao Wu and Meng Fang and Xin Li and Yiwu Zhong},
  year   = {2026},
  note   = {Preprint}
}
```

---

## 🙏 Acknowledgements

Our experiments build on open research infrastructure including **RoboCasa365**, **BEHAVIOR-1K**, and pretrained **vision-language-action** policies. The agentic baseline is adapted from **Harness VLA**. We thank the authors and maintainers of the tools and benchmarks that support mobile manipulation research; please see the [paper](./paper.pdf) for detailed references.

