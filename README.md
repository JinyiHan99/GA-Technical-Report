<div align="center">
  <img src="assets/logo.png" alt="GenericAgent" width="140"/>

  <h1>GenericAgent (GA)</h1>
  <p><b>A Token-Efficient Self-Evolving LLM Agent via Contextual Information Density Maximization</b></p>

  <p>
    <a href="https://arxiv.org/abs/2604.17091"><img src="https://img.shields.io/badge/arXiv-2604.17091-b31b1b?logo=arxiv" alt="arXiv"></a>
    <a href="https://github.com/lsdefine/GenericAgent"><img src="https://img.shields.io/badge/Code-GenericAgent-black?logo=github" alt="Code"></a>
    <img src="https://img.shields.io/badge/Report-v1.0-blue" alt="Version">
  </p>

  <p><i>Advantage AI Agent Lab (A³ LAB) · Shenzhen Aquaintelling Technology × Fudan University</i></p>
</div>

---

> GA is a self-evolving LLM agent that matches or beats Claude Code / OpenClaw on hard tasks while spending **~1/6 the tokens**. It uses just **9 atomic tools**, a **92-line** core loop, and gets cheaper the more you use it (**−89.6%** tokens over 9 rounds).

This repo hosts the **Technical Report** ([PDF](GA_Technical_Report.pdf) · [arXiv](https://arxiv.org/abs/2604.17091)) and all **evaluation datasets**. For the runnable agent, see the source-code repo: [`GenericAgent`](https://github.com/lsdefine/GenericAgent).

## Contents

- [Contents](#contents)
- [News](#news)
- [Highlights](#highlights)
- [How It Works](#how-it-works)
- [Evaluation Results](#evaluation-results)
  - [1. Task Completion \& Token Efficiency](#1-task-completion--token-efficiency)
  - [2. Tool-Use Efficiency](#2-tool-use-efficiency)
  - [3. Memory System Effectiveness](#3-memory-system-effectiveness)
  - [4. Self-Evolution Capability](#4-self-evolution-capability)
  - [5. Web Browsing Capability](#5-web-browsing-capability)
- [Repository Layout](#repository-layout)
- [Citation](#citation)

## News

- **2026.04** — Featured on [机器之心 (jiqizhixin)](https://mp.weixin.qq.com/s/va5DgDbcD07DBFRP1D_RSw).
- **2026.04** — Technical Report v1.0 released on [arXiv](https://arxiv.org/abs/2604.17091).

## Highlights

- 🪶 **9 atomic tools, not 50+** — capability through composition (vs. 53 in Claude Code).
- 📉 **~1/6 the token cost** at matched-or-better success (188.8k vs 537.4k on the 5-task long-horizon benchmark).
- 🔁 **Self-evolving** — −89.6% tokens, −78.2% runtime, −84.4% LLM calls over 9 rounds, no prompt tuning.
- 📚 **No vector DB** — beats embedding retrievers (Mem0, A-MEM) on LoCoMo with pure hierarchical memory.
- 🛡️ **No context explosion** — 2,298-token prompt after 20 skills (~10–20× smaller than baselines).
- 🌐 **Web-ready** — 3× the BrowseComp-ZH score at ~1/6 the tokens vs. OpenClaw.
- ⚙️ **Minimal** — 92-line core loop, ~3.3k LoC total (vs. ~530k for OpenClaw).

## How It Works

<p align="center">
  <img src="assets/framework.png" width="90%" alt="GA framework"/>
</p>

GA runs a unified agent loop and maximizes **context information density** across four mechanisms:

| Mechanism | Idea |
|---|---|
| **Minimal Atomic Toolset** | 9 primitives across 5 capability classes; broad capability via composition, not enumeration. |
| **Hierarchical Memory** | 4 layers (L1 index → L2 facts → L3 SOPs → L4 archive); only the compact L1 stays in-prompt, the rest is retrieved on demand. |
| **Self-Evolution** | Reflection compresses verified trajectories into SOPs → executable code; triggered by the memory system, not the user. |
| **Context Compression** | Truncation, periodic tag-level compression, FIFO eviction past 60% budget — targeting a **<30k token** budget. |

**Emergent capabilities** built on top: **subagent dispatch** (GA CLI as a sub-process), **Reflect Mode** (event-driven watchdog + cron-style tasks), and **autonomous exploration** over a persistent skill tree.

## Evaluation Results

GA is evaluated along **five dimensions** against **Claude Code**, **OpenAI Codex**, and **OpenClaw** under *Claude Sonnet 4.6 / Opus 4.6*, *GPT-5.4*, and *MiniMax M2.7* backbones.<sup>†</sup> Full details in the [Technical Report](GA_Technical_Report.pdf).

> <sup>†</sup> *Model version names follow our internal evaluation pipeline; see §4.1.1 of the report for exact backbone configs.*

### 1. Task Completion & Token Efficiency

> *Efficiency = Accuracy / Total Tokens (M).*

| Benchmark | Agent | Backbone | Accuracy | Total Tokens | Efficiency |
|---|---|---|:---:|:---:|:---:|
| **SOP-Bench** | **GA** | Claude Sonnet 4.6 | **100%** | 2.08M | 0.48 |
| | OpenClaw | Claude Sonnet 4.6 | 100% | 2.64M | 0.38 |
| | Claude Code | Claude Sonnet 4.6 | 85% | 1.25M | 0.68 |
| | **GA** | MiniMax M2.7 | 90% | **0.92M** | **0.97** |
| | OpenClaw | MiniMax M2.7 | 95% | 2.96M | 0.32 |
| **Lifelong AgentBench** | **GA** | Claude Sonnet 4.6 | **100%** | **0.24M** | **4.15** |
| | OpenClaw | Claude Sonnet 4.6 | 70% | 1.45M | 0.48 |
| | Claude Code | Claude Sonnet 4.6 | 75% | 0.81M | 0.92 |
| | **GA** | MiniMax M2.7 | **90%** | 0.42M | 2.12 |
| | OpenClaw | MiniMax M2.7 | 70% | 1.22M | 0.57 |
| **RealFin-Benchmark** | **GA** | Claude Sonnet 4.6 | **65%** | **0.11M** | **5.70** |
| | Claude Code | Claude Opus 4.6 | 60% | 0.31M | 1.95 |
| | Claude Code | Claude Sonnet 4.6 | 55% | 0.24M | 2.31 |
| | OpenClaw | Claude Sonnet 4.6 | 35% | 0.25M | 1.39 |
| | Codex | GPT-5.4 | 60% | 0.89M | 0.67 |

> 📌 On Lifelong AgentBench, GA hits **100%** accuracy with **27.7%** of Claude Code's input tokens and **15.5%** of OpenClaw's.

### 2. Tool-Use Efficiency

GA exposes **9 atomic tools**, vs. **53** in Claude Code (20 base + 33 conditional) and **18** factories in OpenClaw.

**Long-horizon complex tasks (5 tasks, Claude Sonnet 4.6):**

| Agent | Success | Total Tokens | Time (s) | Requests | Tool Calls |
|---|:---:|:---:|:---:|:---:|:---:|
| **GA** | **100.0%** | **188,829** | 220.8 | **11.0** | **12.8** |
| Claude Code | 100.0% | 537,413 | 320.8 | 32.6 | 22.6 |
| OpenClaw | 80.0% | 633,101 | 183.1 | 15.0 | 16.6 |

> 📌 GA matches Claude Code's success while using **35.1%** of its tokens, **33.7%** of its requests, **56.6%** of its tool calls.

<p align="center">
  <img src="assets/result_radar.png" width="90%" alt="Tool-use efficiency radar"/><br/>
  <sub>Tool-use efficiency radar — GA leads on token, request, and tool-call axes while preserving quality.</sub>
</p>

### 3. Memory System Effectiveness

**(a) Condensed memory ablation — SOP-Bench (dangerous goods):**

| Configuration | Memory Size (tokens) | Task Success Rate |
|---|:---:|:---:|
| No-Memory | 0 | 13.87% |
| Full-Memory | 575 | 52.44% |
| Redundant-Memory | 288 | 66.48% |
| **Condensed Memory** | **165** | **66.48%** |

**(b) Long-term factual memory — LoCoMo (no embedding model, no vector DB):**

| System | Multi-Hop F1 | Temporal F1 | Open-Domain F1 | Single-Hop F1 |
|---|:---:|:---:|:---:|:---:|
| Mem0 | 39.32 | 50.03 | 18.32 | 40.32 |
| A-MEM | 29.03 | 46.83 | 13.11 | 44.68 |
| OpenClaw | 21.43 | 22.56 | 9.56 | 23.44 |
| **GA** | **43.33** | **52.23** | **20.41** | **45.69** |

**(c) Context-explosion stress test — full prompt length after installing 20 skills:**

| System | Full Prompt Length |
|---|:---:|
| Claude Code | 22,821 tokens |
| Codex | 23,932 tokens |
| OpenClaw | 43,321 tokens |
| **GA** | **2,298 tokens** |

> 📌 GA's prompt stays **~10–20× smaller** after extensive skill expansion — idle memory is kept strictly off-prompt.

### 4. Self-Evolution Capability

**Nine-round longitudinal study (LangChain GitHub research task).** Stage transitions are triggered autonomously by the memory system.

| Round | Stage | Time | LLM Calls | Total Tokens |
|:---:|---|:---:|:---:|:---:|
| #1 | Initial run | 7m30s | 32 | 222,203 |
| #2 | SOP optimization | 4m19s | 12 | 66,341 |
| #3 | SOP optimization | 2m53s | 8 | 49,825 |
| #4 | SOP optimization | 2m29s | 9 | 51,758 |
| #5 | SOP optimization | 2m50s | 7 | 35,536 |
| #6 | Codified SOP | 2m24s | 6 | 25,762 |
| #7 | Codified SOP | 1m41s | 5 | 23,014 |
| #8 | Codified SOP | 1m35s | 5 | 22,689 |
| **#9** | **Codified SOP** | **1m38s** | **5** | **23,010** |

> 📌 Round #1 → #9: **−89.6%** tokens, **−78.2%** runtime, **−84.4%** LLM calls — savings come from removing whole reasoning loops, not just shorter responses.

Across **8 web tasks** (3 runs each), GA cuts tokens by **61.0%–92.4%** (overall **−79.3%**); OpenClaw shows no comparable convergence.

<p align="center">
  <img src="assets/result_convergence.png" width="90%" alt="Cross-task self-evolution convergence"/><br/>
  <sub>GA converges to a stable low-cost regime across eight web tasks; OpenClaw does not.</sub>
</p>

### 5. Web Browsing Capability

Both GA and OpenClaw use **Claude Opus 4.6**.

| Benchmark | # Tasks | Eval Protocol | GA Score | OpenClaw Score | GA Tokens | OpenClaw Tokens |
|---|:---:|---|:---:|:---:|:---:|:---:|
| **WebCanvas** | 12 | Automatic | **0.834** | 0.722 | **0.18M** | 0.71M |
| **BrowseComp-ZH** | 10 | LLM-as-Judge | **0.600** | 0.200 | **0.47M** | 1.31M |
| **Custom Tasks** | 22 | Human + LLM | **0.577** | 0.500 | **0.26M** | 0.76M |

> 📌 On multi-hop **BrowseComp-ZH**, GA **triples** OpenClaw's score (0.60 vs. 0.20) at **~1/6** the tokens; **2.9×–3.9×** token reduction across all three benchmarks.

## Repository Layout

```
GA-Technical-Report/
├── GA_Technical_Report.pdf       # Full technical report
├── assets/                       # Figures
└── datasets/                     # Evaluation datasets (each with its own README)
    ├── lifelong_agentbench/
    ├── locomo/
    ├── realfin_benchmark/
    ├── sop_bench/
    ├── tool_efficiency_benchmark/
    └── web_browsing/
```

Each benchmark folder ships its own README with task setup, evaluation protocol, and reproduction notes.

## Citation

```bibtex
@misc{liang2026genericagenttokenefficientselfevolvingllm,
      title={GenericAgent: A Token-Efficient Self-Evolving LLM Agent via Contextual Information Density Maximization (V1.0)}, 
      author={Jiaqing Liang and Jinyi Han and Weijia Li and Xinyi Wang and Zhoujia Zhang and Zishang Jiang and Ying Liao and Tingyun Li and Ying Huang and Hao Shen and Hanyu Wu and Fang Guo and Keyi Wang and Zhonghua Hong and Zhiyu Lu and Lipeng Ma and Sihang Jiang and Yanghua Xiao},
      year={2026},
      eprint={2604.17091},
      archivePrefix={arXiv},
      primaryClass={cs.CL},
      url={https://arxiv.org/abs/2604.17091}, 
}
```

<div align="center">
  <sub>© 2026 Advantage AI Agent Lab (A³ LAB). Released alongside the GenericAgent system at <a href="https://github.com/lsdefine/GenericAgent">github.com/lsdefine/GenericAgent</a>.</sub>
</div>