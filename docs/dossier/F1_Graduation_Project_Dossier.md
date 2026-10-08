# F1 Graduation Project Dossier

Three shortlisted ideas, a recommendation, and the decisions still open

## About this document

This dossier compares three graduation project ideas in the F1 domain. Each idea needs a physical build with small cars. Idea 1 uses the team's FastF1 data directly, for a replay test. Idea 2 reuses the team's modelling discipline on logs from its own car. Idea 3 is the least connected to the existing work.

How to read it:

- Sections 4 to 6 describe the three ideas, each with the same eight subsections.
- Section 7 compares them. Section 8 gives the recommendation and a decision path.
- Section 9 covers team, budget and timeline. Section 10 is the risk register. Section 11 covers safety. Section 12 sets evaluation standards.
- Section 13 lists the eight decisions, D1 to D8, with the answers so far.
- Appendices A to E hold the glossary, parts lists, pseudocode, checklists and references.

Numbers in square brackets, such as [4], refer to the reference list in Appendix E.


# 1 Executive summary

The team already has real data. The repository holds 57,813 FastF1 laps from 2022 to 2024 (`Data/raw/master_raw_laps.csv`), a derived feature table of 55,040 rows (`Data/features.csv`), and one notebook that fits lap time against tyre life (`Models/Hazem/Trail.ipynb`).

The notebook's result is a warning, not a finding. On 711 HARD laps from the 2024 Bahrain opener, the TyreLife slope is 0.024521 and R² is 0.024. Driver identity alone gives R² of 0.404, far more than tyre life does. Driver effects, and probably fuel load, must be controlled before the small slope can be read as tyre degradation.

Three ideas are shortlisted. All are in the F1 domain, and all need a physical build with small cars.

- **Idea 1, RaceState (recommended default).** A live timing system for four to six small cars. One overhead camera and a Raspberry Pi 5 produce running order, gaps, intervals and laps. Infrared (IR) gates and tags give an independent check on crossing times.
- **Idea 2, Learn-to-Deploy.** A small RC car controlled by an ESP32. It must spend a virtual, 2026-style energy budget, following a lookup table that is computed offline by dynamic programming (DP).
- **Idea 3, learned racing.** A driving policy trained with reinforcement learning (RL) in simulation, run on a real car, and compared with tuned pure pursuit and model predictive control (MPC).

**Recommendation.** Choose Idea 1. Its first milestone arrives early, its ground truth is independent and easy to check, its safety risk is low, and its machine-learning and computer-vision content is substantial. Choose Idea 2 if the team is strong in embedded systems and accepts battery work. Choose Idea 3 only with real reinforcement-learning experience and a high tolerance for crashes and negative results.

**Planning numbers** (USD, planning estimates):

- Idea 1: subtotal 390 to 865; about 450 to 1,000 with 15% contingency.
- Idea 2: subtotal 230 to 480; about 290 to 600 with 25% contingency.
- Idea 3: subtotal 275 to 650; about 360 to 850 with 30% contingency, plus about 250 for a Jetson if chosen.

**Decisions needed.** Eight decisions are listed in Section 13 (four have answers so far): the idea, team roles, budget band, six or nine months, supervisor sign-off, the computer, the track, car and marker size, and the data policy.

**Caveats.** Prices change, and the figures here are estimates. The novelty claims rest on quick searches, not a systematic review. Scores are judgement. The 2022 to 2024 data does not reflect the 2026 rules. Results from small RC cars do not transfer directly to full-size cars.

# 2 Scope, assumptions and method

## 2.1 Scope

In scope: one graduation project in the F1 domain, with a physical build, using the team's FastF1 data and code.

Out of scope: full-size cars, live F1 data beyond public timing data, and commercial products.

## 2.2 Planning assumptions

These are planning assumptions. Section 13.1 records the answers received so far.

- Seven people, split by role into groups that share one rig (D2, answered; Section 9.1).
- A six-month project (D4, answered).
- A build budget of 500 to 1,000 USD (D3, answered).
- One shared computer for the build, plus a laptop for each person.
- Floor space for a 1.8 m by 3.0 m track, about 3 m of clear height for the overhead camera mount, power, and lockable storage (D7).
- A named supervisor who signs off scope and safety (D5).

## 2.3 How the scores work

Section 7 scores each idea from 1 to 5 on the criteria listed. Each row says which direction is better. The scores are the author's judgement from the evidence in Section 3. They are not measurements.

## 2.4 Evidence standard

Factual claims that come from sources carry a number in square brackets. The full entry is in Appendix E. Where the evidence is thin, the text says "gap to verify". Prices are listing prices seen during research, and some are from June 2026. Check them before ordering.

# 3 Background

## 3.1 Timing vocabulary: GAP and INTERVAL

F1 timing screens show two time columns. GAP is the time back from the race leader. INTERVAL is the time back from the car directly ahead [1]. FLOW RACERS describes the interval as the time between one car crossing a timing point and the next car crossing the same point [2][3].

A fan site claims that timing loops sit every 150 to 200 m [2]. We have not checked that claim against an official source, and nothing in this dossier depends on it.

## 3.2 The 2026 energy rules

The 2026 power unit sets the energy constraints that Idea 2 copies. The figures below come from F1 Briefing [4] and TopGearbox [5].

- The MGU-K delivers up to 350 kW. The combustion engine is about 400 kW. The MGU-H has been removed [4].
- The usable battery window is 4 MJ (about 1.11 kWh). The per-lap limits are 8.5 MJ of deployment and 9 MJ of recovery [4].
- After the opening events, the qualifying recharge limit was cut from 8 MJ to 7 MJ, and peak superclip power was raised from 250 kW to 350 kW [5].
- DRS is replaced by Overtake Mode. A car within one second of the car ahead can deploy extra electrical energy at a detection point. Active aero adds straight (X) and corner (Z) modes [6].

For this project the important point is that energy is a per-lap budget. A scaled version of that budget can be applied to a small car (Section 5).

## 3.3 What the team already has

- `Data/raw/master_raw_laps.csv`: 57,813 laps from 2022 to 2024, 37 columns, 22.5 MB, tracked in Git.
- `Data/features.csv`: 55,040 rows and 22 columns (5.8 MB), including TyreLife2, compound indicators and LapTimeDelta.
- `Models/Hazem/Trail.ipynb`: an ISLP-style notebook with 12 code cells. It ends at a residual plot and has no prediction or simulation cell yet.
- `Models/Abdelazeem/Index.py`: a toy regression on eight hand-typed rows. The other five person folders hold empty placeholders.

The notebook leads to two conclusions. First, the tyre signal is hidden under driver effects and probably fuel load, so the degradation model needs controls before anyone relies on it. Second, the same habit of controlling and calibrating carries into Idea 2. There, the energy model must be calibrated on logged laps before a DP policy can be trusted.

Repository hygiene (not changed; a team decision): README.md is UTF-16 LE with a byte-order mark, so Git treats it as binary. It describes `src/`, `tests/`, `notebooks/`, `requirements.txt` and `.gitignore`, none of which exist yet. None of this affects the ideas, but it will confuse new team members.

## 3.4 Prior art and gaps

Software-only strategy simulators already exist, for example Race-Strategy-Simulator [12], TUM FTM's race-simulation [13], pit-stop-simulator [14] and f1-race-strategy-simulator [15]. They model strategy without any hardware.

