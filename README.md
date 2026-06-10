# AI-Assisted Quantum Circuit Transpilation. Talk & Hands-On Demos

> Companion repository for a 40-minute talk on **quantum circuit transpilation** and how
> **AI (reinforcement learning)** is starting to do it better. Includes two hands-on,
> fully-runnable Jupyter notebooks.
>
> **Speaker:** Duru Tomruk · Bachelor thesis at **IBM Quantum** × **DHBW Stuttgart**
> *Thesis: "Improving AI-Assisted Quantum Circuit Transpilation for Square-Lattice QPUs"*

---

## TL;DR. You don't have to run anything

Both notebooks are **pre-executed**: every circuit drawing, number, and chart is already
saved inside the `.ipynb`. **Just open them on GitHub (or in VS Code) and read top to bottom.**
If you *want* to run them yourself, see [Setup](#setup-optional). They run **100% offline**,
with no IBM Cloud account and no API token.

| Notebook | What it shows | Time |
|---|---|---|
| [`01_classical_transpiler_bell.ipynb`](01_classical_transpiler_bell.ipynb) | The **classical** Qiskit transpiler, walked one step at a time on a **Bell state**, through all **6 stages** (init → layout → routing → translation → optimization → scheduling). Includes the "routing game": why limited qubit connectivity is expensive. | ~10 min |
| [`02_ai_transpiler.ipynb`](02_ai_transpiler.ipynb) | The **AI** transpiler. Frames transpilation as a **game** a reinforcement-learning agent plays, then shows the **real measured results** from the thesis: classical vs. heavy-hex-AI vs. square-lattice-AI. | ~8 min |

---

## What is transpilation? (the 60-second version)

A quantum chip can only do two things your textbook circuit ignores:

1. **It speaks a tiny fixed vocabulary**, a handful of *native gates*. Your `H` and `CNOT`
   aren't on the menu and must be rewritten into what is.
2. **Its qubits are wired in a fixed map**. Most pairs of qubits have **no direct connection**.
   A two-qubit gate can only run on a pair that's physically wired together.

**Transpilation** is the compiler step that rewrites your circuit to obey both rules.
It does this *without changing the computation*. The best everyday analogy: it's a **GPS** that reroutes
your straight-line request onto the roads that actually exist, in the turn-by-turn language the
car understands, as short as possible (because every extra gate adds error).

When two qubits that must interact aren't neighbours, the transpiler inserts **SWAP** gates to
slide them together. And **each SWAP costs three two-qubit gates** on real hardware. Minimising
those SWAPs is the heart of the problem. It is exactly where AI helps.

---

## The thesis in one slide

Reinforcement-learning (RL) transpilers have been shown to beat the classical compiler. But this works
**only on IBM's old "heavy-hex" chip shape** (each qubit has 2-3 neighbours). IBM's new flagship,
the 120-qubit **Nighthawk**, uses a **square lattice** (each interior qubit has **4** neighbours).
Does the AI advantage carry over to the new geometry?

**Research question:** *"Can RL-based synthesis be improved by training on subgraphs of a target topology?"*

**Method (subgraph extraction):** chop the square-lattice chip into every distinct small wiring
pattern ("subgraph", 2-8 qubits), find the **69** patterns the existing heavy-hex AI never saw,
and train a small RL agent (PPO) for each. The result is **SQR**, a square-lattice-trained transpiler.

**Three transpilers, same circuits:**

| | what it is |
|---|---|
| **QPM** | classical Qiskit Pass Manager (today's default) |
| **QTS** | IBM's AI transpiler, trained on **heavy-hex** |
| **SQR** | this thesis's AI, retrained for the **square lattice** |

**Headline results**

- **Up to ~46% shallower two-qubit-gate depth** on *structured* circuits with contiguous CNOT
  blocks (e.g. Bernstein-Vazirani: **429 → 234** CZ gates, −45.5%; EfficientSU2-89: depth −45.9%).
- **SQR beats the heavy-hex AI (QTS)** on its home turf by **13-28%** gate count / **13-24%** depth.
- The square lattice *itself* already gives **~30-45% lower** 2Q depth than heavy-hex for most circuits.
- **Honest limit:** the advantage is topology-specific. On an *unseen* heavy-hex chip it collapses
  to ~0-6%. Retraining per topology is a one-time cost that amortises over high-volume workflows.

**When does the AI help?** You can predict it from the circuit's structure *before* compiling:

| Regime | Circuit structure | Outcome |
|---|---|---|
| 🟢 **Better** | contiguous CNOT-only blocks (BV, SU2-89, QV, QFT, QAOA) | SQR wins, up to ~46% |
| ⚪ **Matches** | already fits the grid (SU2-100, Heisenberg) | all three tie |
| 🔴 **Worse** | routing/SWAP-dominated (dense Clifford) | within ~5%, AI ≤ classical |

📦 **Full artifacts** (benchmark framework + the 69 trained models + the retraining pipeline):
**https://github.com/DuruTmr444/squarebench**

---

## Setup (optional)

You only need this if you want to *run* the notebooks rather than just read them.

```bash
# 1. Create an environment (Python 3.10-3.12 recommended)
conda create -n transpile-demo python=3.12 -y
conda activate transpile-demo

# 2. Install the (small) dependency set
pip install -r requirements.txt

# 3. Launch
jupyter lab          # or: open the folder in VS Code and pick this kernel
```

Then open either notebook and **Run All**. Everything is deterministic (fixed seeds) and runs
offline. The "fake" IBM backends (`FakeTorino`, `FakeMarrakesh`) are frozen local snapshots of
real machines (same wiring map, native gates, and error rates), so **no cloud and no token** is
ever needed.

---

## Repository layout

```
.
├── 01_classical_transpiler_bell.ipynb   # Demo 1: classical transpiler, 6 stages on a Bell state
├── 02_ai_transpiler.ipynb               # Demo 2: AI transpiler: the game + the thesis results
├── Ablaufplan_DE.pdf · RUNDOWN_DE.md    # one-page German run-of-show (presenter cheat sheet)
├── assets/                              # figures & animations the notebooks embed
├── slides/Transpilation.pptx            # the talk deck (lean, 15 slides)
├── slides/Transpilation_full.pptx       # full 42-slide version (all details, backup)
├── requirements.txt
└── README.md
```

---

## Credits & license

- The presentation extends the IBM Quantum **"Circuit Transpilation"** deck; original template and
  foundational slides by **Junye Huang (IBM Quantum)**, who also supervised the thesis.
- Thesis supervised by **Junye Huang** (IBM) and **Prof. Dr. Gerhard Hellstern** (DHBW Stuttgart).
- Built on open tools: [Qiskit](https://www.ibm.com/quantum/qiskit),
  the [Qiskit AI Transpiler passes](https://docs.quantum.ibm.com/guides/ai-transpiler-passes), and
  the [Benchpress](https://github.com/Qiskit/benchpress) benchmark suite (Nation et al., 2025).

Notebooks and slides in this repository are released for educational use under the **MIT License**
(see `LICENSE`). The thesis text itself is not included here.

---

*Slides are in English; the talk is presented in German. Questions welcome. Code and trained models
live at [squarebench](https://github.com/DuruTmr444/squarebench).*
