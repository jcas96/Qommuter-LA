# Qommuter LA

**Quantum optimization for equitable bus investment in Los Angeles**

🏆 **1st Place: Qompute 2026 (Qiskit Fall Fest 2026, hosted by QEE), Track 2: Optimizing for People**

> **The question behind the project:** when a city changes what "optimal" means, which communities get the investment?

---

## What it does

Qommuter LA models a realistic transit-funding decision. A city has a one-time fund that can upgrade **exactly 3 of 6 LA Metro bus routes**. Each candidate route is scored on:

- **Efficiency:** observed average weekday boardings (LA Metro, Q2 2024)
- **Equity:** how much the route serves communities with high transportation need
- **Overlap:** how much two routes cover the same corridor, so the model doesn't fund two lines that serve the same riders

A single slider, the equity weight **α**, sets how much the city values equity relative to ridership. The project turns this decision into a quantum optimization problem, solves it with **QAOA (Quantum Approximate Optimization Algorithm)**, and checks the result against an exact classical solver.

### Candidate routes

| Line | Corridor |
|---|---|
| 4 | Santa Monica – Downtown LA via Santa Monica Blvd |
| 18 | Wilshire/Western – Montebello via 6th/Whittier |
| 33 | Santa Monica – Downtown LA via Venice Blvd |
| 40 | South Bay – Downtown LA via Hawthorne/MLK |
| 51 | CSU Dominguez Hills – Westlake via Avalon/San Pedro |
| 204 | Athens – Hollywood via Vermont Ave |

---

## Problems solved

1. **Ranking routes misses how they interact.** Picking the top three routes by score ignores overlap: two strong routes on the same corridor can deliver less combined coverage than two complementary ones. The model adds a pairwise overlap penalty, which turns the task into a portfolio-selection problem.
2. **"Optimal" usually hides a value judgment.** The equity weight α is an explicit, adjustable parameter. Sweeping α from 0 (ridership only) to 1 (equity only) shows exactly where the recommended set of routes changes, so decision-makers can see the trade-off instead of receiving one unexplained answer.
3. **A civic decision needs a quantum-ready form.** The objective is converted into a **QUBO** and then an **Ising Hamiltonian**. Every coefficient is derived by hand and tested.
4. **Quantum results need to be trustworthy.** QAOA output is never taken at face value. It is compared against exhaustive brute-force search, and the notebook reports feasibility probability, probability of the exact optimum, and variation across random starts.
5. **Constraint enforcement needs tuning.** A penalty sweep finds the smallest penalty that guarantees the "exactly 3 routes" constraint, so the chosen penalty is justified by a test rather than guessed.

---

## What we accomplished

- **Formulated the allocation problem** as an objective function, then as a QUBO and a 6-qubit Ising Hamiltonian (1 constant term, 6 Z terms, 15 ZZ terms).
- **Verified the encoding across all 64 basis states.** The direct objective, the QUBO and the Ising energies agree to within 1e-10, which rules out sign, offset and qubit-order mistakes.
- **Found the exact optimum** by enumerating all C(6,3) = 20 valid portfolios. With α = 0.55 the best choice is **Lines 18, 51 and 204**.
- **Ran QAOA at depth p = 3** across 12 random starts with a COBYLA optimizer, sampled 4,096 shots, and compared the most common valid portfolio with the exact answer.
- **Built two QAOA paths:**
  - a Qiskit implementation using `qaoa_ansatz`, `SparsePauliOp` and `StatevectorEstimator`
  - a fully transparent NumPy statevector implementation that shows every step of the algorithm
- **Ran robustness experiments:**
  - **Circuit depth:** compared p = 1, 2 and 3 across random starts
  - **Equity weight:** swept α to map where the recommended routes change
  - **Overlap ablation:** tested whether removing the overlap term changes the recommendation
- **Wrote an equity and governance review:** a table of risks such as historic-ridership bias, ecological fallacy, ACS uncertainty and displacement, with a mitigation for each, plus governance commitments for any real-world use.

---

## Tech stack

| Area | Tools |
|---|---|
| Quantum computing | **Qiskit** (`qaoa_ansatz`, `SparsePauliOp`, `StatevectorEstimator`, `Statevector`), Qiskit Aer |
| Optimization | SciPy (COBYLA), exhaustive classical search |
| Data and numerics | Python, NumPy, pandas |
| Visualization | Matplotlib |
| Environment | Jupyter Notebook |
| Data sources | LA Metro line-level ridership, LA Metro Bus GTFS, LA Metro 2025 Equity Need Index, U.S. Census ACS 5-Year |

---

## How the model works

For each route *i*, the decision variable *xᵢ* is 1 if the route is funded and 0 if not.

**Route value** blends ridership *Rᵢ* and equity *Eᵢ*:

```
uᵢ(α) = (1 − α)·Rᵢ + α·Eᵢ
```

**Portfolio benefit** adds up the route values and subtracts a penalty for overlapping coverage:

```
F(x) = Σ uᵢ·xᵢ  −  ρ · Σ_{i<j} Oᵢⱼ·xᵢ·xⱼ
```

**Cost minimized by QAOA** adds a penalty that enforces exactly K = 3 routes:

```
C(x) = −F(x) + P·(Σ xᵢ − K)²
```

The parameters used in the main scenario are:

| Parameter | Value | Meaning |
|---|---|---|
| α | 0.55 | Equity weight |
| ρ | 0.35 | Overlap penalty |
| P | 1.5 | Constraint penalty |
| K | 3 | Routes funded |

---

## Running it

```bash
git clone https://github.com/jcas96/Qommuter-LA.git
cd Qommuter-LA
pip install -r requirements.txt
jupyter notebook QommuterLA.ipynb
```

The notebook's first code cell pins the Qiskit versions it was tested with (`qiskit==2.5.2`, `qiskit-aer==0.17.2`). If Qiskit isn't installed, the notebook falls back to the NumPy reference implementation, which runs the same QAOA math.

---

## Limitations and responsible use

- **This is not a claim of quantum advantage.** With 6 routes, brute force is faster. The value of the project is a correct, testable quantum formulation of a civic problem.
- **The ridership data is real, but the equity and overlap inputs are prototype values.** These inputs are labeled in the notebook. A production version would compute them with a GTFS × Metro Equity Need Index spatial join and Jaccard similarity between route catchments.
- **This is an educational prototype,** not a recommendation to change LA Metro service.

---

## Acknowledgments

- The structure of the notebook is adapted from the Qompute 2026 **Track 2 Challenge 2 Tutorial** by Benjamin Tirado / BasQ (narrative under CC BY 4.0, code under Apache 2.0).
- All LA-specific modeling, validation, experiments and impact analysis were developed for Qommuter LA.
- Full source citations are listed at the end of the notebook.

## License

See [LICENSE](LICENSE).
