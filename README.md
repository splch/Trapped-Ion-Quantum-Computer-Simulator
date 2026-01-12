# Quantum Ion Lab Simulator — Educational Runbook

This simulator is a **lab-workflow teaching tool**: you “prepare the hardware” (vacuum + loading + cooling), “write a pulse sequence,” then run many **shots** and interpret **fluorescence-style readout** and diagnostics. It uses **simplified, toy models** to make cause/effect intuitive (it is not a hardware-accurate predictor).

## 1) Mental model of what you're simulating

In real trapped-ion systems, ions are confined by electric fields (RF + DC), cooled and prepared with lasers, and measured by **state-dependent fluorescence** (one state scatters photons strongly while another scatters weakly, and a photon counter distinguishes them).

In addition, real experiments work hard to reduce **excess micromotion** (driven motion caused by stray electric fields); compensation is typically done by applying voltage patterns on compensation electrodes to cancel those stray fields.

The simulator's “MS” operation is inspired by the **Mølmer–Sørensen family of entangling gates**, which couple qubits through a shared motional mode (and are therefore sensitive to motional effects such as heating).

---

## 2) Quick start in ~3 minutes

### A. Get into a “good lab state” (Setup)

1. Go to **Setup → Vacuum, Oven, Loading**.
2. Click **Pump** (turn it ON).
3. Watch **Pressure** until it's low enough that the status bar trends **READY/NO IONS** instead of **VAC**.
4. Click **Load Ions** (loads an ion chain equal to your “Qubits” setting).
5. Click **Cool** (turn it ON) to reduce motional energy (**n̄**).

What to look for:

* **Pressure** decreasing → fewer loss events.
* **Ions** > 0 and pressure acceptable → the app status tends toward **READY**.

### B. Compile a pulse program (Program)

1. Go to **Program**.
2. Pick **Template: Ramsey** (or any template).
3. Click **Compile** (or press **Ctrl+Shift+Enter**).
4. Confirm the **Pulse Timeline** plot updates.

### C. Run shots (Experiment)

1. Go to **Experiment → Shot & Noise Model**.
2. Set **Qubits** and **Shots** (start with 4 qubits, 2000 shots).
3. Click **Run** (or press **Ctrl+Enter**).
4. Check:

   * **Readout Histogram**
   * **Outcome Distribution**
   * **SPAM estimate** (simple “bit error proxy” for this simulator)

---

## 3) Interface map (what each area is for)

### Top bar

* **Compile / Run**: fastest way to build and execute.
* **Filter**: type “laser”, “readout”, “T2”, etc. to quickly hide panels you don't need.
* **Status pill**:

  * **READY**: vacuum OK and ions loaded
  * **NO IONS**: vacuum OK but nothing loaded
  * **VAC**: vacuum too poor for stable trapping (in the simulator)

### Left sidebar (Navigation + Quick actions)

* **Jump to** workflow sections.
* **Quick actions** mirror key buttons (Pump/Oven/Bake/Cool/Load/Reset).
* **Live summary** shows pressure, ion count, and approximate trap numbers.

### Main column (workflow)

* **Setup** → hardware prep
* **Program** → pulse DSL + compile
* **Experiment** → shots/noise/readout + plots
* **Diagnostics** → lasers / micromotion / camera
* **Log** → what you did + run summaries

### Right column (Live plots)

* Live visualization panels for “at-a-glance” monitoring.

---

## 4) The workflow you should teach

### Step 1 — Vacuum discipline (Setup)

Controls:

