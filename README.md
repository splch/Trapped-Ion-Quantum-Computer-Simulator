# Quantum Ion Lab Simulator Runbook

This runbook is a practical “how to use it” guide with just enough context to help you understand what each step is doing.

## 1) What this simulator is

* A **browser-only, single HTML file** that mimics a trapped-ion lab workflow: **vacuum → load ions → cool → run pulse program → photon-count readout → bitstrings**.
* It aims to be **educational**: you can explore how preparation, gates, decoherence, heating, crosstalk, and readout classification change observed outcomes.
* The UI is organized as a workflow:

  * **Setup** (vacuum / loading / cooling / trap parameters)
  * **Program** (pulse DSL, compile, timeline)
  * **Experiment** (shots + noise knobs + readout classifier)
  * **Diagnostics** (lasers, micromotion, camera)
  * **Log** (action history)

**Physics context (optional):** In real trapped-ion systems, preparation uses laser cooling + optical pumping, and measurement is typically **state-dependent fluorescence** (electron-shelving style photon counting). 
**Simulation context (optional):** Efficient simulation of many Clifford-style circuits often uses stabilizer/tableau methods (Gottesman–Knill style), rather than full statevectors.

---

## 2) Launch and basic controls

### Open and run

1. Save the simulator as `quant-ion-lab.html`
2. Open it in a modern browser (Chrome/Firefox/Edge/Safari).
3. If “Copy log” doesn’t work, that’s usually browser clipboard permissions for local files.

### Top-bar essentials

* **Compile**: parses the pulse program and draws the timeline.
* **Run**: compiles then executes shots with current noise/readout settings.
* **Tooltips**: toggles the subtle hover hints.
* **Help**: built-in manual + shortcuts.
* **Filter**: type to quickly find panels/controls.

### Keyboard shortcuts

* **Ctrl/⌘ + Enter** → Run
* **Ctrl/⌘ + Shift + Enter** → Compile
* **?** → Help
* **Esc** → Close help/nav, dismiss tooltip

---

## 3) Standard operating procedure (cold start → data)

### Step A — Reset

* Click **Reset** (top or sidebar).
* Confirm the status pill returns to **INIT/READY/VAC/NO IONS** as it updates.

### Step B — Pump down (vacuum)

In **Setup → Vacuum, Oven, Loading**:

1. Set **Base pressure** (lower is better).
2. Set **Pump speed** (higher pumps faster in the toy model).
3. Toggle **Pump** ON.
4. Optional: toggle **Bake** ON briefly to improve pressure trend.

**Goal:** get pressure into the “OK-ish” or “UHV region” so you don’t lose ions.

### Step C — Set qubit count and load ions

In **Experiment → Shot & Noise Model**:

* Set **Qubits = N** (this is the ion-chain length used for runs).

Back in **Setup**:

* Click **Load Ions** and verify the **Ion Chain canvas** shows **N ions**.

### Step D — Cool the motion

* Toggle **Cool** ON.
* Watch **n̄** fall (lower is better). In real experiments, Doppler cooling reduces motional energy and improves gate performance. 

### Step E — Program, compile, run

1. Go to **Program → Pulse Program**
2. Choose a **Template** or paste your sequence
3. Click **Compile** (timeline should update)
4. Go to **Experiment**, set **Shots**, noise, readout parameters
5. Click **Run**
6. Inspect:

   * **Readout Histogram** (bright vs dark separation)
   * **Outcome Distribution** (bitstring probabilities)
   * **SPAM estimate** (a rough measurement-error proxy)

---

## 4) Pulse Program DSL

### Syntax rules

* **One instruction per line**:
  `Op Duration`
* **Duration units**: `us`, `µs`, `ms`, `s`
* Blank lines allowed.
* Comment lines supported if they start with `#`, `//`, or `;`.

### Supported ops (conceptual meaning)

* `Doppler` — cooling/prep step (brings motion down; sets the stage).
* `OpticalPump` — prepares a clean internal state (like “reset”).
* `Pi/2` — a Hadamard-like single-qubit rotation (basis change / superposition).
* `Pi` / `X` / `Rabi` — a single-qubit flip/drive (simplified).
* `MS` — entangling interaction (common trapped-ion entangler). 
* `Wait` — idle time where decoherence/heating matter.
* `Readout` — fluorescence-style measurement (photon counts → bits). 

### Example programs

**1) Ramsey (coherence / calibration)**

