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

    P1_SIM[Simulations:<br/>Time-domain]()_