Game telemetry is a route from F1-like data to physical output. The F1 25 UDP specification lists car telemetry, car status, car damage and motion data, including tyre wear and ERS fields [7]. An ESP32 parser for the 2026 Season Pack exists [8]. Packet formats change between game editions, so any parser must be pinned to one specification version.

Physical timing has several existing options:

- A club timing system reportedly cost about AUD 25,000, with much of that cost for 60 transmitters [20].
- EasyRaceLapTimer is an FPV-oriented IR timer with its own transponder design and 63 IDs [16].
- laptrack.app is a free, open-source camera lap timer for RC cars [17].
- rc-lap-timer (MIT licence) times one car using motion detection, and its Pi IR mode is in beta [18].
- LapTrap, a Google Play app, claims to time up to 50 cars with ID markers. Its listing shows no running order, gaps or validation [19]. It is the closest match we found.
- Camera-only timing has been studied, and it is weak in crowded areas [21].

Gap to verify: the quick search found no open, on-device, multi-car race-state system. Idea 1's novelty therefore depends on its order, gap and validation logic, and on publishing its evaluation openly. This is a gap to verify, not an established fact.

## 3.5 Racing platforms and possible partners

F1TENTH is a 1/10-scale autonomous racing platform, and 80 to 89 or more universities have used it [33]. A full car with LiDAR costs about 2,500 to 2,800 USD, and LiDAR is about 60% of that. A camera-only build costs about 1,000 USD [34]. LiDAR is therefore out of reach for this budget, and these figures set the cost reference for Idea 3.

Cairo University Racing Team (CURT) had about 120 students in 2022 [35][36]. It could be a partner, or an audience for a demo. We have not approached the team.

## 3.6 Hardware facts used in this dossier

- Raspberry Pi 5, 8 GB: listed at USD 174.99 at Micro Center in June 2026 [29].
- Raspberry Pi AI HAT+: USD 70 for 13 TOPS and USD 110 for 26 TOPS at launch [25]. Retailers listed the 13 TOPS version at about USD 77 to 84 [26][27].
- Raspberry Pi Global Shutter Camera (IMX296): 1456 × 1088 pixels at 60 fps. About USD 55 at PiShop, lens not included [27]. The Pi Hut listed it at GBP 48 on sale [28].
- NVIDIA Jetson Orin Nano Super: USD 249 at launch, with 67 INT8 TOPS [30]. SparkFun listed it at USD 399 [26].
- ESP32: dual-core, 240 MHz, 520 KB SRAM [31].
- YOLOv8n object detection: about 2.1 to 2.6 fps on the Raspberry Pi 5 CPU, against 60.8 fps on a Hailo-8 accelerator, in a 2026 embedded benchmark [22].
- A 2025 Scientific Reports study found quantisation the most effective compression method for MCU models among those it compared [32].

## 3.7 What the evidence does not show

- We have no measurements yet from our own car or track.
- The FastF1 data covers the 2022 to 2024 rules, not 2026.
- Prices come from listings and will change.
- The prior-art search was quick. A full literature review is still needed.


# 4 Idea 1: RaceState

## 4.1 Summary

RaceState tracks four to six small cars on a loop with one overhead camera. A Raspberry Pi 5 reads a marker ID on each car, records line crossings, and publishes a live timing tower: running order, gap to the leader, interval to the car ahead, and laps completed. IR gates and tags give an independent record of crossing times for checking.

## 4.2 The F1 link

The outputs are the quantities a timing screen shows: running order, GAP and INTERVAL (Section 3.1). RaceState computes them from physical cars, so the same logic can be checked against real F1 timing.

The team's FastF1 data supports a replay test. Feed the lap-end timeline of a 2024 race into the state engine, and compare the computed order with the official position data [9]. This tests the engine against real timing before any hardware exists.

## 4.3 Scope and architecture

In scope for the MVP: one overhead camera, one timing line, four to six cars with ID markers, one IR gate pair for ground truth, and a live web page on the Pi. The camera mount needs about 3 m of clear height (D7).

Out of scope for the MVP: a second camera, sector times, pit detection and any sensors on the cars.

Stretch goals: a second timing line for sector splits, and a link to Idea 2's energy model to give each car a virtual energy state.

### Architecture

1. **Camera.** A global-shutter sensor (IMX296, 1456 × 1088 at 60 fps) with fixed exposure and focus. It is mounted about 2.5 m above the track centre, looking straight down, with the loop's long side along the 1456-pixel axis.
2. **Capture.** Frames are timestamped as they arrive. Dropped frames are counted and logged.
3. **Timing path (every frame, classical).** Blob detection on marker colours, decoding of the marker ID, nearest-neighbour tracking with a constant-velocity motion model, and a line-crossing test on each frame. This path must keep up with the frame rate on the Pi 5 CPU.
4. **Validation path (slower, optional).** A small CNN detector runs every few frames. It produces labelled data and cross-checks the timing path. At about 2 fps on the Pi 5 CPU [22], it cannot run on the timing path without an accelerator.
5. **Race-state engine.** Consumes crossings and maintains order, laps and gaps. Writes a log of every event.
6. **IR ground truth.** Two IR gate receivers, one at the timing line and one further round the loop, record the passing of modulated IR tags on each car. Each gate is an ESP32 with its own clock, 38 kHz demodulation, and a shroud against sunlight.
7. **Output.** A local web page on the Pi shows the timing tower and updates five to ten times per second.

Design decision. Classical detection on the timing path keeps the live system within the CPU budget. Deep learning is used for validation, the labelled dataset and the error analysis. The machine-learning work stays central without making live timing depend on a slow model. If the team wants a CNN on the timing path, the AI HAT+ (13 TOPS) is the option. Decision D6 covers this.

## 4.4 Method

### Gaps and intervals

Each car's crossings are stored with their lap numbers. Cars are ordered by completed laps, then by the time of their most recent crossing. When car c completes lap k:

- Gap to leader: if the leader has also completed k laps, the difference between the two crossing times for lap k. Otherwise, the number of laps down.
- Interval: the same comparison with the car directly ahead.

This follows the GAP and INTERVAL definitions in Section 3.1, with the lap-down case shown explicitly. The lap-down display is a design choice, and the replay test in Section 4.2 will check it against official timing screens. Appendix C.1 gives the procedure.

### Camera geometry and marker size

| Lens | Horizontal field of view | Field width at 2.5 m | Field height at 2.5 m | Pixels per cm | Pixels across a 5 cm marker |
|---|---|---|---|---|---|
| 4 mm | about 64° | about 3.1 m | about 2.3 m | about 4.7 | about 23 |
| 6 mm | about 45° | about 2.1 m | about 1.6 m | about 7.0 | about 35 |
| 8 mm | about 35° | about 1.6 m | about 1.2 m | about 9.3 | about 47 |

These numbers come from a pinhole model with a 5.0 mm by 3.75 mm sensor (1456 by 1088 pixels), with the camera 2.5 m above the track. Lens distortion and the real mount height will change them, so measure them on the rig before fixing the design. Pixel density depends only on the ratio of focal length to camera height, so the table applies at other heights if that ratio is kept.

The default is a 4 mm lens at 2.5 m. The field is about 3.1 m by 2.3 m, enough for the 3.0 m by 1.8 m loop in D7, with about 7 cm of margin on the long sides. A 5 cm marker is then about 23 pixels across. That is a working assumption to test in the first two weeks. A 6 mm lens gives 35 pixels per marker, but its field is only 2.1 m by 1.6 m, too small for a 3 m loop at this height. An 8 mm lens gives the most detail, but its field is smaller still.

