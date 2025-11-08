```mermaid
flowchart TB
  %% ============================================================
  %% DYNAMIC WIRELESS EV CHARGING → UTILITY CORRELATION WORKFLOW
  %% Phases 1–5 (detailed). Safe for Mermaid Live, draw.io, Notion.
  %% ============================================================

  %% ---------- PHASE 1 ----------
  subgraph P1[Phase 1: Single-EV Dynamic Charging Model (DWPT)]
    direction TB

    P1_PUR[Purpose:\nBuild a validated single-EV model to produce realistic P(t) and ΔSOC under dynamic wireless charging]

    P1_ACT1[Activities:\nDefine EV motion profile: v(t), a(t), lane alignment\nSet pad geometry and spacing]
    P1_ACT2[Activities:\nModel wireless link: M(x), k(x), η_link(x)\nTrapezoidal power pulse per pad]
    P1_ACT3[Activities:\nAdd power electronics: rectifier/DC-DC, I/V limits, η_ch(t)\nIntegrate battery model + BMS limits; compute SOC(t)]
    P1_ACT4[Activities:\nImplement interface timing: coil enable/disable per segment\nLog instantaneous P_out(t), I_batt(t), V_pack(t)]

    P1_IN[Inputs (Data Sources):\nVehicle: mass/weight, E_pack, V_pack range, I_max, thermal limits\nPads: coil size/turns, resonant f0, pad length/spacing, P_max\nMotion: speed traces (const/ramps/stop-go), alignment offsets, gap]

    P1_EQ[Core Models / Equations:\nx(t) = ∫v(t) dt → M(x), k(x) → P_link(t) ≈ f(M, I_tx)\nPulse shape: rise/flat/fall trapezoid limited by P_max, η_link(x), η_ch(t)\ndSOC/dt = (η_total · P_in(t)) / E_pack\nLimits: I_batt ≤ I_max, V_pack within bounds; thermal derate if needed]

    P1_SIM[Simulations:\nTime-domain per-pad (ms), multi-pad segment (s)\nScenarios: speed {30, 60, 90 km/h}; alignment {0, 10, 20 cm}; SOC {20→80%}]

    P1_OUT[Outputs (Artifacts):\nCSV: P(t) for single EV; ΔSOC per segment; Wh per pad; peak power; η_total\nPlots: power pulse(s), SOC trajectory, sensitivity curves\nReusable “unit load profile” for Phase 2]

    P1_VAL[Validation:\nPulse timing vs v(t) & pad length; energy sanity: Wh ≈ P_max × dwell × η_total\nLimits respected (I/V/thermal); realistic ΔSOC vs distance]

    P1_ASM[Assumptions & Risks:\nQuasi-static coupling; ignore harmonics/EMC; perfect coil timing\nRisks: misalignment variability dominates energy; high speed narrows pulses]

    P1_FLOW1 --> P1_PUR
    P1_PUR --> P1_ACT1 --> P1_ACT2 --> P1_ACT3 --> P1_ACT4
    P1_IN --> P1_EQ --> P1_SIM --> P1_OUT --> P1_VAL --> P1_ASM
    P1_ACT3 --> P1_EQ
  end

  %% ---------- PHASE 2 ----------
  subgraph P2[Phase 2: Multi-EV Load Generation & Pre-Utility KPIs]
    direction TB

    P2_PUR[Purpose:\nAggregate many single-EV profiles into P_sys(t); compute Peak, P95, P99, Energy]

    P2_ACT1[Activities:\nGenerate traffic: Poisson, rush-hour λ(t), platoons\nSpeed/SOC distributions; EV penetration %]
    P2_ACT2[Activities:\nMap EVs to pad timelines; superimpose P_i(t) with spacing/occupancy rules\nBuild 5-s and 15-min P_sys(t) series]
    P2_ACT3[Activities:\nCompute pre-utility KPIs; assemble representative days: baseline, P95-day, P99-day]

    P2_IN[Inputs (Data Sources):\nTraffic: λ, headway distributions, platoon size/spacing, EV%\nVehicle mix: mass, E_pack, SOC deficits, charge-capable share\nInfrastructure: road length, pad density/coverage, per-pad P_max, zone caps]

    P2_EQ[Core Models / Equations:\nP_sys(t) = Σ P_i(t − τ_i)\nPeak = max(P_sys)\nP95/P99 = quantiles{P_sys}\nEnergy = Σ P_sys · Δt / 3600\nDiversity/simultaneity from arrivals & occupancy]

    P2_SIM[Simulations:\nMonte Carlo day (24h) over traffic + vehicle mix\nScenario set: {low/med/high EV%} × {free-flow/rush/platoons} × {coverage levels}]

    P2_OUT[Outputs (Artifacts):\nTime-series P_sys(t) CSV @5-s/15-min; KPI tables (Peak, P95, P99, Energy)\nPlots: daily profile, load duration curve (LDC), histogram\nRepresentative days exported for Phase 3]

    P2_VAL[Validation:\nConservation: Σ vehicle energy gain ≈ delivered (± modeled losses)\nCapacity: per-pad/zone caps not exceeded; occupancy conflicts resolved\nPlausibility: rush > off-peak; platoons → sharper spikes]

    P2_ASM[Assumptions & Risks:\nArrivals ⟂ SOC; no weather/incidents; pad uptime 100%\nRisk: underestimating simultaneity → understated Peak/P99]

    P2_PUR --> P2_ACT1 --> P2_ACT2 --> P2_ACT3
    P2_IN --> P2_EQ --> P2_SIM --> P2_OUT --> P2_VAL --> P2_ASM
  end

  %% ---------- PHASE 3 ----------
  subgraph P3[Phase 3: Utility Grid Impact Assessment]
    direction TB

    P3_PUR[Purpose:\nEvaluate distribution-grid performance under P_sys(t)]

    P3_ACT1[Activities:\nMap P_sys(t) to feeder nodes (lumped or distributed)\nConfigure feeder: lines (R/X), regs, caps, transformer\nRun time-series power flow (baseline vs with EV load)]

    P3_IN[Inputs (Data Sources):\nRepresentative P_sys(t): baseline, P95-day, P99-day\nGrid data: line impedances/topology, transformer rating/impedance\nBase-load profiles, TOU tariffs]

    P3_EQ[Core Models / Equations:\nAC power flow (P, Q, V, θ)\nLosses = Σ I²R over lines\nVoltage limits: V_min ≤ V_bus ≤ V_max\nCost = Energy_kWh×price + Demand_kW×charge]

    P3_SIM[Simulations:\nTime-series load flow (e.g., OpenDSS/pandapower) @15-min\nCases: {weak/strong feeder} × {EV penetration levels} × {seasonal base load}]

    P3_OUT[Outputs (Artifacts):\nVoltage profiles (min/avg/max), line losses (%/kWh)\nTransformer loading (%) & thermal headroom\nPeak feeder demand & TOU cost impact\nPlots: time traces, feeder heatmaps]

    P3_VAL[Validation:\nPower balance: substation supply ≈ base + EV + losses\nVoltage compliance vs standard (e.g., ±5%)\nSanity: higher P_sys → higher losses/ΔV/peaks]

    P3_ASM[Assumptions & Risks:\nHarmonics/EMI ignored; perfect regulator response\nRisks: hidden bottlenecks; undervalued simultaneous pad activation]

    P3_PUR --> P3_ACT1
    P3_IN --> P3_EQ --> P3_SIM --> P3_OUT --> P3_VAL --> P3_ASM
  end

  %% ---------- PHASE 4 ----------
  subgraph P4[Phase 4: Risk & Sensitivity Analysis]
    direction TB

    P4_PUR[Purpose:\nQuantify uncertainty and tail risks for grid KPIs]

    P4_ACT1[Activities:\nMonte Carlo over traffic, EV mix, alignment, base load\nExtract distributions of Peak, ΔV_min, %Losses, Transformer%]
    P4_ACT2[Activities:\nCompute VaR (α) and CVaR (α); build risk heatmaps\nRank sensitivities (Sobol/Pearson/Spearman)]

    P4_IN[Inputs (Randomized):\nλ(t) & headway variability; platoon probability\nSpeed & SOC distributions; charging-capable share\nGrid: feeder strength, base-load scenarios, temperature]

    P4_EQ[Core Metrics:\nVaR_α = quantile_α(metric)\nCVaR_α = E[metric | metric ≥ VaR_α]]

    P4_SIM[Simulations:\nMany runs of (Phase 2 → Phase 3) with new seeds\nRecord KPI vectors per run; build PDFs/CDFs]

    P4_OUT[Outputs (Artifacts):\nVaR/CVaR tables for Peak, ΔV, Losses, Transformer%\nProbability of violations (e.g., V < 0.95 pu, Transformer > 100%)\nTornado/heatmap charts: key drivers]

    P4_VAL[Validation:\nConvergence of tail estimates (stable CVaR vs runs)\nBack-check: extreme runs align with P99/P95 representative days]

    P4_ASM[Assumptions & Risks:\nIndependence approximations; no correlated incidents\nRisk: long-tail from rare high simultaneity events]

    P4_PUR --> P4_ACT1 --> P4_ACT2
    P4_IN --> P4_EQ --> P4_SIM --> P4_OUT --> P4_VAL --> P4_ASM
  end

  %% ---------- PHASE 5 ----------
  subgraph P5[Phase 5: Mitigation — ESS & Smart Scheduling]
    direction TB

    P5_PUR[Purpose:\nReduce grid stress via storage and coordinated control]

    P5_ACT1[Activities:\nAdd ESS at feeder/site: size (MW/MWh), efficiency, dispatch rules\nSmart scheduling: cap P_sys, limit concurrent pads, priority by SOC\nRe-run worst/typical scenarios with control in loop]

    P5_IN[Inputs (Design Params):\nESS: power limit, energy capacity, SoC bounds, thresholds\nScheduling: max concurrent EVs/pads, power cap, fairness/priority\nOperational constraints: service level (energy-not-served limit)]

    P5_EQ[Core Models / Equations:\nNet load = P_sys − P_ESS_discharge + P_ESS_charge\nSoC_{t+1} = SoC_t ± (P_ESS · Δt · η)\nControl: if P_sys > cap ⇒ throttle/share; else ⇒ normal]

    P5_SIM[Simulations:\nTime-series power flow + ESS SoC + control logic\nScenario sweep: {ESS sizes} × {caps} × {priority rules}]

    P5_OUT[Outputs (Artifacts):\nReduced Peak, improved ΔV, lower %Losses, Transformer% < 100%\nRisk shrink: lower VaR/CVaR of Peak & violations\nService metrics: energy-not-served, charging delay distribution]

    P5_VAL[Validation:\nESS energy balance; no SoC cliff; cap adherence\nCompare pre/post mitigation KPIs & risk metrics]

    P5_ASM[Assumptions & Risks:\nIdeal control/communications reliability; ESS lifecycle not modeled\nRisks: prolonged peaks > ESS energy; unfair throttling]

    P5_END[Conclusion / Recommendations:\nOptimal ESS size & control cap for target reliability\nPolicy notes: feeder upgrades vs storage vs scheduling trade-offs]

    P5_PUR --> P5_ACT1
    P5_IN --> P5_EQ --> P5_SIM --> P5_OUT --> P5_VAL --> P5_ASM --> P5_END
  end

  %% ---------- PHASE LINKS ----------
  P1_OUT --> P2_PUR
  P2_OUT --> P3_PUR
  P3_OUT --> P4_PUR
  P4_OUT --> P5_PUR
