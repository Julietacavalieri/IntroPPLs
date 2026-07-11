# Mini-PPL with Python dataclasses

Final project for the course **Introduction to Probabilistic Programming**, taught by **Javier Burroni** at **FCEyN, Universidad de Buenos Aires** in 2026.

Authors: **Julieta Cavalieri** and **Ema Sapirstein**.

## Project overview

This repository reimplements the mini probabilistic programming language used in the June 26 class of the course. The original class notebook is `activity_5_messaging_student.ipynb`, which is included here as reference. Our rewritten implementation is in `ppl_rewrite_dataclasses.ipynb`. We chose to keep Python, but changed the structure to a more object-oriented design based on `dataclasses`.

The original implementation uses a message-interface runtime: the program runs normally until it reaches a probabilistic operation such as `sample` or `observe`. At that point, execution pauses and returns a message. An inference controller handles that message and sends a value back so the machine can continue.

Our rewrite keeps that same idea, but makes the components of the runtime more explicit. Parsed programs from the course parser are converted into dataclass-based AST nodes. Continuations, closures, messages, and the execution machine are also represented with dataclasses.

This makes the separation between the program runtime and the inference algorithms clearer. The same runtime can be executed with different controllers: Likelihood Weighting, Sequential Monte Carlo, and Single-site Metropolis-Hastings.


## Repository structure

```
.
├── README.md
├── activity_5_messaging_student.ipynb
├── ppl_rewrite_dataclasses.ipynb
└── requirements.txt
```

- `activity_5_messaging_student.ipynb`: original notebook from the June 26 class, included only as reference.

[![Open In Colaboratory](https://google.com)](https://google.com/Julietacavalieri/IntroPPLs/blob/main/activity_5_messaging_student.ipynb)

- `ppl_rewrite_dataclasses.ipynb`: main project notebook. This is the file to read and run.

[![Open In Colaboratory](https://google.com)](https://google.com/Julietacavalieri/IntroPPLs/blob/main/ppl_rewrite_dataclasses.ipynb)

- `requirements.txt`: minimal Python dependencies for local execution.

## Optional improvements added

Besides rewriting the runtime using dataclasses, we added two optional improvements.

### 1. Method-based machine interface

The original implementation used standalone functions such as:

```python
msg = resume(m)
send(m, value)
```

In our rewrite, the machine also exposes these operations as methods:

```python
msg = m.resume()
m.send(value)
```

Before moving `resume` into the machine class, we refactored the original long `resume(m)` function into a more object-oriented structure. Instead of handling all expression and continuation cases inside one large function, each dataclass now implements the behavior that belongs to it. Expressions define how they are evaluated, and continuations define how they continue the computation.

This does not change the semantics of the interpreter or the inference algorithms. It only makes the runtime organization more object-oriented. The standalone functions `resume(m)` and `send(m, value)` are kept as wrappers for compatibility with the original notebook style.

### 2. Strict validation in SMC

The original SMC controller checked whether all particles had reached an `ObserveMsg`, but it did not check whether they had reached the same observe site.

This can be a problem in programs with stochastic control flow: different particles may stop at different observation addresses. Resampling them together would be incorrect, because SMC assumes that particles are synchronized at the same observation point.

To make this assumption explicit, we added a validation step in `run_smc` that checks:

- whether all particles reached the same kind of breakpoint,
- whether all particles reached `Done`, or
- whether all particles reached an `ObserveMsg` with the same address.

If particles are not synchronized, the implementation raises a clear error instead of silently continuing.

## How to run

### Option 1: Google Colab

The easiest way to run the project is with Google Colab.

1. Open Google Colab.
2. Upload `ppl_rewrite_dataclasses.ipynb`.
3. Run all cells from top to bottom.

The notebook clones the course repository in its first cells in order to use the original parser and primitive definitions.

### Option 2: Local Jupyter

You can also run the notebook locally.

Clone the repository:

```bash
git clone https://github.com/Julietacavalieri/IntroPPLs.git
cd IntroPPLs
```

Install the dependencies:

```bash
pip install -r requirements.txt
```

Then open the notebook:

```bash
jupyter notebook ppl_rewrite_dataclasses.ipynb
```

or:

```bash
jupyter lab ppl_rewrite_dataclasses.ipynb
```

