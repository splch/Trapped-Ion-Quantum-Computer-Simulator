## Homework: Create a Bell state from absolute beginning to end (using this simulator)

### Goal

Start from a “cold lab” and produce a **two-ion Bell pair**, then verify it through measurement statistics and a simple error study.

### Learning objectives

By the end, you should be able to:

1. Explain what a Bell state is and why it is entangled.
2. Bring the simulator from “vacuum not ready” to “ions loaded + cooled”.
3. Write a pulse program that implements a standard Bell-pair recipe.
4. Collect measurement data and interpret it (histogram, bitstring probabilities, SPAM).
5. Explore how decoherence/readout/crosstalk change your results.

### Background (read before starting)

* **Bell states** are maximally entangled two-qubit states. One standard way to create the (\lvert\Phi^+\rangle = (\lvert00\rangle + \lvert11\rangle)/\sqrt2) Bell state is: start from (\lvert00\rangle), apply a **Hadamard (H)** on qubit 0, then apply **CNOT** (control qubit 0, target qubit 1).
* In trapped-ion hardware, a common native entangling interaction is the **Mølmer–Sørensen (MS) gate**, which can directly generate maximally entangled states (up to phases) for two ions.

This simulator's DSL includes **Pi/2** (think “Hadamard-like”) and **MS** (think “ion entangler”), so we will build a Bell pair using that trapped-ion style recipe, while also connecting it to the H+CNOT circuit picture.

---

# Part 0 — Reset and document your starting point

1. Click **Reset**.
2. In the **Overview** cards, record:

   * Pressure
   * Ions / ( \bar n )
   * RF / ( \omega_z )
   * Ready/status message

**Deliverable:** one screenshot of the Overview + Setup status.

---

# Part 1 — Vacuum + loading (the “lab bring-up”)

### A. Pump down

1. In **Setup → Vacuum, Oven, Loading**:

   * Set **Base pressure** to **1×10⁻¹¹ mbar** (or better).
   * Set **Pump speed** to ~**70%**.
   * Click **Pump** (toggle ON).
2. Optional but instructive:

   * Toggle **Bake** ON and watch how it affects the pressure trend.

**Record:** pressure after ~30–60 s of simulator time.

### B. Load exactly 2 ions

1. In **Experiment → Shot & Noise Model**:

   * Set **Qubits = 2**
2. Go back to **Setup** and click **Load Ions**.
3. Verify the **Ion Chain** canvas shows **2 ions**.

### C. Cool the motion

1. Toggle **Cool ON**.
2. Wait until ( \bar n ) drops noticeably (aim: ( \bar n \lesssim 1 ) if you can).

**Deliverable:** write down:

* Final pressure
* Number of ions loaded
* Final ( \bar n )

---

# Part 2 — Program a Bell-pair sequence

### A. Build the Bell-state pulse program

In **Program → Pulse Program**, paste/edit this exact program (one line per op):

```
Doppler 2ms
OpticalPump 200us
Pi/2 10us
MS 60us
Readout 200us
```

**What each line “means” conceptually**

* **Doppler / OpticalPump**: prepare a clean initial internal state (analog of “reset to (\lvert00\rangle)”).
* **Pi/2**: creates a superposition (Hadamard-like step in the H+CNOT picture).
* **MS**: entangles the two ions (native trapped-ion entangling step).
* **Readout**: measure via fluorescence and convert counts → bit values.

### B. Compile

Click **Compile**.

* Confirm the **Pulse Timeline** updates and the status shows a successful compile.

**Deliverable:** screenshot of the compiled timeline.

> If you see the “Bad duration on line … \n …” error: you pasted escaped newlines. Replace `\n` with real line breaks or re-load from a template and edit.

---

# Part 3 — Run, collect data, and interpret it

### A. Choose baseline experiment settings

In **Shot & Noise Model**:

* Qubits: **2**
* Shots: **5000** (or 2000 if your device is slow)
* Sensor: **PMTs**
* XTalk: **0.05**
* T2: **25 ms**
* Heat: **0.03 q/ms**
* Advanced p1/p2: keep defaults for baseline

In **Readout**:

* Detection time: **200 µs**
* Bright μ: **26**
* Dark μ: **3**
* Threshold: **12**
* Classifier: **Threshold**

### B. Run

Click **Run**.

### C. Record outcomes

After the run, record:

1. **Top outcomes table** (bitstrings + probabilities)
2. **SPAM estimate**
3. A screenshot of the **histogram** and **outcome distribution** plots

**Interpretation prompt (short answer, 3–5 sentences):**

* Which bitstring(s) dominate?
* What does that suggest about correlations between the two ions?
* Is the histogram well-separated (bright vs dark), or overlapping?

---

# Part 4 — Verify correlations in a second basis (coherence check)

A classic entanglement check is to measure correlations in more than one basis (not just Z/“computational”). In many real experiments, this is done by applying basis-rotation pulses before readout.

### A. Add a basis rotation before measurement

Modify your program to:

```
Doppler 2ms
OpticalPump 200us
Pi/2 10us
MS 60us
Pi/2 10us
Readout 200us
```

Compile and Run again (same shots).

### B. Compare distributions

**Deliverable:** put side-by-side (two screenshots):

* Distribution for the original program
* Distribution for the rotated-basis program

**Questions**

1. Did the dominant outcomes change?
2. What does that imply about the role of the **final Pi/2**?
3. In an ideal Bell-pair experiment, why is checking a second basis important? (Conceptual answer.)

---

# Part 5 — Error study (pick 2 knobs)

Pick **two** of the following and do a mini-study. For each knob: run at 3 settings, record SPAM + top outcomes, and write 2–3 sentences summarizing the trend.

Choose from:

* **T2**: 25 ms → 10 ms → 3 ms
* **XTalk**: 0.00 → 0.05 → 0.20
* **Heat**: 0.00 → 0.03 → 0.10
* **Readout threshold**: 8 → 12 → 16
* **Classifier**: Threshold vs Naive Bayes vs LLR (try LLR at −2, 0, +2)

---

## Submission checklist (what students hand in)

1. Screenshot: **starting state** + **final prepared state** (pressure/ions/(\bar n))
2. Your **final Bell program text** (both versions if you did the basis-rotation part)
3. Screenshot(s): timeline, histogram, distribution, top outcomes
4. A short write-up (½–1 page) answering:

   * What is your target Bell state (in words and/or ket notation)?
   * How does your program map to the idea “superposition + entangling gate”?
   * What happened when you added the final Pi/2?
   * What did your error-knob study show?
