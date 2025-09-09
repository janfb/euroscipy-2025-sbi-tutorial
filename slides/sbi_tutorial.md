---
marp: true
title: "Beyond Likelihoods: Bayesian Parameter Inference for Black-Box Simulators with sbi"
theme: default
paginate: true
backgroundColor: #ffffff
header: '&nbsp;'
footer: '&nbsp;'
style: |
  /* Global slide styling */
  section {
    font-family: 'Helvetica Neue', Arial, sans-serif;
    background-color: #ffffff;
    padding: 80px 50px 50px 50px;
  }

  /* Dark blue stripe at the top - using header element */
  header {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    height: 70px;
    background-color: #1e5266;
    border: none;
    margin: 0;
    padding: 0;
  }

  /* Logo in header */
  header::after {
    content: '';
    position: absolute;
    top: 50%;
    right: -0px;
    transform: translateY(-50%);
    width: 220px;
    height: 300px;
    background-image: url('./images/aai_logo.svg');
    background-size: contain;
    background-repeat: no-repeat;
    background-position: center;
  }

  /* Page number positioning - bottom right */
  section::after {
    position: absolute;
    bottom: 10px;
    right: 30px;
    color: #7f8c8d;
    font-size: 20px;
    font-weight: 500;
  }

  .columns {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 30px;
  }

  img[alt~="center"] {
    display: block;
    margin: 0 auto;
  }

  /* Headings styling - unified theme color */
  h1 {
    font-size: 36px;
    color: #1e5266;
    font-weight: 700;
    margin-bottom: 0.5em;
  }

  h2 {
    font-size: 28px;
    color: #1e5266;
    font-weight: 600;
    margin-bottom: 0.5em;
  }

  h3 {
    font-size: 26px;
    color: #1e5266;
    margin-bottom: 0.5em;
  }

  /* Link styling */
  a {
    color: #56B4E9;
    text-decoration: none;
    border-bottom: 1px solid transparent;
    transition: border-bottom 0.2s;
  }

  a:hover {
    border-bottom: 1px solid #0066cc;
  }

  /* Keep existing code and list styles */
  code {
    font-size: 16px;
    background: #f5f5f5;
    padding: 2px 4px;
    border-radius: 4px;
  }

  pre code {
    font-size: 14px;
    line-height: 1.4;
  }

  ul {
    text-align: left;
  }

  li {
    margin-bottom: 0.5em;
    font-size: 26px;
  }

  strong {
    color: #1e5266;
    font-weight: 700;
  }

  table {
    font-size: 20px;
    margin: 0 auto;
  }

  blockquote {
    border-left: 4px solid #1e5266;
    padding-left: 20px;
    font-style: italic;
    color: #555;
  }

  /* Keep color-blind friendly palette (Okabe-Ito) */
  .cb-orange { color: #E69F00; }
  .cb-skyblue { color: #56B4E9; }
  .cb-green { color: #009E73; }
  .cb-yellow { color: #F0E442; }
  .cb-blue { color: #0072B2; }
  .cb-red { color: #D55E00; }
  .cb-purple { color: #CC79A7; }

  /* Additional utility classes */
  .small {
    font-size: 18px;
  }

  .tiny {
    font-size: 15px;
  }

  .highlight {
    background: #f0f7fb;
    padding: 10px;
    border-radius: 8px;
    border-left: 4px solid #1e5266;
  }
---

<!-- _class: lead -->

# Beyond Likelihoods: Bayesian Parameter Inference for Black-Box Simulators with sbi

## A Hands-On Introduction to Simulation-Based Inference

**[EuroSciPy 2025](https://euroscipy.org/)** | Kraków, Poland | 90 minutes
**Case Study:** Ecological Monitoring with Limited Data

<br>

[Jan Teusen (Boelts)](https://janfb.github.io/) | [TransferLab](https://transferlab.ai/about/), appliedAI Institute for Europe

📱 **Materials:** [`github.com/janfb/euroscipy-2025-sbi-tutorial`](https://github.com/janfb/euroscipy-2025-sbi-tutorial)


![width:400px center](images/logo.png)

---

# 🐺 A Real Conservation Crisis in Poland

## October 2024: Headlines from Southern Poland

<div class="columns">
<div>

[![width:400px](images/tvp_cracow_terror_of_podhale.png)](https://krakow.tvp.pl/82694720/wilki-postrachem-podhala-rolnicy-apeluja-o-odstrzal)

**[TVP Kraków Reports](https://krakow.tvp.pl/82694720/wilki-postrachem-podhala-rolnicy-apeluja-o-odstrzal):**
*"Wolves are the terror of Podhale. Farmers are calling for a cull"*

</div>
<div>

### The Crisis

- **Wolf attacks increasing** south Poland
- Targeting livestock and domestic animals
- **Farmers demanding action**
- **Wolves strictly protected** by law

</div>
</div>

---

# 📊 Research Confirms the Growing Problem

<div class="columns">
<div>

[![width:500px](images/pasternak_et_al_wolf_attacks_report.png)](https://www.researchgate.net/publication/389812129_Preliminary_report_on_wolf_attacks_on_flocks_of_sheep_of_native_breeds_in_Poland)

**[Pasternak et al.](https://www.researchgate.net/publication/389812129_Preliminary_report_on_wolf_attacks_on_flocks_of_sheep_of_native_breeds_in_Poland) (March 2025):**
*"Preliminary report on wolf attacks on flocks of sheep"*

</div>
<div>

### Key Findings (2015-2020)
- **76.9% of attacks** in southern Poland
- **Peak season:** July-August
- **Trend:** Increasing year over year
- **Most affected:** Podhale Zackel sheep

<br>

> *"Methods of protecting flocks should be improved"*

</div>
</div>

---

# 🎯 Your Mission: Inform Policy Decisions

## You're consulting for the State Environmental Agency

<div class="columns">
<div>

### The Dilemma

- **Conservation success:** Wolves recovering after near-extinction
- **Economic impact:** Farmers losing livestock
- **Policy question:** How much culling?

### Your Task

- Model wolf-deer dynamics
- Infer population parameters
- **Provide uncertainty estimates**

</div>
<div>

### Available Data

```python
# Summary statistics from monitoring
observations = {
    "deer_mean": 45.2,
    "wolf_mean": 8.7,
    "deer_std": 12.1,
    "wolf_std": 2.4,
    "max_counts": [78, 15],
    ...
}
```

**Challenge:** From limited data, infer ecosystem dynamics to guide policy.

</div>
</div>


---

# 🔬 Our Tool: The Lotka-Volterra Model

## Classic predator-prey dynamics

<div class="columns">
<div>

### The Equations

$$\frac{dx}{dt} = \alpha x - \beta xy\;,\; \frac{dy}{dt} = \delta xy - \gamma y$$

- $x$ = deers, $y$ = wolves
- $\alpha$ = deer birth rate
- $\beta$ = predation rate
- $\delta$ = wolf efficiency
- $\gamma$ = wolf death rate

</div>
<div>

### Why This Model?

<br>

- **Well-understood** ecological dynamics
- **Captures oscillations** seen in nature
- **Parameters map** to real processes
- **Fast to simulate** (enables SBI)

<!-- ```python
def lotka_volterra(params):
    α, β, δ, γ = params
    # Simulate populations
    return deer, wolves
``` -->

</div>
</div>

> **Next challenge:** How do we infer these parameters from observations?


---

# The Traditional Approach: Optimization

<div class="columns">
<div>

### Finding the "best" parameters

```python
# Grid search or optimization
best_params = optimize(
    simulator,
    observed_data
)
```

✅ **Gives an answer**
❌ **No uncertainty**
❌ **Misses alternatives**

</div>
<div>

### The result: A single point

```
α* = 0.52  # Birth rate
β* = 0.024 # Predation
δ* = 0.011 # Efficiency
γ* = 0.48  # Death rate
```

**But how confident are we?**

</div>
</div>


---

# 🎯 The Hidden Problem

## **Many parameters can explain your data!**

<div class="highlight">

### Three different parameter sets, similar observations:

| Parameters | α | β | δ | γ | Result |
|-----------|---|---|---|---|---------|
| **Set 1** | 0.52 | 0.024 | 0.011 | 0.48 | ✓ Matches |
| **Set 2** | 0.48 | 0.026 | 0.009 | 0.51 | ✓ Matches |
| **Set 3** | 0.55 | 0.022 | 0.012 | 0.45 | ✓ Matches |

</div>

<br>

> **Which one is correct?** 🤔
> **What about future predictions?** 📈


---

# What We Really Want: Distributions

<div class="columns">
<div>

### ❌ Point Estimate
- Single "best" value
- No uncertainty
- False confidence
- Poor predictions

</div>
<div>

### ✅ Posterior Distribution
- **Range of plausible values**
- **Quantified uncertainty**
- **Parameter correlations**
- **Robust predictions**

</div>
</div>

<br>

> **Goal:** `p(parameters | observation)`
> The probability distribution of parameters given what we observed


---

# The Likelihood Problem

## Why can't we just use Bayes' rule?

### Bayes' Rule:

$$p(θ|x) ∝ p(x|θ) × p(θ)$$

<br>


<div class="highlight">

**For complex simulators:**
- 📦 **Black-box:** No analytical likelihood `p(x|θ)`
- 🐌 **Slow:** Likelihood evaluations infeasible

</div>

<br>

**Examples:** Climate models, neural circuits, epidemics, cosmology...


---

# 🚀 Enter: Simulation-Based Inference

## Let neural networks learn from simulations!

```python
# The SBI workflow
1. parameters ~ prior()           # Sample parameters
2. data = simulator(parameters)   # Run simulation
3. train neural_network on (parameters, data) pairs
4. posterior = neural_network(observed_data)  # Inference!
```

<div class="highlight">

**Key insight:** Turn inference into supervised learning!
- No likelihood needed ✓
- Works with any simulator ✓
- Learns from examples ✓

</div>

---

# What You'll Learn Today

## Three hands-on exercises, progressive difficulty

<div class="columns">
<div>

### 📓 Exercise 1: First inference

**15 minutes**
- Load Lotka-Volterra simulator
- Run NPE in 5 lines
- Visualize posterior

</div>
<div>

### 🔍 Exercise 2: SBI Diagnostics
**20 minutes**
- Posterior predictive checks
- Coverage diagnostics
- Warning signs

</div>
</div>

<div class="center">

### 🚀 Exercise 3: Your SBI Problem

**20 minutes**

- Adapt template to your simulator, OR use provided examples

</div>

---

<!-- _class: lead -->

# Part 2: Core Intuition
## Two Approaches to SBI

---

# Classical vs Modern SBI

<div class="columns">
<div>

### 📚 Classical: Rejection Sampling

- Simple and intuitive
- No neural networks
- Inefficient in high-D
- Good for understanding

</div>
<div>

### 🧠 Modern: Density Estimation

- Efficient and scalable
- Amortized inference
- Handles high-D
- Powers the `sbi` package

</div>
</div>

<br>

> We'll see both for intuition, then use the modern approach

---

# Rejection Sampling in 5 Lines

```python
# The simplest SBI algorithm
accepted_params = []

for _ in range(n_simulations):
    θ = prior.sample()                    # 1. Sample parameters
    x_sim = simulator(θ)                  # 2. Simulate data
    if distance(x_sim, x_obs) < ε:        # 3. Accept if close
        accepted_params.append(θ)         # 4. Store accepted

posterior_samples = accepted_params       # 5. These approximate p(θ|x)
```
<br>

<div class="highlight">

**Intuition:** Keep parameters that produce data similar to observations

</div>

---

# The Curse of Dimensionality

## Acceptance rate drops exponentially! 📉

| Dimensions | Acceptance Rate | Simulations for 1000 samples |
|------------|----------------|------------------------------|
| **2D** | 10% | 10,000 ✅ |
| **5D** | 0.1% | 1,000,000 😐 |
| **10D** | 0.00001% | 10,000,000,000 ❌ |

<br>

<div class="highlight">

**Problem:** In high dimensions, almost nothing is "close" to your observation

</div>

<br>

> **Solution:** Learn the relationship instead of rejecting!

---

# Neural Posterior Estimation (NPE)

## Learning to predict _distributions_ given data

<div class="columns">
<div>

### The Network

**Input:** Observed data `x`
**Output:** Distribution `p(θ|x)`

```python
# Training
for θ, x in training_data:
    loss = -log q(θ|x)
    optimize(loss)

# Inference (instant!)
posterior = q(θ|x_observed)
```

</div>
<div>

### Key Innovation

Transform inference into **supervised learning**

1. Generate training pairs
2. Train neural density estimator
   - Gaussian: learn mean and std
   - Mixture of Gaussians
   - Normalizing flows
3. Instant posterior for new data!

</div>
</div>

---

# How NPE Training Works

### 1️⃣ Generate Training Data
```python
for i in range(n_simulations):
    θ[i] ~ prior()
    x[i] = simulator(θ[i])
```

### 2️⃣ Train Neural Network
```python
neural_net = NeuralPosterior()
neural_net.train(parameters=θ, observations=x)
```

### 3️⃣ Get Posterior (instant!)
```python
posterior = neural_net(x_observed)
samples = posterior.sample(10000)  # Milliseconds!
```

---

# The Power of Amortization

## Train once, infer many times! ⚡

| Method | New observation | Computational Cost |
|--------|-----------------|-------------------|
| **MCMC** | Re-run everything | Hours ⏰ |
| **Rejection** | Re-run everything | Hours ⏰ |
| **NPE** | Forward pass | **Milliseconds!** ⚡ |

<div class="highlight">

**Perfect for:**
- Real-time applications
- Interactive exploration
- Multiple observations

</div>

---

<!-- _class: lead -->

# 🚀 Let's Code!

## Three exercises, increasing complexity

### 📓 **Exercise 1:** First Inference (15 min)
### 🧑‍🎓 **SBI Diagnostics:** Recap and Input (5 min)
### 🔍 **Exercise 2:** Diagnostics (20 min)
### 🎯 **Exercise 3:** Your Problem (20 min)

<br>

> **Setup check:** Can everyone run this?

```python
import sbi
import torch
print("Ready for SBI! 🚀")
```

---

# Exercise 1: Your First Inference

## Wolf-Deer Dynamics from Summary Statistics!

```python
# The entire SBI workflow
from sbi.inference import NPE

# 1. Setup: simulator outputs summary stats
θ = prior.sample((10_000))
x = lambda θ: compute_summary_stats(lotka_volterra(θ))

# 2. Train neural network on summary statistics
npe = NPE(prior)
npe.append_simulations(θ, x).train()

# 3. Infer parameters from observed summaries
posterior = npe.build_posterior()

# 4. Sample & visualize uncertainty!
samples = posterior.sample((1000,), x=observed_stats)
```

**📝 Open notebook:** [`01_first_inference.ipynb`](../src/01_first_inference.ipynb)

---

<!-- _class: lead -->

# 🔍 Diagnostics for SBI

## Building Trust in Neural Posteriors

---

# What We Just Did: Recap

## Neural Posterior Estimation (NPE)

<div class="columns">
<div>

### 1️⃣ **Observe and simulate data**

- Observed data as summary stats
- Choose Lotka-Volterra and prior
- Generate parameters and data

### 2️⃣ **Trained NPE on simulations**

- Neural network learned p(θ|x)
- Amortized for instant inference

</div>
<div>

### 3️⃣ **Got posterior distributions**
- Not just point estimates!
- Full uncertainty quantification
- Parameter correlations revealed

### 4️⃣ **Made predictions**

```python
# Sample full trajectories
θ_post ~ posterior
x_pred = simulator(θ_post)
```

</div>
</div>

> **But wait...** How do we know we can trust these results? 🤔

---

# Why Diagnostics are Critical

## SBI is approximate inference - verification essential!

<div class="highlight">

### ⚠️ **Three sources of error:**

1. **Neural approximation:** Is the network accurate? Did we use enough data?
2. **Summary statistics:** Did we lose critical information?
3. **Prior specification:** Does it cover the true parameters?

</div>

### Without diagnostics, you risk:

- ❌ **Overconfident conclusions** (too narrow posteriors)
- ❌ **Policy disasters** (remember the wolves!)

> **Remember:** Your recommendations affect real ecosystems and livelihoods!

---

# The Four Essential Diagnostics

## Your trust-building workflow 🛡️

<div class="columns">
<div>

### 🎲 **1. Prior Predictive Check**
**Question:** Can my prior generate realistic data?

**How:** Sample prior → simulate → compare to observed

### 📊 **2. Training Convergence**
**Question:** Did the neural network learn properly?

**How:** Check loss curves, validation metrics

</div>
<div>

### 🔄 **3. Posterior Predictive Check**
**Question:** Can the posterior recreate observations?

**How:** Sample posterior → simulate → compare to observed

### 📏 **4. Calibration Check**
**Question:** Are uncertainties calibrated?

**How:** Test if 90% CI contains truth 90% of time


</div>
</div>

---

# Diagnostic 1: Prior Predictive Check

## Start before training! 🎲

```python
# Sample from prior and simulate
for _ in range(100):
    θ ~ prior()
    x = simulator(θ)
    plot(x)  # Should look reasonable!
```

<div class="columns">
<div>

### ✅ **Good Prior**
- Generates diverse, realistic data
- Covers observed range
- Includes edge cases

</div>
<div>

### ❌ **Bad Prior**
- Creates impossible scenarios
- Too narrow/wide
- Misses observed data

</div>
</div>

<div class="highlight">

**Example failure:** Prior allows negative birth rates → Populations go extinct instantly!

</div>

---

# Diagnostic 3: Posterior Predictive Check

## Can we recreate what we observed? 🔄

```python
θ_samples = posterior.sample((1000,))
for θ in θ_samples:
    x_pred = simulator(θ)
    summary_pred = compute_summaries(x_pred)

compare(summary_pred, summary_observed)
```

<div class="columns">
<div>

### What to look for:

✅ **Predicted summaries match observed**
✅ **Reasonable variation**

</div>
<div>

### Red flags:

❌ **Can't recreate observations**
❌ **Too narrow/wide predictions**

</div>
</div>

> **If this fails:** Your summary statistics likely lost critical information!

---

# Diagnostic 4: Simulation-Based Calibration

## Are your uncertainties honest? 📏

<div class="columns">

<div>

### The test:
1. Sample "true" parameters from prior
2. Simulate data and infer posterior
3. Check: Is truth in the credible interval?
4. Repeat 100+ times

</div>

<div>

```python
coverage_test = []
for _ in range(100):
    θ_true ~ prior()
    x = simulator(θ_true)
    posterior = infer(x)

    # Check if truth in 90% CI
    in_ci = θ_true in posterior.confidence_interval(0.9)
    coverage_test.append(in_ci)

coverage = mean(coverage_test)  # Should be ~0.9!
```

</div>
</div>

<div class="highlight">

**Expected:** 90% CI contains truth 90% of time
**Overconfident:** Coverage < 0.9 (CIs too narrow)
**Underconfident:** Coverage > 0.9 (CIs too wide)

</div>

---

# Exercise 2: Trust but Verify

## Critical with Summary Statistics! 🔍

**Why extra important?** SBI is approximate → Needs validation!

### Four key diagnostics:

<div class="columns">
<div>

### 1. Prior Predictive Check
- Can prior generate observations?
- Catch bad prior specification

### 2. Training Diagnostics
- Did neural network converge?
- Check for overfitting

</div>
<div>

### 3. Posterior Predictive Check
- Can posterior recreate data?
- Validates summary statistics choice

### 4. Simulation-Based Calibration
- Are credible intervals calibrated?
- 90% CI contains truth 90% of time?

</div>
</div>

**📝 Open notebook:** [`02_diagnostics.ipynb`](../src/02_diagnostics.ipynb)

---

<!-- _class: lead -->

# 🔍 Recap: Diagnostics for SBI

## Building Trust in Neural Posteriors

---

# Exercise 3: Your Own Problem

## Three options:

### 🔬 **Option A: Your Simulator**
If you brought one, we'll adapt it!

### 🎾 **Option B: Ball Throw Physics**
Simple projectile motion with air resistance

### 🦠 **Option C: SIR Epidemic Model**
Disease spread dynamics


**📝 Open notebook:** [`03_your_sbi_problem.ipynb`](../src/03_your_sbi_problem.ipynb)

---

<!-- _class: lead -->

# Part 4: Next Steps
## Where to go from here

---

# Beyond NPE: The Full SBI Toolbox

| Method | What it learns | Best for | Key advantage |
|--------|---------------|----------|---------------|
| **NPE** | `p(θ\|x)` | Fast amortized inference | Instant posteriors |
| **NLE** | `p(x\|θ)` | MCMC sampling | Exact inference |
| **NRE** | `p(θ,x)/p(θ)p(x)` | Model comparison | Hypothesis testing |
| **Sequential** | Iteratively | Sample efficiency | 10x fewer simulations |

<br>

<div class="highlight">

All available in the `sbi` package with the same interface!

</div>

---

# Advanced Topics

## Where to dive deeper 🏊

<div class="columns">
<div>

### Methods
- [NLE+`pyro` (**Talk Wed, 11:40, 1.38**](https://euroscipy.org/talks/KCYYTF/))
- Multi-round inference (sequential)
- Flow matching, diffusion models
- Tabular Foundation Models for NPE

</div>
<div>

### Applications
- Hierarchical Bayesian inference
- Expensive simulators
- High-dimensional problems
- Training-free SBI

</div>
</div>

<br>

> 📚 **Resources:** Papers, tutorials, and examples at [sbi.readthedocs.io](https://sbi.readthedocs.io/en/latest/)

---

# 🌍 Real-World Applications

## SBI in the wild

<div class="columns">
<div>

### Science

- 🧠 **Neuroscience:** [Neural circuits](https://elifesciences.org/articles/56261)
- 🦠 **Epidemiology:** [COVID-19 models](https://arxiv.org/abs/2005.07062)
- 🌍 **Climate:** [Weather prediction](https://gmd.copernicus.org/articles/14/7659/2021/gmd-14-7659-2021.html)
- 🔬 **Physics:** [Gravitational waves](https://journals.aps.org/prl/abstract/10.1103/PhysRevLett.127.241103)
- 🧬 **Biology:** [Gene regulation](https://link.springer.com/article/10.1186/s13059-021-02289-z)

</div>
<div>

### Engineering
- 🚗 **Automotive:** [Safety testing](https://2025.uncecomp.org/proceedings/pdf/21274.pdf)
- 📞 **Telecomm.:** [Radio propagation](https://arxiv.org/abs/2410.07930)

</div>
</div>

<br>

<div class="highlight">

[Webapp with overview of SBI applications](https://sbi-applications-explorer.streamlit.app/)

</div>

---

# Join the SBI Community!

![width:800px center](images/sbi_hackathon_crew.jpg)
*SBI Hackathon 2025, Tübingen - Join us next time!*

---

<div class="columns">
<div>

### 📦 **The Package**

- GitHub: [github.com/sbi-dev/sbi](https://github.com/sbi-dev/sbi)
- 700+ stars, 82+ contributors
- Active development

<br>

### 💬 **Get Help & Connect**

- [GitHub Discussions](https://github.com/sbi-dev/sbi/discussions)
- [Discord Server](https://discord.gg/eEeVPSvWKy)
- [🦋 Bluesky](https://bsky.app/profile/sbi-devs.bsky.social)

</div>
<div>

### 📚 **Resources**

- [SBI Documentation](https://sbi.readthedocs.io/en/latest/)
- New paper out **today**:
["SBI: a practical guide"](https://arxiv.org/abs/2508.12939)

<br>

### 🤝 **Contribute!**

- Join the next hackathon
- Use the package, raise issues
- Help others get started

</div>
</div>

---

<!-- _class: lead -->

# Thank You! 🙏

<div class="columns">
<div>

## Questions?

- [**GitHub Discussions**](https://github.com/sbi-dev/sbi/discussions)
- [**Discord Server**](https://discord.gg/eEeVPSvWKy)
- **Let's talk after the session**

<br>

## 📱 Materials

[github.com/janfb/euroscipy-2025-sbi-tutorial`](https://github.com/janfb/euroscipy-2025-sbi-tutorial)


</div>
<div>

## 📱 Feedback Form

![width:300px center](images/qr_code.png)

https://forms.gle/vf6rHA5DcAt2ird98

</div>
</div>

<br>

---

# References & Acknowledgments

## 🙏 Thanks To

- **Funding**: appliedAI Institute for Europe

<br>

- **Communities**: SBI community & EuroSciPy community

## 🛠️ Tools Used

- **[Marp](https://github.com/marp-team/marp)**: Markdown presentation ecosystem
- **Claude + [Serena MCP](https://github.com/oraios/serena)**: AI-assisted drafting & refactoring

<br>

See also the [references file](../materials/references.md)


---

<!-- _class: lead -->

# Backup Slides

---

# Mathematical Details: NPE Loss

## Training objective

The neural posterior estimator minimizes:

$$\mathcal{L} = -\mathbb{E}_{p(\theta, x)}[\log q_\phi(\theta|x)]$$

Where:
- $q_\phi(\theta|x)$ is the neural network approximation
- $\phi$ are the network parameters
- Expectation over joint distribution of parameters and data

**Implementation:** Normalizing flows for flexible distributions
