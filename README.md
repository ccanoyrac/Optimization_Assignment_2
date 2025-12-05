# README – Stochastic Fuel Procurement and Power Generation Optimization
A Multi‑Stage, Multi‑Scenario, Risk‑Aware Energy Procurement Model (Drafts 1 → 3)

This project develops a progressively more sophisticated optimization framework for fuel procurement and electricity generation planning across Oil, Gas, and Coal technologies. The work evolves across **three major drafts**, each adding realism and complexity.

---

## 1. Project Overview

The goal is to determine **optimal daily fuel purchases** and **hourly generation schedules** across multiple conventional technologies, while considering:

- Daily auctions for fuel (Oil, Gas, Coal)
- Hourly electricity market prices
- Storage dynamics and costs
- Volatility of commodity markets
- Multi‑scenario stochastic behavior
- **Risk‑aversion using CVaR**
- Technology‑specific penalty factors (e.g., emissions)
- Elastic procurement decisions (shadow-price interpretation)

The project is implemented in **Python**, using **Gurobi** for optimization and a **JSON-based data structure** for modularity.

---

# ---------------------------------------------
# 2. Evolution of the Model (Draft 1 → Draft 3)
# ---------------------------------------------

The README now explicitly explains **all three drafts** and how the model evolves.

---

## Draft 1 — Deterministic Cost Minimization  
The starting point is a simple, fully deterministic optimization problem:

**Objective:**  
Minimize *total weekly cost* of procuring fuel to satisfy a fixed electricity demand.

### Features
- 3 technologies: Oil, Gas, Coal  
- Fixed weekly demand (e.g., 100 MWh)  
- Daily procurement decision  
- No uncertainty  
- Simple storage model  
- No revenue component  

### Purpose
This draft introduces:
- Basic formulation
- Constraints (generation limit, balance, storage)
- Gurobi workflow

It serves as the foundation for later stochastic and profit‑maximizing versions.

---

## Draft 2 — Profit Maximization with Daily Auctions and Hourly Prices  
The model becomes **market-driven**:

**Objective:**  
Maximize profit = electricity revenue − fuel costs − storage costs − penalties.

### Added features
- Hourly electricity demand using an hourly shape factor  
- Hourly electricity market prices  
- Technology-specific revenue penalties (emissions factors)  
- Daily fuel auctions per technology  
- Volatile daily fuel prices  
- Storage penalty per day  
- Multiple suppliers sorted by cost  
- JSON-based modular data generation

### Purpose
Draft 2 introduces:
- Multi-time-scale structure (daily purchase → hourly generation)
- Revenue maximization
- Multiple suppliers and auctions
- Data-driven approach

It is still **deterministic**, but is nearly ready for multi-scenario stochastic modeling.

---

## Draft 3 — Full Stochastic Optimization with CVaR and Technology-Specific Scenarios  
This is the **final and complete model**, integrating:

### ✔ Multi-scenario stochasticity  
Fuel prices vary by:
- Day type (Base / Spike / Severe)
- Technology (Oil/Gas/Coal)
- Day of week

Scenarios consist of sequences such as:  
```
Base — Base — Spike — Severe — Severe — Base — Base
```

### ✔ Hourly electricity prices also stochastic  
Created with volatility profiles analogous to fuel markets.

### ✔ CVaR-based risk aversion  
The objective becomes:

$$
\max_{x} \; \mathbb{E}[\text{profit}(x,\omega)] - \lambda \cdot \text{CVaR}_{\alpha}(\text{Loss})
$$


Where:
- Loss = max(0, target − profit)
- η and u(w) enforce CVaR linearization
- λ controls conservativeness  

### ✔ First-stage vs second-stage separation  
- Fuel purchase = first stage (same for all scenarios)
- Generation = second stage (scenario-dependent)

This allows procurement decisions to hedge against future volatility.

### ✔ Shadow prices → “Elastic demand price”
Dual variables of fuel‑availability constraints compute the *optimal willingness to pay* for fuel under all risks.

### Purpose
Draft 3 adds:
- Scenario tree
- Stochastic modeling
- Advanced risk modeling
- Economic interpretation

It is the final version implemented in this repository.

---

# ---------------------------------------------
# 3. Mathematical Model (Draft 3 Summary)
# ---------------------------------------------

## 3.1 Decision Variables

### First-stage (same across all scenarios)
- `purchase[t,s,d]` — MWh fuel purchased  
- `fuel_available[t,d]` — total available fuel  
- `eta` — VaR threshold  

### Second-stage (scenario dependent)
- `generate[t,d,h,w]`  
- `inventory[t,d,w]`  
- `u[w]` — CVaR linear slack  
- `L[w]` — shortfall / loss variables  
- `under_generation[...]`, `over_generation[...]`

---

# 4. Scenario Construction  
Scenarios represent many possible weekly price paths.

### For each scenario:
- Randomly construct a 7-day sequence  
- For each tech (Oil/Gas/Coal), each day is classified as:
  - Base  
  - Spike  
  - Severe  

Each class modifies prices using a multiplier distribution:
- Base: mild noise  
- Spike: rare, high volatility  
- Severe: extreme jumps  

Scenario probabilities are proportional to their likelihood.

---

# 5. Risk Modeling with CVaR

The model uses **Conditional Value at Risk**:

### Loss definition:
$$
L(w) \ge 0,\quad  L(w) \ge 	{target\_profit} - 	{profit}(w)
$$

### CVaR linearization:
$$
u(w) \ge L(w) - \eta
$$

### Final CVaR term:
$$
	{CVaR}_{\alpha} = \eta + \frac{1}{1-\alpha} \sum_w P(w) \, u(w)
$$

### Final objective:
$$
\max \; \mathbb{E}[{profit}] - \lambda \cdot 	{CVaR}_{\alpha}
$$

- λ = 0 → risk-neutral  
- λ large → highly conservative procurement

---

# 6. Repository Structure

```
/
│── data/
│     ├── data_draft_1.json
│     ├── data_draft_2.json
│     ├── data_draft_3.json
│
│── plots/
│     ├── ...
│
│── Assignment_2.ipynb
│
│── Assignment_2.pdf
│
│── README.md
```

---

# 7. How to Run

### 1. Install dependencies
```
pip install gurobipy numpy pandas
```

### 2. Ensure Gurobi license is active.

### 3. Run data generation
```
first cell of requirements and the data creation cells (only necessary if you dont have the data already)
```

### 4. Run optimization
```
run the python cells with the optimization
```

### 5. Run visualization cells
```
run the python cells with the optimization results to see the charts (this cells are right under the optimization)
```

---

# 8. Key Insights from the Final Model

- Severe volatility scenarios encourage **buying fuel earlier**.  
- Coal remains dominant due to low volatility and low price.  
- Gas becomes the flexible mid-merit technology.  
- Oil is purchased only when profitable relative to its emission penalty.  
- CVaR significantly reshapes procurement:
  - Risk-averse solutions stock up earlier.
  - Shadow prices rise (higher willingness-to-pay).
- The model produces economically interpretable **elastic procurement prices**.

---

# 9. Future Extensions

- Multi-week horizon  
- Ramping constraints  
- Start-up/shutdown costs  
- CO₂ markets and allowance pricing  
- Renewable generation + storage  
- Scenario reduction (fast SAA)  

---

