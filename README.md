# F1 Data Scientist Case Study — Braking & Lock-up Risk

## Contents

- [Problem statement](#problem-statement)
- [The data](#the-data)
- [Key definitions](#key-definitions)
- [Part A – Brake report](#part-a--brake-report)
- [Part B – Lock-up automated alert model](#part-b--lock-up-automated-alert-model)
- [What to hand in](#what-to-hand-in)
- [Practicalities](#practicalities)

---

## Problem statement

You are a data scientist supporting the Technical Delegates of a GT sprint series. You are working towards enabling a new safety measure to reduce the risk of incidents caused by **brake lock-up**: when the braking force exceeds the grip of the tyre, the wheels stop rotating and slide on the asphalt.

The first exploration was to **report lock-up occurrences** to the delegates. The delegates are now excited about the results and they want to **trigger an alert on the driver's dash**, allowing them to react before they lock up. FIA has hired you to build an automated alert model that predicts imminent brake lock-ups.

---

## The data

You get **25 stints** of sim-racing telemetry from **8 drivers** racing a **BMW Z4 GT3** on the **Barcelona GP circuit**, amounting to about **4.8 hours** of driving.

Each stint is a folder, `data/session_XX/`, with five files written by different loggers at different rates:

| File | Content | Nominal rate |
|---|---|---|
| `driver_inputs.parquet` | Throttle, brake, clutch, steering, gear | 50 Hz |
| `chassis.parquet` | Speed, RPM, accelerations, rotation rates, wheel speeds, tyre slip ratios, ABS/TC activity | 50 Hz |
| `tyre_loads.parquet` | Tyre loads, suspension travel, ride height, loaded radius | 20 Hz |
| `tyre_temps.parquet` | Tyre core and surface (inner/middle/outer) temperatures, pressures | 5 Hz |
| `position.parquet` | Position on track, lap counter and times, tyres off track, fuel | 10 Hz |

> **IMPORTANT**:  Every file has its own `t_s` timestamp column (seconds from the start of the stint). 

Also included:

| File | Purpose |
|---|---|
| `data/sessions.csv` | One row per stint: anonymised session and driver, number of laps (including partial out- and in-laps), duration |
| `data_dictionary.csv` | Every channel with its unit, nominal rate, logger latency and, where useful, how it is computed (e.g. the tyre slip ratio) |


> **Data source:** Assetto Corsa Gym (A. Remonda et al., *"A Simulation Benchmark for Autonomous Racing with Large-Scale Human Data"*, NeurIPS 2024 Datasets and Benchmarks), licensed CC BY 4.0. The data was resampled, split and anonymised for this exercise.

---

## Key definitions

### Tyre slip ratio

$$
\text{slip}_* = \frac{\omega_* \cdot R_0 - v}{v}
$$

where:

- $\omega_*$ is `wheel_angular_speed_*`, in rad/s where `*` can be any of the wheels (FR for front right, RL for rear left)
- $R_0$ is the unloaded tyre radius (0.340 m)
- $v$ is `ground_speed` in m/s

The slip ratio characterises the speed of a tyre surface relative to the speed of the car:

| Slip ratio | Meaning | Example |
|---|---|---|
| ≈ 0 | The tyre has full adherence to the track and does not slip | Steady driving |
| > 0 | The tyre spins faster than the car is moving | Burnout / wheelspin |
| < 0 | The tyre spins slower than the car is moving | Brake lock-up |

### Brake lock-up

A **brake lock-up** is a front wheel (FL or FR) whose slip ratio stays **below −60% (negative) for at least 0.2 s without interruption**.

Some negative slip is acceptable: it is part of braking at the limit and is deemed recoverable. Full lock-ups are much riskier.

Additional conditions:

- **Start time**:  the lock-up starts at the first sample below −60%.

To help you make sure you have the right definition, we have included in the training stints two label columns in  chassis.parquet`: 
- lockup_start: a lock-up starts at this timestamp.
- target: a lock-up starts within the next 0.5 s. You can use this column as reference label for your task. 

---

## Part A – Brake report

Before building your model, prepare a **5-minute explanation** to give at the follow-up interview, presenting how you approached the lock-up analysis and the concepts behind your automated alert model. Your audience is the **Technical Delegates**: engineers, but with no ML background. You can prepare any format and material that you feel comfortable with. 

Questions you could investigate:

- Are certain drivers more likely to experience lock-ups, and what differs in their inputs?
- Where do lock-ups tend to happen on the track?
- Do lock-ups have any lasting effect on other areas of the car?

These questions are a guideline only. We encourage you to explore other areas that will drive your pre-processing and feature engineering in Part B.

---

## Part B – Lock-up automated alert model

Build a model that decides whether to **raise an alert (1/0) because a brake lock-up will start within the next 0.5 s**.

### What we ask

| Topic | Expectation |
|---|---|
| **Models** | At least two approaches (one can be a simple baseline, e.g. a heuristic or simple model). |
| **Inputs** | Any channel may be used, including the slip ratios. Tell us which inputs you chose, which you left out, and why. |
| **What to predict** | At time *t*, an alert decision (1/0): will a brake lock-up start within the next 0.5 s? |
| **When to predict** | Speed above 30 km/h. Below this speed the slip ratio is not well-defined. |
| **Validation** | Must reflect performance on drivers the model has not seen. |
| **Interpretation** | What drives lock-up risk according to your model, and how far you would trust that explanation. |
| **Delivery** | The code reflecting your data exploration and that can run your selected model with new data following the same format. 

---

## What to hand in

- **Code** provide access to a git repository with the code and any relevant artifacts. Please ensure the code can run with new data following the same format and on an environment created from scratch. 

---

## Practicalities

- **Deadline:** 15 November, 23:59 CET. 
- **Effort**: we ask you to not spend more than a total of 6 hours of work on the assignment. This facilitates a fair comparison across submissions. If you feel curious about the task and decide to spend a bit more time, we encourage you to share the real time you use to produce the shared results.  
- **Tools:** use Python and any open-source libraries you like.
- **AI assistants** are allowed, but be transparent about their use. In the follow-up interview you will walk us through your code and run it live, so make sure you understand everything you submit.
- **Open questions:** some parts of this take-home are deliberately left open to interpretation. Feel free to contact us for any technical issues (access rights, etc.).