```text
Doppler 2ms
OpticalPump 200us
Pi/2 10us
Wait 50us
Pi/2 10us
Readout 200us
```

**2) Bell-style “superposition + entangle”**

```text
Doppler 2ms
OpticalPump 200us
Pi/2 10us
MS 60us
Readout 200us
```

**3) “Second basis” measurement (add a basis rotation before readout)**

```text
Doppler 2ms
OpticalPump 200us
Pi/2 10us
MS 60us
Pi/2 10us
Readout 200us
```

---

## 5) Experiment knobs: what to touch first

### Shots & noise model

* **Qubits**: number of ions/qubits.
* **Shots**: number of repetitions (Monte Carlo sampling).
* **XTalk (0–1)**: readout crosstalk; bright ions can bleed into neighbors.
* **T2 (ms)**: coherence time; lower → more dephasing-like effects.
* **Heat (q/ms)**: motional heating rate; higher can degrade gates.
* **p1 / p2** (advanced): extra stochastic error knobs (single-qubit / correlated).

**Practical defaults for “clean-ish” behavior**

* Start with: **XTalk 0.00–0.05**, **T2 25 ms**, **Heat 0.00–0.03**.

### Readout model

* **Det. time (µs)**: longer → more photons (better separation, slower).
* **Bright μ / Dark μ**: expected photon counts for bright/dark.
* **Threshold**: counts ≥ threshold → “bright (1)”.
* **Classifier**:

  * *Threshold*: simplest and usually easiest to reason about
  * *Naive Bayes / LLR*: more flexible if distributions overlap

**How to “tune readout” quickly**

1. Run a simple `Readout`-heavy program (or any program).
2. Look at the **histogram**:

   * If bright/dark peaks overlap → increase det. time or improve μ separation.
   * If threshold is cutting through the wrong place → adjust threshold.

(Real trapped-ion readout is commonly modeled as photon counting from state-dependent fluorescence; thresholding is a standard baseline classifier. )

---

## 6) How to interpret outputs

### Pulse Timeline (after Compile)

* Confirms the program parsed correctly.
* Lets you sanity-check **relative durations** (e.g., a 2 ms Doppler pulse dwarfs a 10 µs gate).

### Readout Histogram

* Shows distributions of photon counts for “bright-like” vs “dark-like.”
* The vertical threshold line is your **bit decision boundary**.

### Outcome Distribution + Top outcomes table

* Shows the most frequent bitstrings and their probabilities.
* For an ideal Bell-type correlation test in the Z basis, you’d expect **mostly `00` and `11`** (or another correlated pair depending on phases/basis).

### SPAM estimate

* A rough proxy for **State Prep And Measurement** error.
* Use it as a knob-check: if SPAM gets worse as you increase XTalk or lower det. time, that’s expected.

---

## 7) Troubleshooting checklist

### Compile errors (most common)

* **“Bad duration on line …”**

  * Ensure *every op that needs a duration has one*, with a unit: `10us`, `0.2ms`, `200µs`, etc.
  * Make sure each op is on its **own line** (don’t paste literal `\n` sequences).

### Status shows VAC / NO IONS

* **VAC**: pressure too high → turn **Pump** on, reduce oven, consider **Bake**.
* **NO IONS**: set **Qubits** then click **Load Ions**.

### “Results look wrong”

Work through these in order:

1. **Qubits mismatch**: confirm Experiment Qubits equals the loaded chain length.
2. **Readout threshold wrong**: histogram overlap or poorly placed threshold can make one bitstring dominate.
3. **XTalk too high**: can turn neighbors bright and bias strings.
4. **T2 too low / Heat too high / p1/p2 too high**: can destroy expected correlations.

---

## 8) Recommended learning workflow (for students)

1. **Readout sanity check**: run `Readout 200us` with different thresholds until histogram separation makes sense.
2. **Ramsey**: vary `Wait` and see how outcomes change vs T2.
3. **Bell recipe**: run the Bell program, then add a final `Pi/2` before readout and compare distributions.
4. **Error study**: change **two knobs** (e.g., XTalk and T2) and record how top outcomes + SPAM shift.

---

## 9) Glossary (quick)

* **n̄ (n-bar)**: average motional excitation; lower is “colder.” 
* **T2**: coherence time (dephasing-like).
* **SPAM**: state preparation and measurement error proxy.
* **MS gate**: Mølmer–Sørensen entangling interaction used in trapped ions. 
* **State-dependent fluorescence**: readout via photon counts that differ between internal states. 