* **Base pressure**: chooses the best-case floor your chamber can reach.
* **Pump speed**: stronger pumping → pressure approaches base faster (toy behavior).
* **Oven set / Oven toggle**: increases loading conditions but often worsens pressure (toy outgassing).
* **Bake**: in real life, bake-out helps long-term vacuum health; in this simulator it mainly behaves like a “thermal outgassing” knob (expect a pressure rise while it's on).

Teaching points:

* Higher pressure → more collisions → more ion loss events (simulated).
* Loading is easier when the “source” is on (oven), but it's a tradeoff.

### Step 2 — Load ions, then cool (Setup)

* **Load Ions** creates a chain with N ions (based on “Qubits”).
* **Cool** reduces **n̄** over time (the toy “motional energy” variable).

Teaching points:

* Cooling is the bridge between “we trapped something” and “we can do coherent control.”

### Step 3 — Write/compile a pulse program (Program)

The pulse DSL is **one operation per line**:

**Format**

```
Operation Duration
```

**Duration units** (must include a unit):

* `us` or `µs`
* `ms`
* `s`

**Common ops you'll see in templates**

* `Doppler 2ms`
* `OpticalPump 200us`
* `Pi/2 10us`
* `Wait 50us`
* `MS 60us`
* `Readout 200us`

**Common compile mistake**

* If your whole program accidentally becomes a single line containing the literal text `\n`, compilation will fail because the parser expects real line breaks. Make sure each op is on its own line (press Enter in the text area), and every duration includes units.

Teaching points:

* “Compile” is a sanity check + timeline visualization step. It's an opportunity to discuss sequencing and timing.

### Step 4 — Configure experiment statistics + noise (Experiment)

Key controls:

* **Qubits**: number of ions used as qubits (1–16).
* **Shots**: repetitions; larger shots → smoother histograms and distributions.
* **Sensor**:

  * **PMTs**: simple counts model
  * **Camera**: additionally generates a synthetic camera frame after a run
* **XTalk**: neighbor contamination (toy crosstalk).
* **T2**: coherence time knob; lower T2 → worse outcomes over waits.
* **Heat**: increases **n̄** over time (toy heating).
* Advanced:

  * **p1 / p2**: extra error knobs (single-qubit-like and correlated-like proxies)

Teaching points:

* Separate **statistical uncertainty** (shots) from **systematic error** (noise knobs).

### Step 5 — Understand readout (Experiment → Readout)

The simulator models fluorescence-style measurement:

* Each ion produces a photon count.
* Two “count clouds” appear: **dark** vs **bright**.
* A classifier converts counts into bit values.

This mirrors the key real-world idea: measurement in trapped ions often relies on **state-dependent fluorescence** with photon-counting detectors.

Controls:

* **Detection time**: longer generally separates bright/dark better (but can have tradeoffs).
* **Bright μ / Dark μ**: average counts for each state (toy model).
* **Threshold**: simplest classifier.
* **Classifier**:

  * Threshold: deterministic cutoff
  * Naive Bayes: soft/probabilistic decision
  * LLR: shifts decision boundary with a slider
* Advanced dynamics:

  * **Bin / Pump / Depump**: toy dynamics that mix bright/dark behavior over time

Outputs:

* **SPAM estimate**: simplified error proxy for “state preparation and measurement” effects in the simulator.
* **Histogram**: shows overlap; overlap means misclassification risk.
* **Outcome distribution**: top measured bitstrings and their frequencies.

---

## 5) Diagnostics panels (how to use them in lessons)

### Laser Rack

What it teaches:

* “Locks” and “errors” are a **control-system** story: a locked laser has smaller drift; unlocked tends to wander.
  How to demo:
* Turn one lock off → watch error grow → discuss why laser stability matters for gates and readout.

### Micromotion

What it teaches:

* Excess micromotion comes from imperfect fields; compensation reduces it.
  How to demo:
* Change **Comp X/Y** → see MM amplitude / sideband proxy change.
* Use **Auto-minimize** → watch it step toward a better point.

### Camera View

How to demo:

* Set **Sensor = Camera**, run shots, then view the synthetic frame.
* Toggle overlays (ROI / centroid) to discuss imaging-based readout concepts.

---

## 6) Troubleshooting checklist (student-friendly)

### “VAC” / ions keep getting lost

* Turn **Pump ON**, increase pump speed.
* Reduce **Oven** (it can raise pressure in this toy model).
* Load ions again after pressure improves.

### Compile error: “Bad duration…”

* Ensure **each operation is on its own line** (real line breaks, not `\n` text).
* Ensure the duration has a unit: `200us`, `2ms`, `0.2ms`, etc.
* Example of a valid minimal program:

  ```
  Readout 200us
  ```

### SPAM is high / histogram overlaps a lot

* Increase **Detection time**.
* Adjust **Threshold** (move it between dark/bright peaks).
* Reduce **XTalk**, increase **T2**, reduce **Heat**.

### Camera view is blank

* Set **Sensor = Camera**, then **Run** again (camera frame updates on runs).

---

## 7) Suggested 15–30 minute classroom activities

1. **Vacuum vs stability**

   * Compare high vs low pressure settings.
   * Observe ion loss frequency and discuss collisions.

2. **Thresholding lab**

   * Keep everything fixed; vary **Threshold** and **Detection time**.
   * Goal: minimize SPAM by separating distributions.

3. **Decoherence intuition**

   * In the program, increase `Wait` duration.
   * Lower **T2** and compare outcomes; discuss why “waiting” is an operation.

4. **Entangling-gate discussion**

   * Use the **MS Gate** template.
   * Discuss that MS-family gates couple through motion (and why heating can matter).

---

## 8) Mini-glossary (for the handout margin)

* **RF trap / Paul trap**: electric-field confinement method for ions (RF + DC).
* **Optical pumping**: laser-based preparation into a known internal state.
* **Laser cooling**: reduces ion motion to enable coherent control.
* **n̄ (n-bar)**: mean motional quanta (toy “how hot is the motion”).
* **T2**: coherence time scale (toy knob controlling decoherence).
* **State-dependent fluorescence**: measurement method where one state scatters many photons and another scatters few.
* **Micromotion**: driven RF motion; “excess” micromotion arises from stray fields and is compensated.
* **MS gate**: a common trapped-ion entangling-gate family mediated by shared motion.
