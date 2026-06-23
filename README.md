# Quantum Subset Sum Demo

Educational Qiskit demos for solving small Subset Sum instances with Grover's
algorithm.

（It isn't fully complete yet）

## Problem and Method

Given positive integers $a_0, a_1, \ldots, a_{n-1}$ and a target integer $T$,
the Subset Sum problem asks for a selector bitstring $x \in \{0,1\}^n$ such
that:

$$
\sum_i x_i a_i = T
$$

The selector bit `x_i` means whether the number `a_i` is included in the
subset. Each basis state $|x\rangle$ represents one candidate subset.

Grover's algorithm searches over these candidate subsets as follows.

Step 1: prepare a uniform superposition over all $N = 2^n$ candidates:

$$
H^{\otimes n}|0^n\rangle = \frac{1}{\sqrt{N}} \sum_{x \in \{0,1\}^n}|x\rangle = |\varphi\rangle
$$

Step 2: use a phase oracle to mark the desired state. In the simplest case,
for a target solution $s$, the oracle acts as:

$$
U_s|x\rangle =
\begin{cases}
-|x\rangle, & x=s,\\
|x\rangle, & x\neq s.
\end{cases}
$$

For Subset Sum, the oracle marks every selector bitstring whose weighted sum is
exactly the target $T$:

$$
\sum_i x_i a_i = T
$$

Step 3: apply the diffusion operator, a reflection around the uniform state:

$$
U_d = 2|\varphi\rangle\langle\varphi| - I
$$

Step 4: repeat the oracle and diffusion operators. If there are $M$ valid
solutions among $N$ candidates, a standard iteration estimate is:

$$
r = \left\lfloor \frac{\pi}{4}\sqrt{\frac{N}{M}} \right\rfloor
$$

## Repository Overview

The main implementation uses Qiskit's `WeightedAdder` to compute the selected
subset sum inside the oracle. The notebook is the best starting point for
reading the workflow, and the Python file keeps the same idea in a reusable
function. The circuit prepares selector qubits in superposition, computes the
weighted sum, phase-flips states whose sum equals the target, uncomputes the
adder workspace, and applies the diffuser to the selector qubits.

- `subset_sum_grover_weighted_adder.ipynb` explains the weighted-adder Grover
  construction step by step. The crucial step is using Qiskit's
  `WeightedAdder` library circuit to perform the reversible weighted-sum
  computation. This keeps the main demo compact, but the adder itself is mostly
  treated as a library-level black box.
- `quantum_subset_sum_weighted_adder.py` provides the reusable
  `quantum_subset_sum(xs, target)` function and a quick command-line check.
- `subset_sum_grover_based_on_paper.ipynb` follows the lower-level construction
  from https://arxiv.org/pdf/2410.01775 and uses the public repository
  `ABenoit0226/quantum-place-route` as a code reference. This notebook explores
  how the subset-sum summation is carried out at the circuit level instead of
  relying on Qiskit's `WeightedAdder` helper. Different adder/oracle designs can
  lead to different circuit sizes and depths, which is why the paper focuses on
  optimization.
- `grover_demo.ipynb` is a smaller introductory Grover / subset-sum notebook.

## Files

- `subset_sum_grover_weighted_adder.ipynb` - notebook using
  Qiskit's `WeightedAdder` to compute the weighted sum.
- `quantum_subset_sum_weighted_adder.py` - reusable Python implementation of
  the weighted-adder Grover approach.
- `subset_sum_grover_based_on_paper.ipynb` - notebook sketch following the
  procedure described in the paper at https://arxiv.org/pdf/2410.01775.
- `grover_demo.ipynb` - introductory Grover / subset-sum notebook.

## Install

```bash
python -m pip install -r requirements.txt
```

## Run the Notebook Demos

Recommended starting point:

```bash
jupyter notebook subset_sum_grover_weighted_adder.ipynb
```

Notebook based on the paper:

```bash
jupyter notebook subset_sum_grover_based_on_paper.ipynb
```

Introductory notebook:

```bash
jupyter notebook grover_demo.ipynb
```

## Python Function and Quick Check

The reusable entry point is `quantum_subset_sum(xs, target)`. It takes a finite
list of positive integers and a positive target, then returns a subset of
indices whose selected values sum to the target. This matches the expected
project interface: return a valid index subset, or report that no solution was
found for the tested small instance.

```python
from quantum_subset_sum_weighted_adder import quantum_subset_sum

result = quantum_subset_sum([1, 2, 3], 3, verbose=True)
print(result)
```

The returned result includes the subset indices, selector bits, selected total,
Statevector probability for the chosen bitstring, status, and Grover iteration
count.

The same implementation can be run as a quick command-line check:

```bash
python quantum_subset_sum_weighted_adder.py
```

## Notes

The `WeightedAdder` version is the main working implementation. The notebook
based on the paper/repository is a study track for building the oracle at a
lower level.

## TODO

- Finish the notebook based on the paper/repository implementation.
- Add a phase-estimation or quantum-phase-transition based variant.
- Add more benchmark cases for small inputs.
- Compare the `WeightedAdder` version with a lower-level reversible-arithmetic
  oracle.