At 3 m/s (about 11 km/h), a car moves 5 cm between frames at 60 fps, which is the width of a marker. Tracking therefore needs a motion model, and the timing resolution is one frame, about 17 ms.

## 4.5 Milestones (six-month version)

- Month 1: mount the camera, record first videos, and count laps for one car. Target: a one-car lap count by week 4.
- Month 2: marker design and ID decoding with four cars; first labelled frames.
- Month 3: tracking and order; IR gates and tags; first crossing comparison.
- Month 4: gaps and intervals; live web tower; lap-down logic.
- Month 5: evaluation runs (20 runs of 10 laps); second lighting setup; error analysis.
- Month 6: report, demo and handover, with a buffer for slips.

## 4.6 Evaluation

### Ground truth and targets

Camera crossings are timestamped at frame resolution: 16.7 ms at 60 fps. The IR gate timestamps each tag pass on its own clock, to millisecond scale. Comparing the two gives an independent error estimate for each crossing.

Proposed targets, to be confirmed in weeks 1 and 2 and not yet tested:

- Median absolute difference between camera and IR crossing times under 30 ms.
- Lap-count error of zero on at least 95% of laps.
- Correct order on at least 95% of crossings in clean runs.
- Median gap error under 0.1 s when cars are at least 0.5 s apart.

### Evaluation plan

Section 12 sets the general rules. For Idea 1 the plan is:

1. Tracking: label at least 500 frames across at least five sessions. Report HOTA and IDF1, and count identity switches [23]. ByteTrack's own benchmarks report IDF1 gains of about 1 to 10 points over nine other trackers [24], so the association method is worth comparing.
2. Timing: at least 20 timed runs of 10 laps each, with four to six cars, under two lighting conditions. Compare each crossing with the IR gate.
3. Order and gaps: compare the live tower with the IR-based order at each crossing. Report accuracy and the error distribution.
4. Replay: run the FastF1 replay test from Section 4.2 [9].
5. Stress tests: side-by-side passes, a car stopped on the line, and a marker damaged during a run.

### What counts as a useful result

A working tower with measured accuracy is a result. So is a measured limit, for example: "marker decoding fails above four cars at 60 fps with the default 4 mm lens". Both are worth publishing if the protocol is sound and the limits are reported honestly.

## 4.7 Risks specific to Idea 1

- Identity switches when cars are close or overlapping (R-01).
- Lighting changes and shadows (R-02).
- Dropped frames and timing jitter (R-03).
- Pi 5 CPU limits on the timing path (R-04).
- False IR triggers (R-05).

Mitigations are in Section 10.

## 4.8 Assessment

Strengths: strong machine-learning and computer-vision content (5), high measurability (5), low safety risk (2), and an early first milestone.

Weaknesses: moderate embedded depth (3), and partial overlap with LapTrap [19], so novelty depends on the validation and on the open evaluation.


# 5 Idea 2: Learn-to-Deploy

## 5.1 Summary

A small RC car, controlled by an ESP32, runs laps under a virtual energy budget modelled on the 2026 rules. An offline DP solver chooses how much power to deploy on each track segment for a given budget. The ESP32 follows a compact lookup table built from that solution. A learned model is tested as an alternative to the table, and the project reports which one does better.

Version one uses a deploy-only budget. Recovery, or regenerative braking, needs an electronic speed controller (ESC) that supports braking current, which we do not assume. If the chosen ESC supports it, recovery can be added in version two.

## 5.2 The F1 link

Under the 2026 rules the electric motor can deliver 350 kW against about 400 kW from the combustion engine, close to an even split. Energy is also a per-lap resource: 8.5 MJ of deployment and 9 MJ of recovery per lap [4]. Overtake Mode lets a car within one second of the car ahead deploy extra energy at a detection point [6]. Energy management is therefore a real strategic question.

Prior work supports the approach. Limebeer, Perantoni and Rao used optimal control to study KERS and reported an optimal lap of 80.23 s, 0.27 s faster than without KERS [10]. Javed and Samuel compared rule-based, genetic-algorithm and DP energy strategies in a GT-Suite powertrain model. Their DP strategy kept the speed target within 1% and deployed about 72% less energy than their rule-based baseline. Both results come from simulation [11].

The team's regression work carries over. The car and energy models are fitted on logged laps, and they need the same care about confounders that the tyre notebook needs. On the car, battery state plays the role that fuel load plays in the tyre analysis (Section 3.3).

## 5.3 Scope and architecture

- **Car.** A small RC car with an ESC, motor and servo.
- **Controller.** An ESP32 that sets the motor power through the ESC. It runs at 240 MHz on two cores, with 520 KB of SRAM [31].
- **Energy sensing.** A 2S LiPo pack, with an INA219 or INA226 sensor measuring voltage and current.
- **Track sensing.** A wheel encoder for distance, and an IR gate pair for lap start and finish.
- **Track.** A 1.8 m by 3.0 m loop, split into 24 to 40 segments.
- **Laptop.** The DP solver and the model fitting.

### Why a table first

A table for 24 to 40 segments, with 20 speed bins and 20 energy bins, holds 9,600 to 16,000 one-byte entries, about 10 to 16 KB. That fits easily in the ESP32's memory, and a lookup is a few array reads, so it takes microseconds. A learned model is only worth deploying if it beats the table on conditions the table never saw. An energy lookup table may well beat a learned model on a microcontroller. That would be a valid result.

## 5.4 Method

1. **Track model.** Split the track into 24 to 40 segments. Each has a length and an estimated curvature, taken from a reference lap using wheel-encoder distance and IR gate timing.
2. **Car model.** A longitudinal model: motor force minus drag and rolling resistance. Its parameters are fitted to logged speed and current.
3. **Energy model.** Battery energy from measured voltage and current, converted to joules per segment.
4. **DP solver (laptop).** State: segment, speed bin and energy bin. Action: motor power level. Cost: segment time. Constraint: energy never falls below zero within the lap. The solver works backwards from the end of the lap (Appendix C.2).
5. **Lookup table (ESP32).** The policy is indexed by segment and energy bin and returns a quantised power level. Out-of-range states fall back to a safe default.
6. **Learned alternative.** A small regression model is trained on logged states to predict the DP action. It is compared with the exact table on laps it has not seen.

Budget setting. Choose the per-lap budget as a fraction of the energy a constant-power lap uses, for example 70% to 90%. A budget that is too loose leaves the DP nothing to decide. One that is too tight makes the lap infeasible at the speeds the track demands.

## 5.5 Milestones (six-month version)

- Months 1 and 2: car, sensors and logging; baseline laps.
- Month 3: track and car models fitted on validation laps.
- Month 4: DP solver and first table; simulation results.
- Month 5: ESP32 deployment; first budgeted runs on the track.
- Month 6: evaluation, write-up and demo.

## 5.6 Evaluation

