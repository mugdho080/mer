```mermaid
flowchart TB
  %% ============================================
  %% DYNAMIC WIRELESS EV CHARGING → UTILITY FLOW
  %% Phases 1–5 (GitHub-friendly Mermaid)
  %% ============================================

  %% ---------- PHASE 1 ----------
  subgraph P1[Phase 1: Single-EV Dynamic Charging Model (DWPT)]
    P1_PUR[Purpose:<br/>Build a validated single-EV model to produce realistic P(t) and dSOC under dynamic wireless charging]

    P1_ACT1[Activities:<br/>Define EV motion profile: v(t), a(t), lane alignment<br/>Set pad geometry and spacing]
    P1_ACT2[Activities:<br/>Model wireless link: M(x), k(x), eta_link(x)<br/>Trapezoidal power pulse per pad]
    P1_ACT3[Activities:<br/>Add power electronics: rectifier/DC-DC, I/V limits, eta_ch(t)<br/>Integrate battery model + BMS limits; compute SOC(t)]
    P1_ACT4[Activities:<br/>Implement interface timing: coil enable/disable per segment<br/>Log instantaneous P_out(t), I_batt(t), V_pack(t)]

    P1_IN[Inputs (Data Sources):<br/>Vehicle: mass/weight, E_pack, V_pack range, I_max, thermal limits<br/>Pads: coil size/turns, resonant f0, pad length/spacing, P_max<br/>Motion: speed traces (const/ramps/stop-go), alignment offsets, gap]

    P1_EQ[Core Models / Equations:<br/>x(t) = integral v(t) dt -> M(x), k(x) -> P_link(t) approx f(M, I_tx)<br/>Pulse shape: rise/flat/fall trapezoid limited by P_max, eta_link(x), eta_ch(t)<br/>dSOC/dt = (eta_total * P_in(t)) / E_pack<br/>Limits: I_batt <= I_max, V_pack within bounds; thermal derate if needed]

    P1_SIM[Simulations:<br/>Time-domain per-pad (ms), multi-pad segment (s)<br/>Scenarios: speed {30, 60, 90 km/h}; alignment {0, 10, 20 cm}; SOC {20->80%}]

    P1_OUT[Outputs (Artifacts):<br/>CSV: P(t) for single EV; dSOC per segment; Wh per pad; peak power; eta_total<br/>Plots: power pulse(s), SOC trajectory, sensitivity curves<br/>Reusable "unit load profile" for Phase 2]

    P1_VAL[Validation:<br/>Pulse timing vs v(t) & pad length; energy sanity: Wh approx P_max * dwell * eta_total<br/>Limits respected (I/V/thermal); realistic dSOC vs distance]

    P1_ASM[Assumptions & Risks:<br/>Quasi-static coupling; ignore harmonics/EMC; perfect coil timing<br/>Risks: misalignment variability dominates energy; high speed narrows pulses]

    P1_PUR --> P1_ACT1 --> P1_ACT2 --> P1_ACT3 --> P1_ACT4
    P1_IN --> P1_EQ --> P1_SIM --> P1_OUT --> P1_VAL --> P1_ASM
    P1_ACT3 --> P1_EQ
  end

  %% ---------- PHASE 2 ----------
  subgraph P2[Phase 2: Multi-EV Load Generation & Pre-Utility KPIs]
    P2_PUR[Purpose:<br/>Aggregate many single-EV profiles into P_sys(t); compute Peak, P95, P99, Energy]

    P2_ACT1[Activities:<br/>Generate traffic: Poisson, rush-hour lambda(t), platoons<br/>Speed/SOC distributions; EV penetration %]
    P2_ACT2[Activities:<br/>Map EVs to pad timelines; superimpose P_i(t) with spacing/occupancy rules<br/>Build 5-s and 15-min P_sys(t) series]
    P2_ACT3[Activities:<br/>Compute pre-utility KPIs; assemble representative days: baseline, P95-day, P99-day]

    P2_IN[Inputs (Data Sources):<br/>Traffic: lambda, headway distributions, platoon size/spacing, EV%<br/>Vehicle mix: mass, E_pack, SOC deficits, charge-capable share<br/>Infrastructure: road length, pad density/coverage, per-pad P_max, zone caps]

    P2_EQ[Core Models / Equations:<br/>P_sys(t) = sum P_i(t - tau_i)<br/>Peak = max(P_sys)<br/>P95/P99 = quantiles{P_sys}<br/>Energy = sum P_sys * dt / 3600<br/>Diversity/simultaneity from arrivals & occupancy]

    P2_SIM[Simulations:<br/>Monte Carlo day (24h) over traffic + vehicle mix<br/>Scenario set: {low/med/high EV%} x {free-flow/rush/platoons} x {coverage levels}]

    P2_OUT[Outputs (Artifacts):<br/>Time-series P_sys(t) CSV @5-s/15-min; KPI tables (Peak, P95, P99, Energy)<br/>Plots: daily profile, load duration curve (LDC), histogram<br/>Representative days exported for Phase 3]

    P2_VAL[Validation:<br/>Conservation: sum vehicle energy gain approx delivered (plus/minus modeled losses)<br/>Capacity: per-pad/zone caps not exceeded; occupancy conflicts resolved<br/>Plausibility: rush > off-peak; platoons -> sharper spikes]

    P2_ASM[Assumptions & Risks:<br/>Arrivals independent of SOC; no weather/incidents; pad uptime 100%<br/>Risk: underestimating simultaneity -> understated Peak/P99]

    P2_PUR --> P2_ACT1 --> P2_ACT2 --> P2_ACT3
    P2_IN --> P2_EQ --> P2_SIM --> P2_OUT --> P2_VAL --> P2_ASM
  end

  %% ---------- PHASE 3 ----------
  subgraph P3[Phase 3: Utility Grid Impact Assessment]
    P3_PUR[Purpose:<br/>Evaluate distribution-grid performance under P_sys(t)]

    P3_ACT1[Activities:<br/>Map P_sys(t) to feeder nodes (lumped or distributed)<br/>Configure feeder: lines (R/X), regs, caps, transformer<br/>Run time-series power flow (baseline vs with EV load)]

    P3_IN[Inputs (Data Sources):<br/>Representative P_sys(t): baseline, P95-day, P99-day<br/>Grid data: line impedances/topology, transformer rating/impedance<br/>Base-load profiles, TOU tariffs]

    P3_EQ[Core Models / Equations:<br/>AC power flow (P, Q, V, theta)<br/>Losses = sum I^2 R over lines<br/>Voltage limits: V_min <= V_bus <= V_max<br/>Cost = Energy_kWh * price + Demand_kW * charge]

    P3_SIM[Simulations:<br/>Time-series load flow (e.g., OpenDSS/pandapower) @15-min<br/>Cases: {weak/strong feeder} x {EV penetration levels} x {seasonal base load}]

    P3_OUT[Outputs (Artifacts):<br/>Voltage profiles (min/avg/max), line losses (%/kWh)<br/>Transformer loading (%) and thermal headroom<br/>Peak feeder demand and TOU cost impact<br/>Plots: time traces, feeder heatmaps]

    P3_VAL[Validation:<br/>Power balance: substation supply approx base + EV + losses<br/>Voltage compliance vs standard (e.g., +/-5%)<br/>Sanity: higher P_sys -> higher losses/DeltaV/peaks]

    P3_ASM[Assumptions & Risks:<br/>Harmonics/EMI ignored; perfect regulator response<br/>Risks: hidden bottlenecks; undervalued simultaneous pad activation]

    P3_PUR --> P3_ACT1
    P3_IN --> P3_EQ --> P3_SIM --> P3_OUT --> P3_VAL --> P3_ASM
  end

  %% ---------- PHASE 4 ----------
  subgraph P4[Phase 4: Risk & Sensitivity Analysis]
    P4_PUR[Purpose:<br/>Quantify uncertainty and tail risks for grid KPIs]

    P4_ACT1[Activities:<br/>Monte Carlo over traffic, EV mix, alignment, base load<br/>Extract distributions of Peak, V_min, %Losses, Transformer%]
    P4_ACT2[Activities:<br/>Compute VaR(alpha) and CVaR(alpha); build risk heatmaps<br/>Rank sensitivities (Sobol/Pearson/Spearman)]

    P4_IN[Inputs (Randomized):<br/>lambda(t) and headway variability; platoon probability<br/>Speed and SOC distributions; charging-capable share<br/>Grid: feeder strength, base-load scenarios, temperature]

    P4_EQ[Core Metrics:<br/>VaR_alpha = quantile_alpha(metric)<br/>CVaR_alpha = E[metric | metric >= VaR_alpha]]

    P4_SIM[Simulations:<br/>Many runs of (Phase 2 -> Phase 3) with new seeds<br/>Record KPI vectors per run; build PDFs/CDFs]

    P4_OUT[Outputs (Artifacts):<br/>VaR/CVaR tables for Peak, V_min, Losses, Transformer%<br/>Probability of violations (e.g., V < 0.95 pu, Transformer > 100%)<br/>Tornado/heatmap charts: key drivers]

    P4_VAL[Validation:<br/>Convergence of tail estimates (stable CVaR vs runs)<br/>Back-check: extreme runs align with P99/P95 representative days]

    P4_ASM[Assumptions & Risks:<br/>Independence approximations; no correlated incidents<br/>Risk: long-tail from rare high simultaneity events]

    P4_PUR --> P4_ACT1 --> P4_ACT2
    P4_IN --> P4_EQ --> P4_SIM --> P4_OUT --> P4_VAL --> P4_ASM
  end

  %% ---------- PHASE 5 ----------
  subgraph P5[Phase 5: Mitigation — ESS & Smart Scheduling]
    P5_PUR[Purpose:<br/>Reduce grid stress via storage and coordinated control]

    P5_ACT1[Activities:<br/>Add ESS at feeder/site: size (MW/MWh), efficiency, dispatch rules<br/>Smart scheduling: cap P_sys, limit concurrent pads, priority by SOC<br/>Re-run worst/typical scenarios with control in loop]

    P5_IN[Inputs (Design Params):<br/>ESS: power limit, energy capacity, SoC bounds, thresholds<br/>Scheduling: max concurrent EVs/pads, power cap, fairness/priority<br/>Operational constraints: service level (energy-not-served limit)]

    P5_EQ[Core Models / Equations:<br/>Net load = P_sys - P_ESS_discharge + P_ESS_charge<br/>SoC(t+1) = SoC(t) +/- (P_ESS * dt * eta)<br/>Control: if P_sys > cap then throttle/share else normal]

    P5_SIM[Simulations:<br/>Time-series power flow + ESS SoC + control logic<br/>Scenario sweep: {ESS sizes} x {caps} x {priority rules}]

    P5_OUT[Outputs (Artifacts):<br/>Reduced Peak, improved V profile, lower %Losses, Transformer% < 100%<br/>Risk shrink: lower VaR/CVaR of Peak and violations<br/>Service metrics: energy-not-served, charging delay distribution]

    P5_VAL[Validation:<br/>ESS energy balance; no SoC cliff; cap adherence<br/>Compare pre/post mitigation KPIs and risk metrics]

    P5_ASM[Assumptions & Risks:<br/>Ideal control/communications reliability; ESS lifecycle not modeled<br/>Risks: prolonged peaks > ESS energy; unfair throttling]

    P5_END[Conclusion / Recommendations:<br/>Optimal ESS size and control cap for target reliability<br/>Policy notes: feeder upgrades vs storage vs scheduling trade-offs]

    P5_PUR --> P5_ACT1
    P5_IN --> P5_EQ --> P5_SIM --> P5_OUT --> P5_VAL --> P5_ASM --> P5_END
  end

  %% ---------- PHASE LINKS ----------
  P1_OUT --> P2_PUR
  P2_OUT --> P3_PUR
  P3_OUT --> P4_PUR
  P4_OUT --> P5_PUR

   