- Budget violations: zero across all evaluation laps.
- Lap time against three references: the DP solution (the model's upper bound), constant power (the rule-based baseline), and a simple speed-limit controller.
- Energy-model error: mean absolute error per lap, in watt-hours, on held-out laps.
- Robustness: runs from full and half charge, and at two budget levels.
- Embedded figures: flash and RAM used, lookup time, and loop jitter in milliseconds.

## 5.7 Risks specific to Idea 2

- Battery damage or fire (R-07).
- Motor or ESC overheating (R-08).
- Energy-model mismatch with the real car (R-09).
- A simple baseline matching DP on a short track (R-10).
- The 2022 to 2024 data does not reflect the 2026 rules (R-16).

## 5.8 Assessment

Strengths: the most F1-specific idea (5), the strongest embedded depth (5), and a clear comparison between an exact policy and a learned one.

Weaknesses: the highest safety risk of the three (4), heavy hardware work including battery handling, and less machine-learning depth (4).


# 6 Idea 3: Learned racing (GT-Sophy style)

## 6.1 Summary

Train a driving policy with reinforcement learning in simulation, run it on a small car, and compare it with two tuned classical controllers: pure pursuit and model predictive control (MPC). The question is whether a learned policy matches tuned controllers on lap time and robustness, and how much of the simulated performance survives the move to hardware.

## 6.2 The F1 link

This idea is about driving, not strategy. Wurman et al. describe agents for Gran Turismo, trained with deep reinforcement learning, that compete with the world's best e-sports drivers [37]. The F1 link is racing lines, overtaking and defence. It is weaker than the links in Ideas 1 and 2, which use F1 timing and energy data directly.

## 6.3 Scope and architecture

- **Car.** One 1/16 to 1/24 scale RC car, plus a spare if the budget allows. It carries a Raspberry Pi 5, an onboard camera module, an IMU and wheel encoders.
- **Track.** A 1.8 m by 3.0 m loop.
- **Ground truth.** The overhead camera from Idea 1 if the team has it. Otherwise, a simpler marker-based pose system (D7).
- **Compute.** Training on a laptop. Onboard inference on the Pi 5 CPU, which is enough for a small network at 20 to 50 Hz. A Jetson Orin Nano Super (67 INT8 TOPS [30]) is an option if the Pi cannot keep up with the control rate.

### Compute and time

Small networks need many simulated steps, which means hours to days of training. This fits a laptop, with a GPU if one is available. Set a fixed compute cap before training starts (Section 12). Hardware time is short. Most of the effort goes into simulation work and tuning.

## 6.4 Method

- **Simulator.** A kinematic or dynamic bicycle model, fitted to logged runs of the real car (system identification).
- **Observation.** During training, the simulator supplies the true pose and track geometry. On the car, the pose comes from the overhead camera, or the lateral offset comes from the onboard camera. Speed and steering angle are included in both cases.
- **Action.** Steering and throttle.
- **Reward.** Progress along the track, minus penalties for leaving the track and for abrupt steering changes. A crash ends the episode.
- **Algorithms.** PPO and SAC are the two standard options. Compare them in simulation before any hardware run.
- **Domain randomisation.** Friction, mass, motor gain, control latency and camera noise vary during training.
- **Baselines.** Pure pursuit with a tuned lookahead distance, and MPC with the same bicycle model and tuned weights. Both are tuned on training runs and tested on held-out runs.

Pure pursuit steering is δ = atan2(2 · L · sin α, L_d), where L is the wheelbase, α is the angle between the heading and the lookahead point, and L_d is the lookahead distance. Appendix C.4 gives the procedure.

## 6.5 Milestones (six-month version)

- Months 1 and 2: simulator, and baseline controllers (pure pursuit and MPC) running in simulation.
- Month 3: car platform and logging on the track; system identification.
- Month 4: first RL policy in simulation; PPO and SAC compared.
- Month 5: sim-to-real tests on the track, with domain randomisation.
- Month 6: held-out evaluation, write-up and demo.

## 6.6 Evaluation

- Lap time, mean and best, with its spread over at least 20 laps per controller.
- Crashes and off-track events per 100 laps.
- Sim-to-real gap: the lap-time difference for the same controller in simulation and on the real track.
- At least five training seeds for the learned policy. Paired comparisons on identical track conditions, with confidence intervals.
- A stop rule: if the policy is still slower than tuned pure pursuit after the fixed budget, report that as a negative result and analyse why.

## 6.7 Risks specific to Idea 3

- Training does not converge (R-11).
- The sim-to-real gap is too large (R-12).
- A crowded topic weakens novelty (R-13).
- Crashes damage cars (R-06, R-14).
- The number of moving parts is high, so scope creep is likely.

## 6.8 Assessment

Strengths: the strongest CS, AI and ML depth (5), and the most visible demo (5).

Weaknesses: the lowest F1 authenticity (3), the most crowded field (freedom from crowding 2), the longest time to a working MVP, and the highest chance of a negative result (4).


# 7 Side-by-side comparison

Scores are the author's judgement on a 1 to 5 scale. The direction that is better is stated in each row's label.

| Criterion | 1. RaceState | 2. Learn-to-Deploy | 3. Learned racing |
|---|---|---|---|
| F1 authenticity (higher = better) | 4 (live timing, gaps) | 5 (2026 energy budget) | 3 (driving, not strategy) |
| CS, AI and ML depth (higher = better) | 5 | 4 | 5 |
| Embedded depth (higher = better) | 3 | 5 | 3 |
| Build complexity (higher = harder) | 3 | 4 | 4 |
| Safety risk (higher = riskier) | 2 | 4 | 4 |
| Cost band, with contingency (planning) | About 450 to 1,000 USD (CPU only) | About 290 to 600 USD | About 360 to 850 USD (Pi-based); about 250 USD more with a Jetson |
| Time to a working MVP (shorter = better) | Medium | Medium | Long |
| Freedom from crowding (higher = less crowded) | 4 (gap in quick search; to verify) | 4 (gap in quick search; to verify) | 2 (crowded) |
| Measurability, ground truth and metrics (higher = better) | 5 | 4 | 4 |
| Demo appeal (higher = better) | 4 | 3 | 5 |
| Impact beyond the project (higher = better) | 4 (clubs, schools, tools) | 3 (teaching, research) | 3 (education) |
| Risk of no clear result (higher = more likely) | 2 | 3 | 4 |

**How to read this.** Idea 1 is the most balanced and the safest. Idea 2 is the most F1-specific, but it carries battery and embedded risk. Idea 3 is the most exciting to watch, but it has the most competition and the highest chance of a negative result.

### Which idea fits which team

- Strong in machine learning and computer vision, and comfortable with Python: Idea 1.
- Strong in embedded or control systems, with some optimisation background: Idea 2.
- Strong in reinforcement learning and simulation, and a team that accepts crashes and negative results: Idea 3.
- A mixed team: Idea 1, with the embedded lead also owning the IR timing.

# 8 Recommendation and decision path

## 8.1 Recommendation

Choose Idea 1 (RaceState) as the default. Its first milestone is reachable early. Its ground truth is clear and independent, its safety risk is low, and its machine-learning and computer-vision content is substantial and measurable. Its F1 link is direct: order and gaps are what timing screens show.

Choose Idea 2 if the team is strongly embedded, wants the most F1-specific project, and accepts battery handling and hardware risk.

Choose Idea 3 only if the team has real reinforcement-learning experience, is comfortable with crashes and possible negative results, and wants the most visible demo.

## 8.2 Decision path

1. Hold a discussion to settle D1 (the idea) and D2 (roles), using the fit list in Section 7.
2. If the team cannot decide, run two-week spikes before committing. With seven people, the two spikes can run in parallel, one per group. For Idea 1, count laps for one car from camera frames and compare them with one IR gate. For Idea 2, log one lap of battery voltage and current on the car.
3. Pick the idea with the clearer spike result and the better team fit. Record the choice and the reason in the project log.
4. Get supervisor sign-off (D5) before buying parts.


# 9 Team, budget and timeline

## 9.1 Team and roles

The team is seven people, the answer to D2. The repository has seven person folders: Abdelazeem, Abdulrahman, Eman, Halim, Hazem, Noor and Osama. One rig can serve all seven if the work is split by role, as set out below the table.

| Role | Idea 1 | Idea 2 | Idea 3 |
|---|---|---|---|
| Lead for vision, modelling or learning | Detection, tracking and ID decoding (one or two people) | Track and car model fitting | Simulator and RL training (one or two people) |
| Lead for hardware and timing | Camera rig, IR gates, Pi setup | Car, battery, ESC and ESP32 firmware | Car platform, Pi and sensors |
| Data and evaluation | Labelling, metrics and replay test | DP solver and energy-model error | Baselines, statistics and sim-to-real analysis |
| Integration, interface and report | Live tower, logs and report | Table export, logging and report | Integration, demo and report |

Suggested split for seven, if Idea 1 is chosen. Vision and tracking (three people): detection, tracking, ID decoding, the labelled dataset and the live tower. Hardware and timing (two people): the camera rig, IR gates and tags, the Pi setup and the track. Data, evaluation and report (two people): the replay test, metrics and statistics, budget tracking and the report. Put the integration row with the vision group and the report row with the data group.

Two separate rigs, for two ideas, would need 620 to 1,345 USD in subtotals alone. That is beyond the 500 to 1,000 USD band (D3), so the team should build one idea and share the rig.

## 9.2 Budget

| Idea | Subtotal (USD) | With contingency (USD) | Contingency | Notes |
|---|---|---|---|---|
| 1. RaceState | 390 to 865 | about 450 to 1,000 | 15% | CPU only. An AI HAT+ adds 70 to 110. |
| 2. Learn-to-Deploy | 230 to 480 | about 290 to 600 | 25% | Battery and crash spares drive the top of the range. |
| 3. Learned racing | 275 to 650 | about 360 to 850 | 30% | Add about 250 for a Jetson Orin Nano Super. |

Budget bands (D3) are under 500 USD, 500 to 1,000 USD, or over 1,000 USD. The answer so far is 500 to 1,000 USD. Idea 1 needs the 500 to 1,000 band for most of its range. Idea 2 can fit under 500 USD. Idea 3 fits either band, depending on the car and on whether a Jetson is added.

Line items are in Appendix B. The biggest cost risks are LiPo packs and chargers (Idea 2), replacement cars after crashes, IR tags, the lens choice, and shipping and import duties.

## 9.3 Timeline

The six-month plan for Idea 1 fits in about 26 weeks, with milestones in Section 4.5. Ideas 2 and 3 follow the same shape, with milestones in Sections 5.5 and 6.5.

| Month | Work | Output |
|---|---|---|
| 1 | Order parts in week 1. Mount the camera. Record first videos. Count laps for one car. | One-car lap count by week 4 |
| 2 | Marker design and ID decoding. Four cars. First labelled frames. | Labelled dataset v1 |
| 3 | Tracking and order. IR gates and tags. First crossing comparison. | Accuracy report v1 |
| 4 | Gaps and intervals. Live web tower. Lap-down logic. | Live tower demo |
| 5 | Evaluation runs (20 runs of 10 laps). Second lighting setup. Error analysis. | Results tables |
| 6 | Report, demo and handover. Buffer for slips. | Final report and demo |

A nine-month project adds three months. Months 7 and 8 cover extended evaluation: more runs, a second track layout and a second lighting setup. Month 9 covers an accelerated timing path if the AI HAT+ is chosen, the write-up and a public demo. The buffer absorbs slips throughout.

# 10 Risk register

L is likelihood and I is impact, each rated H, M or L. "All" means the risk applies to every idea.

| ID | Risk | Idea | L | I | Mitigation |
|---|---|---|---|---|---|
| R-01 | Identity switches when cars pass close together or overlap | 1 | H | H | Large high-contrast markers; track width and car spacing rules; an ID confidence threshold; report switches on labelled clips |
| R-02 | Lighting changes and shadows degrade detection | 1 | M | H | Fixed LED lighting; locked exposure; matte track surface; test two lighting setups |
| R-03 | Dropped frames or timestamp jitter | 1 | M | H | Log frame timestamps and drops; set a drop limit for valid runs; use a hardware trigger if available |
| R-04 | Pi 5 CPU too slow for the timing path | 1 | M | M | Classical timing path; CNN for validation only; AI HAT+ as an option (D6) |
| R-05 | IR gate false triggers from sunlight or other IR sources | 1, 2 | M | M | 38 kHz demodulation; shrouds; indoor tests; threshold checks before each session |
| R-06 | Cars leave the track or collide | 1, 2, 3 | H | M | Speed caps; walls or bumpers; supervised runs; spare cars |
| R-07 | LiPo damage, swelling or fire | 2, 3 | L | H | LiPo bag; balance charger; storage charge; fuse and alarm; extinguisher; no unattended charging |
| R-08 | Motor or ESC overheating under a DP profile | 2 | M | M | Power and duty-cycle caps; temperature logging; cool-down breaks |
| R-09 | Energy model does not match the real car | 2 | H | M | Calibrate on logged laps; report the model error; fall back to a constant-power baseline |
| R-10 | A simple baseline matches DP on a short track | 2 | M | M | Report it as a result; test more track shapes and budget levels |
| R-11 | RL training does not converge within the compute budget | 3 | H | H | Start in simulation with baselines; curriculum; compute cap; stop rule |
| R-12 | Sim-to-real gap is too large | 3 | H | H | System identification; domain randomisation; early hardware tests |
| R-13 | A crowded topic weakens novelty | 3 (partly 1) | H | M | Frame as a controlled comparison; cite prior art; state a specific question |
| R-14 | Budget overrun from crashes, spares or shipping | All | M | M | Contingency of 15 to 30%; spares list; order in week 1 |
| R-15 | Parts arrive late and delay the schedule | All | M | H | Order in week 1; do software work (replay test, simulator) while waiting |
| R-16 | The 2022 to 2024 data does not represent the 2026 rules | 2 (and 1 for the replay) | H | M | Treat 2026 as a scenario, not as data; state the rule era in every result |
| R-17 | Uneven workload or availability within the team | All | M | H | Named roles; weekly plan; shared log; agreed fallback scope |
| R-18 | Supervisor or lab access delays | All | M | M | Book track space and get sign-off (D5) in the first two weeks |


# 11 Safety

Safety rules apply to every idea. They are stricter for Ideas 2 and 3 because of the batteries.

## 11.1 Rules for every session

- A named supervisor approves each session and the safety plan (D5).
- Everyone who handles a car has had the safety briefing.
- Power the car off before touching it. Keep hands, hair and loose clothing away from the wheels.
- Run only on the marked track, with the car's speed capped.
- Keep a first-aid kit and a working fire extinguisher in the room.
- Log every incident, including near misses.

## 11.2 Batteries (Ideas 2 and 3)

- Use only LiPo packs with a matching balance charger, set to the pack's chemistry and cell count.
- Charge inside a LiPo bag, on a non-flammable surface, and never unattended.
- Store packs at storage voltage. Do not use a pack that is swollen, punctured or hot.
- Put a fuse and a low-voltage alarm on each battery path. Stop the run when the alarm sounds.
- Keep a metal or ceramic container near the charging area for damaged packs.

## 11.3 Moving cars

- Keep speeds low during tests. Cap motor power in software and in the ESC.
- Fit a physical stop switch to each car that cuts the motor.
- Use track walls or bumpers. Nobody reaches onto the track while a car is powered.

## 11.4 Electrical and IR hazards

- Keep bench supplies at 12 V or less. Do not work on mains power at the track.
- Drive the IR emitters at normal currents. Do not look into an IR emitter at close range.
- Secure cables and LED panels to avoid trips.

## 11.5 Camera and data privacy

- Aim the camera at the track only. Do not record bystanders. If people appear in footage, crop or blur them before storing.
- Follow the data policy in D8 for storage, sharing and retention.

## 11.6 Sign-off

The supervisor signs off the safety plan before the first powered run. Re-sign it when the battery type, the track or the number of cars changes.

# 12 Evaluation standards

## 12.1 Principles

- Define success criteria and the stop rule before the first measured run.
- Tune on separate runs. Keep a held-out test set that is never used for tuning.
- Report every run, including failures.
- Fix random seeds, and record software versions and data hashes.
- Report uncertainty: confidence intervals, and paired comparisons on the same laps or conditions.
- Treat a negative result as a result, and explain it with evidence.

## 12.2 Metrics by idea

| Idea | Primary metrics | Secondary metrics |
|---|---|---|
| 1. RaceState | Lap-count accuracy. Order accuracy per crossing. Gap error (median and 95th percentile) against IR. | Identity switches per 100 crossings. HOTA and IDF1 on labelled clips. Latency. Dropped frames. |
| 2. Learn-to-Deploy | Budget violations (target zero). Lap time against DP and baselines. | Energy error per lap (Wh). Table size (KB). Lookup time (microseconds). Loop jitter (ms). |
| 3. Learned racing | Lap time, mean and best, against tuned pure pursuit and MPC. | Crashes and off-track events per 100 laps. Sim-to-real lap-time gap. Success across seeds. Compute used. |

## 12.3 Statistical reporting

Use at least five training seeds for any learned policy. Report 95% confidence intervals from a bootstrap over laps or runs. Compare controllers on paired conditions. Do not choose a result after looking at the test set.

# 13 Decisions

## 13.1 Decisions (D1 to D8)

| ID | Decision | Options | Suggested default | Needed by |
|---|---|---|---|---|
| D1 | Which idea | 1 RaceState; 2 Learn-to-Deploy; 3 Learned racing | 1 RaceState | Before buying parts |
| D2 | Team size and roles | 3, 4 or 5 people; or a split if all seven join | 4 people, with the roles in Section 9.1 | Week 0 |
| D3 | Budget band | Under 500 USD; 500 to 1,000 USD; over 1,000 USD | 500 to 1,000 USD | Before buying parts |
| D4 | Project length | Six months; nine months | Six months, with a review at month 4 about extending | Week 0 |
| D5 | Supervisor sign-off | Scope; safety plan; budget | All three, in week 2 | Week 2 |
| D6 | Computer | Pi 5 alone; Pi 5 with AI HAT+; Jetson Orin Nano Super | Pi 5 alone; AI HAT+ only if the timing path needs it | Before buying parts |
| D7 | Track, car and marker size | Track 1.8 × 3.0 m, or 2.4 × 4.0 m with a camera about 3.5 m high and 8 cm markers; cars 1/24 to 1/16; marker 5 cm by default | 1.8 × 3.0 m track, 5 cm markers, 4 mm lens at 2.5 m | Week 2 |
| D8 | Data policy | What is stored, for how long, who sees footage, what is published | Keep raw footage for the project only. Publish code and aggregate results. Check data terms before publishing any raw data | Week 2 |

**Answers recorded on 8 October 2026.** D1: not decided, so run the two-week spikes in Section 8.2 first. D2: seven people, split by role (Section 9.1). D3: 500 to 1,000 USD. D4: six months.

## 13.2 Next steps after the decisions

1. Week 1: order the spike parts and run both spikes in parallel (Section 8.2). Settle D1 from the results, then order the parts for the chosen idea within the 500 to 1,000 USD band (D3).
2. Week 2: supervisor sign-off (D5), the safety plan, the track layout (D7) and the data policy (D8).
3. Weeks 3 and 4: the first one-car lap count (Idea 1), the first logged lap (Idea 2), or the first simulator run (Idea 3).
4. Week 4 review: keep, narrow or switch the idea.


# Appendix A: Glossary

| Term | Meaning |
|---|---|
| AI HAT+ | Raspberry Pi add-on board with a neural-network accelerator, sold in 13 and 26 TOPS versions [25]. |
| Active aero | Movable front and rear wings that switch between straight (X) and corner (Z) modes in 2026 cars [6]. |
| DP | Dynamic programming: solves a multi-step decision problem by working backwards from the end. |
| DRS | Drag reduction system: the rear-wing flap used until 2025, replaced in 2026 by Overtake Mode [6]. |
| ERS | Energy recovery system: the hybrid system that harvests and deploys electrical energy. |
| ESC | Electronic speed controller: drives the motor from the battery. |
| ESP32 | Microcontroller board: dual-core, 240 MHz, 520 KB SRAM [31]. |
| FastF1 | Python package for F1 timing, results and telemetry [9]. |
| GAP | Time back from the race leader on a timing screen [1]. |
| GT Sophy | The reinforcement-learning agent for Gran Turismo described by Wurman et al. [37]. |
| HOTA | Higher Order Tracking Accuracy: a multi-object tracking metric [23]. |
| ID switch | An event in which a tracker swaps the identities of two objects. |
| IDF1 | Identity F1 score: a tracking metric that rewards keeping identities right [23]. |
| IMU | Inertial measurement unit: an accelerometer and gyroscope in one package. |
| INTERVAL | Time back from the car directly ahead on a timing screen [1]. |
| IR gate | An infrared receiver that timestamps a passing IR tag. |
| IR tag | An infrared emitter fitted to a car, modulated so that a gate can tell it apart from sunlight. |
| Jetson Orin Nano Super | NVIDIA's embedded AI computer, with 67 INT8 TOPS in research listings [30]. |
| LiPo | Lithium-polymer battery used in RC cars. It needs special handling (Section 11.2). |
| MGU-H | Heat-recovery motor-generator, removed from the 2026 power unit [4]. |
| MGU-K | Kinetic motor-generator, up to 350 kW in 2026 [4]. |
| MJ | Megajoule. One MJ is about 0.278 kWh. |
| MOTA | Multiple Object Tracking Accuracy: a combined tracking metric. |
| MPC | Model predictive control: chooses the best short sequence of actions at each step, using a model. |
| Overtake Mode | 2026 system that gives a car within one second of the car ahead extra electrical energy at a detection point [6]. |
| Pinhole model | Simple camera model used for field-of-view and pixel-size calculations. |
| Pure pursuit | Geometric path-tracking controller that steers towards a point ahead on the path. |
| Quantisation | Storing model weights with fewer bits, to save memory and compute [32]. |
| RL | Reinforcement learning: learning a policy from a reward signal. |
| Sim-to-real gap | The performance difference between simulation and the real system. |
| Superclip | 2026 mode with a peak electrical power of 350 kW [5]. |
| TOPS | Tera operations per second: a peak throughput figure, not a measure of real speed. |
| TyreLife | The number of laps on the current set of tyres, as used in the notebook. |

# Appendix B: MVP parts lists

These are planning ranges in USD, not quotes. Prices change, so check them before ordering. Each table gives the subtotal before contingency. Contingency is applied in Section 9.2.

## B.1 Idea 1: RaceState

| Item | Low | High | Notes |
|---|---|---|---|
| Raspberry Pi 5, 8 GB | 175 | 175 | June 2026 listing at USD 174.99 [29] |
| Power supply, 64 GB microSD, cooler and case | 25 | 40 | 27 W USB-C supply |
| Global shutter camera module (IMX296), no lens | 50 | 60 | About USD 55 [27]; GBP 48 on sale [28] |
| C or CS-mount lens, 4 to 8 mm | 15 | 50 | Choose from the table in Section 4.4 |
| Overhead mount about 2.5 m high (tall tripod or aluminium boom) | 20 | 60 | Rigid, so vibration does not blur the frame |
| Track surface, 1.8 × 3.0 m to 2.4 × 4.0 m | 25 | 120 | Flat, matte and high-contrast to the cars |
| Four to six small RC or toy cars | 40 | 150 | 1/24 to 1/18 scale; spares at the top of the range |
| IR ground truth: two gates and car tags | 35 | 150 | DIY ESP32 kit at the low end; commercial kit at the top |
| Printed or laminated ID markers | 5 | 15 | 5 cm; test wear during runs |
| Cables, spares, tools and consumables | 0 | 45 | Covers crash losses and tape |
| **Total** | **390** | **865** | Subtotal before 15% contingency |

## B.2 Idea 2: Learn-to-Deploy

| Item | Low | High | Notes |
|---|---|---|---|
| ESP32 boards (car and logger or gate) | 16 | 30 | 240 MHz, 520 KB SRAM [31] |
| RC car chassis with ESC, motor and servo | 70 | 110 | Brushed or brushless, with a programmable ESC |
| 2S LiPo pack, balance charger and LiPo bag | 35 | 60 | The charger must match the pack |
| Current and voltage sensor (INA219 or INA226) | 4 | 10 | Energy per segment from voltage times current |
| Wheel speed sensor and magnets | 5 | 15 | Hall or optical |
| IR start and finish gate pair | 40 | 60 | Shared with Idea 1 if both are built |
| Fuse, switch and low-voltage alarm | 10 | 25 | Battery protection |
| Track surface (1.8 × 3.0 m, or a smaller loop) | 30 | 50 | Same surface as Idea 1 if shared |
| Spares and consumables (tyres, fuses) | 10 | 30 | Crash and wear losses |
| Cables, connectors and miscellaneous | 10 | 20 | Connectors, heat-shrink and small parts |
| IMU (optional) | 0 | 25 | Adds yaw rate for model fitting |
| Bench power meter (optional) | 0 | 40 | Checks energy per lap independently |
| SD card and logging extras (optional) | 0 | 5 | Storage for long logging sessions |
| **Total** | **230** | **480** | Subtotal before 25% contingency |

## B.3 Idea 3: Learned racing

| Item | Low | High | Notes |
|---|---|---|---|
| Raspberry Pi 5, 8 GB | 175 | 175 | As Idea 1 [29] |
| Power supply, microSD, cooler and case | 25 | 40 | As Idea 1 |
| Small RC car platform, 1/24 to 1/16 scale (two units) | 30 | 150 | Two cars, so one can be repaired while the other runs |
| Onboard camera module (CSI) | 10 | 35 | For lateral offset estimation |
| IMU and wheel encoder | 5 | 20 | Pose between camera frames |
| Track surface, 1.8 × 3.0 m | 20 | 80 | Matte surface |
| Battery, charger, fuse and cabling | 10 | 40 | LiPo rules in Section 11.2 |
| Overhead ground-truth camera (reuse the Idea 1 kit or buy) | 0 | 60 | Zero if Idea 1 is built |
| Spares, tyres and consumables | 0 | 50 | Crash and wear losses |
| **Total** | **275** | **650** | Subtotal before 30% contingency. Add about 250 for a Jetson Orin Nano Super [30] |


# Appendix C: Pseudocode

These are sketches for design discussion. They are not tested code.

## C.1 Line crossings and standings (Idea 1)

```
# n[c] = laps completed by car c
# T[c] = crossing times of car c; T[c][1] is its first crossing (1-based)

on_crossing(c, t):                 # t is interpolated between the two frames
    n[c] = n[c] + 1
    append t to T[c]

standings():
    order = cars sorted by (n[c] descending, T[c][n[c]] ascending)
    L = order[1]                   # the leader
    for each car c below the leader:
        k = n[c]
        A = the car directly ahead of c in order
        gap_to_leader = lap_gap(L, c, k)
        interval = lap_gap(A, c, k)

lap_gap(other, c, k):
    if n[other] == k:
        return T[c][k] - T[other][k]           # seconds, on the same lap line
    return "+" + (n[other] - k) + " lap(s)"    # behind by whole laps
```

Because the cars are sorted by completed laps first, the leader and the car ahead always have at least k crossings. The lookups are therefore always defined.

## C.2 Dynamic programming for the energy budget (Idea 2)

```
# Inputs: segments s = 1..S with length d[s] and curvature kappa[s]
#         speed bins V, energy bins E, power levels P (for example 0% to 100% in 10% steps)
# Models: energy_used(v, p, s), speed_after(v, p, s), lap_time(v, v2, s)

J[S+1][v][e] = 0 for every speed bin v and every non-negative energy bin e

for s = S down to 1:
    for each speed bin v and energy bin e:
        J[s][v][e] = infinity
        for each power level p in P:
            e2 = e - energy_used(v, p, s)
            if e2 < 0: continue                  # the budget would be violated
            v2 = speed_after(v, p, s)            # then snap v2 and e2 to the nearest bins
            cost = lap_time(v, v2, s) + J[s+1][snap(v2)][snap(e2)]
            if cost < J[s][v][e]:
                J[s][v][e] = cost
                policy[s][v][e] = p

# Start state: entry speed v0 with the full lap budget. Lap time = J[1][v0][E_full].
# Export the policy as one byte per entry, then check that the table fits in flash.
```

## C.3 Training loop outline (Idea 3)

```
initialise policy pi, critic Q and replay buffer B
for episode = 1 to N:
    s = sim.reset(random_track_and_physics())    # domain randomisation
    for t = 1 to T_max:
        a = pi(s) + exploration_noise
        s2, r, done = sim.step(a)                # r = progress - off_track - steering_change
        B.add(s, a, r, s2, done)
        if len(B) >= warmup:
            update pi and Q from a batch of B    # SAC step, or PPO update
        s = s2
        if done: break
    if episode mod eval_every == 0:
        evaluate pi on fixed held-out seeds; log lap time and off-track count
stop at the compute cap, or when the stop rule in Section 6.6 applies
```

## C.4 Pure pursuit baseline (Idea 3)

```
pure_pursuit(pose, path, L_d):
    p = point on path nearest to pose.position
    g = point on path that is L_d ahead of p, measured along the path
    alpha = angle from pose.heading to the vector from pose.position to g
    delta = atan2(2 * wheelbase * sin(alpha), L_d)
    return clamp(delta, -delta_max, delta_max)
```

# Appendix D: Checklists

## D.1 Before the first powered run (all ideas)

- [ ] Safety plan signed by the supervisor (D5)
- [ ] Stop switch tested on every car
- [ ] Track edges and walls checked; no loose or sharp objects
- [ ] Speed cap set in software and in the ESC
- [ ] First-aid kit and extinguisher in the room
- [ ] Incident log open

## D.2 Battery checks (Ideas 2 and 3)

- [ ] Pack undamaged: no swelling, punctures or heat
- [ ] Charger set to the correct chemistry and cell count
- [ ] Pack charged inside a LiPo bag, and never left unattended
- [ ] Low-voltage alarm set and tested
- [ ] Fuse fitted and rated for the motor
- [ ] Pack returned to storage voltage after the session

## D.3 Data logging

- [ ] Frame timestamps and dropped-frame counts saved with every run
- [ ] IR gate logs saved, and their clocks checked against the Pi
- [ ] Run ID, date, lighting, car IDs, battery state and software version recorded
- [ ] Raw footage stored under the data policy (D8)

## D.4 Evaluation sign-off

- [ ] Success criteria and stop rule written down before the run
- [ ] Held-out runs kept apart from tuning runs
- [ ] Every failed run included in the report
- [ ] Seeds and data versions recorded

## D.5 Decision meeting (D1 to D8)

- [ ] Each decision has an owner and a date
- [ ] The choice and its reason are recorded in the project log
- [ ] Open items have a named person and a review date

# Appendix E: References

The sources below were checked during research, through search results or page excerpts. Vendor prices are listing prices seen at the time, and they will have changed. Where a source is a fan site or rests on a single snippet, the text says so. This version lists no references from memory.

## E.1 F1 timing and rules

- [1] The Field F1, How to Read the F1 Timing Screen and Timing Tower. https://www.thefieldf1.com/charts/how-to-read-timing-screen
- [2] FLOW RACERS, What Does Interval Mean In F1? The 150 to 200 m loop claim is on this page and is unverified. https://flowracers.com/blog/what-does-interval-mean-in-f1/
- [3] FLOW RACERS, How Are F1 Lap Times Measured? https://flowracers.com/blog/how-are-f1-lap-times-measured/
- [4] F1 Briefing, MGU-K Parameters: 2026 Rules. https://f1briefing.com/mgu-k-parameters-2026-rules/
- [5] TopGearbox, How Formula 1's 2026 Energy Rules Are Redefining Racecraft. https://topgearbox.com/cars/entertainment/2026-formula-1-energy-rules-racecraft/
- [6] The Independent, F1's new rules for 2026 explained, including overtake mode and active aero. https://www.independent.co.uk/f1/f1-2026-new-rules-overtake-mode-active-aero-drs-b2934133.html

## E.2 Game telemetry and data

- [7] EA, Data Output from F1 25 (v3 PDF). https://forums.ea.com/t5/s/tghpe58374/attachments/tghpe58374/f1-games-game-info-hub-en/61/4/Data%20Output%20from%20F1%2025%20v3.pdf
- [8] MacManley, f1-26-udp: F1 25 (2026 Season Pack) telemetry parser for ESP32 and ESP8266. https://github.com/MacManley/f1-26-udp
- [9] FastF1, Python package for F1 timing, results and telemetry (theOehrly, MIT licence). https://github.com/theOehrly/Fast-F1 and documentation at https://docs.fastf1.dev

## E.3 Energy optimisation

- [10] Limebeer, Perantoni and Rao (2014), Optimal control of Formula One car energy recovery systems. International Journal of Control 87(10), 2065 to 2080. https://doi.org/10.1080/00207179.2014.900705
- [11] Javed and Samuel (2022), Energy Optimal Control for Formula One Race Car. SAE Technical Paper 2022-01-1043. https://doi.org/10.4271/2022-01-1043

## E.4 Prior art: strategy simulators and timing systems

- [12] MDerazNasr, Race-Strategy-Simulator. https://github.com/MDerazNasr/Race-Strategy-Simulator
- [13] TUM FTM, race-simulation. https://github.com/TUMFTM/race-simulation
- [14] rembertdesigns, pit-stop-simulator. https://github.com/rembertdesigns/pit-stop-simulator
- [15] shaxammm, f1-race-strategy-simulator. https://github.com/shaxammm/f1-race-strategy-simulator
- [16] polyvision, EasyRaceLapTimer. https://github.com/polyvision/EasyRaceLapTimer
- [17] laptrack.app, free camera lap timer for RC cars. https://laptrack.app
- [18] kbennett2000, rc-lap-timer (MIT licence). https://github.com/kbennett2000/rc-lap-timer
- [19] LapTrap, Google Play listing. https://play.google.com/store/apps/details?id=com.laptrap&hl=en_US
- [20] Raspberry Pi Forums, club timing system discussion. https://forums.raspberrypi.com/viewtopic.php?t=198740
- [21] Silies and Braubach (2024), camera-only timing (Springer chapter). https://link.springer.com/chapter/10.1007/978-3-031-60023-4_20

## E.5 Vision and tracking

- [22] Comparative benchmarking of CPU, GPU and NPU architectures for real-time YOLOv8n inference on embedded edge platforms (IJERT, 2026). https://www.ijert.org/comparative-benchmarking-of-cpu-gpu-and-npu-architectures-for-real-time-yolov8n-inference-on-embedded-edge-platforms-ijertv15is030232
- [23] HOTA: a higher order metric for evaluating multi-object tracking, arXiv:2009.07736. https://arxiv.org/abs/2009.07736
- [24] ByteTrack, multi-object tracker (FoundationVision). https://github.com/FoundationVision/ByteTrack

## E.6 Hardware, platforms and compression

- [25] Raspberry Pi, Raspberry Pi AI HAT+ announcement. https://www.raspberrypi.com/news/raspberry-pi-ai-hat/
- [26] SparkFun, product listings for the AI HAT+ and the Jetson Orin Nano Super. https://www.sparkfun.com
- [27] PiShop, listings for the Global Shutter Camera and the AI HAT+. https://pishop.us
- [28] The Pi Hut, Global Shutter Camera listing. https://thepihut.com
- [29] Micro Center, Raspberry Pi 5 8 GB listing. https://www.microcenter.com
- [30] NVIDIA, Jetson Orin Nano Super (launch price and specification as recorded in research). https://www.nvidia.com
- [31] Espressif, ESP32 product information. https://www.espressif.com
- [32] Scientific Reports 15, 44299 (2025): quantisation as the most effective MCU compression method in its comparison. https://www.nature.com/articles/s41598-025-27818-9

## E.7 Racing platforms and teams

- [33] F1TENTH, 1/10-scale autonomous racing platform and community. https://f1tenth.org
- [34] Indro Robotics, F1TENTH platform costs. https://indrorobotics.ca
- [35] Formula Student Germany, Cairo University Racing Team (CURT) team page. https://www.formulastudent.de/teams/fsc/details/tid/479
- [36] Racecar Engineering, Cairo University Racing Team feature. https://www.racecar-engineering.com/cars/cairo/

## E.8 Reinforcement learning

- [37] Wurman et al. (2022), Outracing champion Gran Turismo drivers with deep reinforcement learning. Nature 602, 223 to 228. https://www.nature.com/articles/s41586-021-04357-7
