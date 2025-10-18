# 🌟 RISC-V SoC Tapeout – Week-3: Post-Synthesis Gate-Level Simulation (GLS) and STA Fundamentals

> **Purpose:** Verify BabySoC behavior after synthesis (gate-level netlist) and generate post-synthesis waveforms & logs for submission.

---

## 🚀 Quick summary

🔍 Goal: Validate functionality after synthesis and guarantee timing closure for the VSDBabySoC using GLS and OpenSTA.

⚡ Outcome: Gate‑level netlist, GLS waveform comparison, per‑corner STA reports (WNS, TNS), and suggested fixes for violations.

---

## 📂 Folder Structure

```
VSDBabySoC/
├─ src/
│  ├─ module/
│  │  ├─ vsdbabysoc.v
│  │  ├─ rvmyth.v
│  │  ├─ clk_gate.v
│  │  └─ testbench.v
│  └─ include/
├─ src/lib/
│  ├─ sky130_fd_sc_hd__tt_025C_1v80.lib
│  ├─ avsdpll.lib
│  └─ avsddac.lib
└─ output/
   └─ post_synth_sim/
      ├─ vsdbabysoc.synth.v
      ├─ post_synth_sim.out
      └─ post_synth_sim.vcd
```

---

# 🧭 Synthesis (Yosys) — Step-by-Step

1. 🔁 Start Yosys:

```bash
yosys
```
<div align="center">

<img width="1024" height="1024" alt="yosys" src="https://github.com/user-attachments/assets/5985d5d4-6747-4e9f-b0b1-7155fd36c36c" />

</div>

2. 📥 Read top-level and modules (use `-I` to include headers):

```yosys
read_verilog /home/chittesh/VLSI/VSDBabySoC/src/module/vsdbabysoc.v
read_verilog -I /home/chittesh/VLSI/VSDBabySoC/src/include /home/chittesh/VLSI/VSDBabySoC/src/module/rvmyth.v
read_verilog -I /home/chittesh/VLSI/VSDBabySoC/src/include /home/chittesh/VLSI/VSDBabySoC/src/module/clk_gate.v
```
<div align="center">

<img width="1024" height="1024" alt="read_top_level" src="https://github.com/user-attachments/assets/89dc2a79-4ae9-4da3-8d67-804c31875acf" />

</div>

3. 📚 Load liberty files:

```yosys
read_liberty -lib /home/chittesh/VLSI/VSDBabySoC/src/lib/avsdpll.lib
read_liberty -lib /home/chittesh/VLSI/VSDBabySoC/src/lib/avsddac.lib
read_liberty -lib /home/chittesh/VLSI/VSDBabySoC/src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```
<div align="center">

<img width="1024" height="1024" alt="load_liberity_files" src="https://github.com/user-attachments/assets/89daaa98-015e-4941-bbf0-026e87ec84dd" />

</div>

4. 🧩 Synthesize the top module:

```yosys
synth -top vsdbabysoc
```
<div align="center">

<img width="1024" height="1024" alt="synth_top1" src="https://github.com/user-attachments/assets/f96e693c-5ca2-479b-bb10-da7362dd02ad" />

</div>

<div align="center">

<img width="1024" height="1024" alt="synth_top2" src="https://github.com/user-attachments/assets/f5181295-e9aa-4ed0-98e5-bd0389026d04" />

</div>

<div align="center">

<img width="1024" height="1024" alt="synth_top3" src="https://github.com/user-attachments/assets/d8731fd6-b4d2-44bc-9d55-390411dc9d51" />

</div>

<div align="center">

<img width="1024" height="1024" alt="synth_top4" src="https://github.com/user-attachments/assets/723ca445-4f19-4739-8374-43cf420abb15" />

</div>

5. 🔁 Map dff/lib primitives:

```yosys
dfflibmap -liberty /home/chittesh/VLSI/VSDBabySoC/src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```
<div align="center">

<img width="1024" height="1024" alt="dfflibmap" src="https://github.com/user-attachments/assets/758d74a3-06c3-4dcf-a7cf-4605935ff7d4" />

</div>

6. ⚙️ Optimize & technology map:

```yosys
opt
abc -liberty /home/chittesh/VLSI/VSDBabySoC/src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib -script +strash;scorr;ifraig;retime;{D};strash;dch,-f;map,-M,1,{D}
```
<div align="center">

<img width="1024" height="1024" alt="abc1" src="https://github.com/user-attachments/assets/9d8d62aa-d01d-42ea-9990-420ca554fce2" />

</div>

<div align="center">

<img width="1024" height="1024" alt="abc2" src="https://github.com/user-attachments/assets/7e601a06-4f49-406e-95ae-d75db3f89137" />

</div>

7. 🧼 Final clean-up & rename:

```yosys
flatten
setundef -zero
clean -purge
rename -enumerate
```
<div align="center">

<img width="1024" height="1024" alt="clean_up_rename" src="https://github.com/user-attachments/assets/69f3a301-3f69-4a83-8ff3-5c56ab257de6" />

</div>

8. 📊 Check stats and write netlist:

```yosys
stat
write_verilog -noattr /home/chittesh/VLSI/VSDBabySoC/output/post_synth_sim/vsdbabysoc.synth.v
```
<div align="center">

<img width="1024" height="1024" alt="stat_netlist" src="https://github.com/user-attachments/assets/8405bdf4-2d4f-4e64-8944-a6e3af6d94e3" />

</div>

> 📌 Tip: Save the full Yosys transcript to `synthesis_logs.txt` for submission.

---

# 🧪 Part - 1: Post-Synthesis Simulation (GLS)

1. 🧾 Compile the testbench with the synthesized netlist (example using Icarus):

```bash
iverilog -o /home/chittesh/VLSI/VSDBabySoC/output/post_synth_sim/post_synth_sim.out \
  -DPOST_SYNTH_SIM -DFUNCTIONAL -DUNIT_DELAY=#1 \
  -I /home/chittesh/VLSI/VSDBabySoC/src/include \
  -I /home/chittesh/VLSI/VSDBabySoC/src/module \
  /home/chittesh/VLSI/VSDBabySoC/src/module/testbench.v \
  /home/chittesh/VLSI/VSDBabySoC/output/post_synth_sim/vsdbabysoc.synth.v
```

2. 📁 Move to output dir:

```bash
cd /home/chittesh/VLSI/VSDBabySoC/output/post_synth_sim/
```

3. ▶️ Run the simulation:

```bash
./post_synth_sim.out
```

4. 🔍 View the waveform in GTKWave:

```bash
gtkwave post_synth_sim.vcd
```
<div align="center">

<img width="1024" height="1024" alt="post_synth_sim_out" src="https://github.com/user-attachments/assets/ce24435c-bf0f-4826-b7b1-c9fd5c5fdac6" />

</div>

> ⚠️ If you want timing-annotated simulation, back-annotate with an SDF file (generated by your place-and-route tool). SDF may reveal real timing failures — run it only if SDF is available.

---

## 🧾 How to compare GLS vs RTL outputs

- 📝 Save both simulation outputs as logs: `functional.out` (RTL) and `gls.out` (GLS).
- 🔁 Use `diff` for textual comparison:

```bash
diff -u functional.out gls.out > functional_vs_gls.diff
```

- 🖼️ For waveforms: open both `.vcd` files in GTKWave and overlay or visually compare signals (top-level outputs, clocks, resets, and key internal signals).

**Pre-Synthesis Simulation**

<div align="center">

<img width="1024" height="1024" alt="pre_synth_sim_out" src="https://github.com/user-attachments/assets/78520cf7-e8ea-4606-8fb1-c1cc42245c9e" />

</div>

**Post-Synthesis Simulation**

<div align="center">

<img width="1024" height="1024" alt="post_synth_sim_out" src="https://github.com/user-attachments/assets/d8bc0fad-069f-4f49-ba87-efb8ce2a149b" />

</div>

- ✅ Attach examples of matching vectors and at least one GTKWave screenshot showing identical outputs for your deliverable.

---

## 📦 Deliverables (Part-1)

- 📄 `synth.log` — full Yosys transcript
- 🧾 `pre_synth_sim.out` — simulation output logs
- 🧪 `pre_synth_sim.vcd` (or `.wlf`) — GLS waveform dump
- 🧾 `post_synth_sim.out` — simulation output logs
- 🧪 `post_synth_sim.vcd` (or `.wlf`) — GLS waveform dump
- 🧾 `vsdbabysoc.synth.v` — synthesized netlist
- 🖼️ GTKWave screenshots showing comparison

> It's all given in the **outputs** folder

---

## 📝 Short note — GLS vs Functional simulation

I performed Gate-Level Simulation (GLS) using the synthesized netlist `vsdbabysoc.synth.v` and the same testbench/stimulus used for the Week-2 functional simulation. The GLS waveform (`post_synth_sim.vcd`) was compared against the functional waveform both visually (GTKWave screenshots).

Result: The outputs matched for all applied test vectors. No functional mismatches were observed.

Attached: synth.log, pre_synth_sim.out, pre_synth_sim.vcd, post_synth_sim.out, post_synth_sim.vcd, vsdbabysoc.synth.v, waveform screenshots.

---

# 🧭 Part - 2: Fundamentals of STA (Static Timing Analysis)

> Source note: core teaching reference is the VLSI Academy STA course (Udemy). Supporting technical definitions and commands referenced from OpenSTA, Synopsys, and SDC docs. 

---

## 🔖 Quick navigation
🧩 Setup & Hold checks
  
🧮 Slack (calculation & meaning)  

⏰ Clock definitions (create_clock / generated clocks / uncertainty)  

🔗 Path-based analysis (types, reporting, how to interpret)  

🧰 Code snippets (SDC + OpenSTA)
  
✅ SoC checklist & deliverables

---

## 🧩 Setup & Hold checks

⚠️ **What they check:**  
  - Setup: data must arrive and be stable *before* the capture clock edge (so the downstream flop samples valid data).  
  - Hold: data must remain stable *after* the capture edge for the hold interval (to avoid sampling a changed value).

🧾 **Setup timing equation (single-register path):**  
  - `T_clock` (period) ≥ `T_comb` + `T_setup` + `Clock_uncertainty` + `Clock_latency_src - Clock_latency_dst`  
  - Where `T_comb` = combinational propagation from launch flop Q to capture flop D. (This is the notion STA checks when computing setup slack.)

🧾 **Hold timing equation (single-register path):**  
  - `T_comb_min` ≥ `T_hold` − (`Clock_latency_src - Clock_latency_dst`) − `uncertainty`  
  - If `T_comb_min` (earliest data arrival) is smaller than required hold window, hold violation occurs.

🛠 **Common causes of failures:**  
  - Setup fails: long combinational logic, slow cells, long nets (placement/routing).  
  - Hold fails: very short nets or very fast cells (data arrives too quickly), or incorrect clock insertion delay modeling.  

✅ **Practical tip:** Run *functional GLS without SDF* first to confirm logical parity; then run STA and, if needed, SDF back-annotated sims to observe real-race behavior. 

---

## 🧮 Slack — meaning, how to compute, and why it matters

🔍 **Definition (plain):**  
  - **Slack = Required Time − Arrival Time**. Positive slack → timing met; negative slack → timing violation. This is the single most used metric to judge timing health. 

🔢 **Step-by-step slack calculation (setup):**  
  1. compute arrival time (A) at capture pin = launch_edge + clock_to_Q + path_cell/net delays.  
  2. compute required time (R) = capture_edge_time − setup_time − clock_uncertainty − clock_latency_offset.  
  3. slack = R − A. If slack < 0, the path fails. 

📊 **Key slack metrics reported by STA tools:**  
  - **WNS (Worst Negative Slack)** — most negative single-path slack.  
  - **TNS (Total Negative Slack)** — sum of negative slacks across failing paths.  
  - **WHS (Worst Hold Slack)** — worst (most negative) hold slack. 

🧩 **Interpretation rules:**  
  - WNS ≈ primary indicator for setup closure.  
  - TNS shows cumulative severity — a large TNS is harder to fix.  
  - WHS guides small local fixes (buffers, delay insertion) for hold issues. 

---

## ⏰ Clock definitions — how to declare and what’s important

📝 **create_clock (SDC)** — the fundamental command declaring a primary clock:  
  ```sdc
  create_clock -name clk_core -period 10.0 [get_ports clk_core]
  ```  
  Use this to anchor all timing calculations. Tools expect accurate clock period and source.

🔁 **Generated clocks (from PLL/clock generator):**  
  - Use `create_generated_clock` to declare derived clocks (e.g., PLL ×2, ÷2), and specify the source pin so STA can compute launch/capture relations correctly. Example:  
  ```sdc
  create_generated_clock -name clk_core -source [get_ports clk_xtal] -multiply_by 2 -divide_by 1 [get_pins pll_inst/clk_core]
  ```  
  Generated clocks must include source latency to model the PLL insertion delay.

⚖️ **Clock uncertainty & skew:**  
  - `set_clock_uncertainty` (or `set_clock_jitter`) reduces available time (required) to account for jitter, clock tree imbalance, and clock insertion variability. Always include realistic uncertainty values. 

🔍 **Clock latency & source/destination offsets:**  
  - STA considers clock source latency differences between launch and capture flop (these modify required/arrival calculations). If you want to ignore latency for a path, many tools have flags, but be careful — ignoring can hide real problems. 

🧾 **Practical SoC note:** For BabySoC, explicitly declare clocks for PLL outputs (core clock, DAC clock). Use generated clocks and include `set_clock_uncertainty` with conservative values until CTS/clock-tree is known. 

---

## 🔗 Path-based analysis — types of paths & how STA reports them

🧭 **Path types (quick reference):**  
  - `Input → Register` (external data captured by a flop)  
  - `Register → Register` (typical sequential path)  
  - `Register → Output` (registered outputs driving pins)  
  - `Input → Output` (combinational path) 

🧾 **False paths & multi-cycle paths:**  
  - `set_false_path` — marks paths that are functionally unreachable (prevents wasting effort on impossible timing failures).  
  - `set_multicycle_path` — tells STA a path can use N clock cycles, relaxing the required time accordingly (useful for slow debug buses or intentional multi-cycle transfers). 

🛠 **How STA enumerates paths:**  
  - Tools compute arrival/required at all endpoints and then traverse topology to find worst-case (max/min) paths. Reporting lists the sequence of cells/nets, per-stage delays, and slack per path. Use `report_timing -max_paths N` to list the worst N paths. 

🔍 **What to inspect in a reported path:**  
  - Launch reg, capture reg, path delay breakdown (cell vs net), total slack, and any applied exceptions (false/mc-paths). If net delay dominates → physical fixes; if cell delay dominates → synthesis or cell selection fixes. 

📌 **Example OpenSTA command to inspect specific path segment:**  
  ```tcl
  report_checks -from [get_pins U1/Q] -to [get_pins U2/D]
  report_timing -from [get_registers regA] -to [get_registers regB] -max_paths 5
  ```  
  Use targeted `report_checks` / `report_timing` to reduce noise and focus on meaningful SoC paths (e.g., CPU → peripheral bus → DAC). 

---

## 🧰 Code snippets — SDC + OpenSTA focused on SoC cases

### SDC (SoC / PLL example)
```sdc
# Primary oscillator
create_clock -name clk_xtal -period 20.0 [get_ports clk_xtal]

# Generated clocks (PLL)
create_generated_clock -name clk_core -source [get_ports clk_xtal] -multiply_by 2 -divide_by 1 [get_pins pll_inst/clk_core]
create_generated_clock -name clk_dac  -source [get_ports clk_xtal] -multiply_by 1 -divide_by 2 [get_pins pll_inst/clk_out_dac]

# Clock uncertainty (jitter + skew margin)
set_clock_uncertainty 0.25

# IO delays
set_input_delay -clock clk_core 2.5 [get_ports uart_rx]
set_output_delay -clock clk_core 2.5 [get_ports dac_dout]

# Exceptions
set_false_path -from [get_pins */sync_*] -to [get_pins */sync_*]
set_multicycle_path 2 -from [get_registers dbg_src] -to [get_registers dbg_dst]
```

### OpenSTA (batch example)
```tcl
read_liberty -lib src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog output/post_synth_sim/vsdbabysoc.synth.v
link_design vsdbabysoc
# source constraints.sdc if OpenSTA variant supports it
create_clock -name clk_core -period 10 [get_ports clk_core]
report_timing -max_paths 10 > output/post_synth_sim/opensta_timing_report.txt
report_checks -path full > output/post_synth_sim/timing_checks.txt
```

---

## ✅ SoC-focused checklist (what to run & produce)

- 🧾 Generate and save `synthesis_logs.txt` (Yosys or commercial synth).  
- 🧪 Run GLS to produce `post_synth_sim.vcd` and capture waveform screenshots showing clocks, PC, bus transfers and DAC handshake.  
- 🧰 Prepare `constraints.sdc` with generated clocks and uncertainties; run OpenSTA to get `opensta_timing_report.txt` and `timing_checks.txt`. 

---

# 🧩 Part - 3: Installation of STA (Static Timing Analysis) and Generating Timing Graphs with OpenSTA

## 🧾 Executive summary

- 🎯 Goal: Run OpenSTA on the synthesized vsdbabysoc.synth.v, generate per-corner timing reports (min/max), extract WNS / TNS / WNS per corner, and produce timing graphs.

- ✅ Outcome: one-shot interactive flow + non-interactive TCL scripts for single-corner and multi-corner (PVT) runs, plus reporting / troubleshooting guidance.

---

## 📦 Quick checklist (what you must have)
- 📥 `vsdbabysoc.synth.v` — Yosys gate-level netlist.  
- 📚 `*.lib` — Liberty timing libraries for each PVT corner (e.g. `sky130_*`).  
- 🗂 `vsdbabysoc_synthesis.sdc` — SDC constraints (clocks, delays, exceptions).  
- 🧩 `sky130_fd_sc_hd.v` & `primitives.v` — cell Verilog models (for GLS if needed).  
- 🐳 Docker + local workspace to mount into the OpenSTA container.

---

## 🐳 Install & launch OpenSTA (Docker)
1. 🔁 `git clone` OpenSTA:
   ```bash
   git clone https://github.com/parallaxsw/OpenSTA.git
   cd OpenSTA
   ```
2. 🏗️ Build Docker image:
   ```bash
   docker build --file Dockerfile.ubuntu22.04 --tag opensta .
   ```
3. 🚀 Run interactive container (mount host `$HOME` to `/data`):
   ```bash
   docker run -it -v $HOME:/data opensta
   ```
4. 🔎 Inside container you’ll see OpenSTA prompt `%` — ready for commands.

---

## 🧩 Example used:



## ▶ Interactive STA flow
1. 📖 Load standard cells (typical/tt corner):
   ```tcl
   read_liberty /data/OpenSTA/examples/timing_libs/sky130_fd_sc_hd__tt_025C_1v80.lib
   ```
2. 🔌 Read gate netlist:
   ```tcl
   read_verilog /data/OpenSTA/examples/BabySoC/vsdbabysoc.synth.v
   ```
3. 🔗 Link design (creates the timing graph):
   ```tcl
   link_design vsdbabysoc
   ```
4. 🧾 Load SDC (clocks, constraints, exceptions):
   ```tcl
   read_sdc /data/OpenSTA/examples/BabySoC/vsdbabysoc_synthesis.sdc
   ```
5. ⏰ Define or verify clocks:
   ```tcl
   create_clock -name clk -period 10 [get_ports clk]
   set_input_delay -clock clk 0 {in1 in2}
   report_checks
   ```
   
> * By default, `report_checks` performs **setup (max delay)** checks.

6. 📊 Run setup & hold checks:
   ```tcl
   report_checks -path_delay min_max 
   ```
   
7. 📊 View only **hold (min delay)** paths:

  ```tcl
  report_checks -path_delay min
  ```

. 📉 Pull summary stats:
   ```tcl
   report_worst_slack -max     # worst setup (WNS)
   report_worst_slack -min     # worst hold
   report_tns                  # total negative slack
   report_wns                  # worst negative slack
   ```

---

## ⚡ SPEF-Based Parasitic Timing Analysis

- 📌 **Why:** SPEF/DSPEF adds post-layout RC; gives realistic net delays and better sign-off accuracy.  
- 🔧 **How:** `read_spef <path>` before `report_checks`. Ensure net naming in SPEF matches your netlist (PAR can rename nets).

To perform more realistic STA with parasitic RC information:

```tcl
read_liberty /OpenSTA/examples/nangate45_slow.lib.gz
read_verilog /OpenSTA/examples/example1.v
link_design top
read_spef /OpenSTA/examples/example1.dspef
create_clock -name clk -period 10 {clk1 clk2 clk3}
set_input_delay -clock clk 0 {in1 in2}
report_checks
```

---

## 🧾 Useful per-path fields:
- 🧭 `capacitance` — shows node cap; high → candidate for buffering.  
- ↗️ `slew` — reveals driver slew that increases delay.  
- 🧷 `input_pins` — endpoint pin info for traceability.  
- ⚖️ `fanout` — high fanout suggests buffering or sizing.  
- 🔢 `-digits 4` — readable precision.

Examples:
```tcl
report_checks -digits 4 -fields capacitance
```

```tcl
report_checks -digits 4 -fields {capacitance slew input_pins fanout}
```

```tcl
report_power
report_pulse_width_checks
report_units
```

---

## 🧰 Automate: single-corner script (`min_max_delays.tcl`)
```tcl
# min_max_delays.tcl
read_liberty -max /data/.../nangate45_slow.lib
read_liberty -min /data/.../nangate45_fast.lib

read_verilog /data/.../example1.v
link_design top
create_clock -name clk -period 10 {clk1 clk2 clk3}
set_input_delay -clock clk 0 {in1 in2}

report_checks -path_delay min_max -fields {capacitance slew fanout} -digits 4 > /data/.../min_max_report.txt
report_worst_slack -max -digits 4 >> /data/.../sta_worst_max_slack.txt
report_worst_slack -min -digits 4 >> /data/.../sta_worst_min_slack.txt
report_tns -digits 4 >> /data/.../sta_tns.txt
report_wns -digits 4 >> /data/.../sta_wns.txt
```
- ▶ Run non-interactive:
  ```bash
  docker run -it -v $HOME:/data opensta /data/path/to/min_max_delays.tcl
  ```

---

## 🔁 Automate: multi-corner PVT script (`sta_across_pvt.tcl`)
```tcl
# define corners
set list_of_lib_files {
 "sky130_fd_sc_hd__tt_025C_1v80.lib"
 "sky130_fd_sc_hd__ff_100C_1v95.lib"
 "sky130_fd_sc_hd__ss_n40C_1v28.lib"
}

# read constant IP libs
read_liberty /data/.../timing_libs/avsdpll.lib
read_liberty /data/.../timing_libs/avsddac.lib

# loop
foreach lib $list_of_lib_files {
  read_liberty /data/.../timing_libs/$lib
  read_verilog /data/.../BabySoC/vsdbabysoc.synth.v
  link_design vsdbabysoc
  read_sdc /data/.../BabySoC/vsdbabysoc_synthesis.sdc

  report_checks -path_delay min_max -fields {nets cap slew input_pins fanout} -digits 4 > /data/.../STA_OUTPUT/min_max_$lib.txt

  exec echo "$lib" >> /data/.../STA_OUTPUT/sta_worst_max_slack.txt
  report_worst_slack -max -digits 4 >> /data/.../STA_OUTPUT/sta_worst_max_slack.txt

  exec echo "$lib" >> /data/.../STA_OUTPUT/sta_worst_min_slack.txt
  report_worst_slack -min -digits 4 >> /data/.../STA_OUTPUT/sta_worst_min_slack.txt

  exec echo "$lib" >> /data/.../STA_OUTPUT/sta_tns.txt
  report_tns -digits 4 >> /data/.../STA_OUTPUT/sta_tns.txt

  exec echo "$lib" >> /data/.../STA_OUTPUT/sta_wns.txt
  report_wns -digits 4 >> /data/.../STA_OUTPUT/sta_wns.txt
}
```
- ✅ **Tip:** precreate `STA_OUTPUT/` and add `exec date >> file` lines for timestamps.

---

## 📂 Expected outputs — collect these files
- 📄 `STA_OUTPUT/min_max_<lib>.txt` — detailed per-corner path reports.  
- 📄 `STA_OUTPUT/sta_worst_max_slack.txt` — worst setup slack per corner (WNS).  
- 📄 `STA_OUTPUT/sta_worst_min_slack.txt` — worst hold slack per corner.  
- 📄 `STA_OUTPUT/sta_tns.txt` — Total Negative Slack per corner.  
- 📄 `STA_OUTPUT/sta_wns.txt` — Worst Negative Slack per corner.  
- 🖼 `opensta_console_<user>_<date>.png` — screenshot (include `whoami` + `date`).  
- 🖼 `critical_path_<corner>.png` — screenshot of a `report_timing`/critical path.

---

## 🔎 Example interpretation — per-path excerpt (how to read)
```
Cap     Delay    Time     Description
275.9346 2.5838  2.5838  ^ r2/Q (DFF_X1)
275.9392 2.5778  5.1617  ^ u1/Z (BUF_X1)
276.1091 2.7520  7.9137  ^ u2/ZN (AND2_X1)
...
10.0000  10.0000 10.0000 ^ r3/CK (DFF_X1)
-0.5697   9.4303  9.4303 data required time
--------------------------------------------
slack = required - arrival = 9.4303 - 7.9150 = 1.5153 (MET)
```
- 🔎 **Actionable:** large `Cap`/`Delay` rows indicate heavy nets — consider buffering, downsizing fanout, or changing cell to a faster drive.

---

## 📊 Example results table (sample)
- ✅ `tt_025C_1v80` — WNS: **+0.45 ns**, TNS: **0** → timing met.  
- ✅ `ff_100C_1v65` — WNS: **+0.62 ns**, TNS: **0** → faster corner.  
- ⚠ `ss_n40C_1v28` — WNS: **−0.37 ns**, TNS: **−1.10 ns** → timing violation (fix required).

---

## ✅ Final checklist before submission
- ☑ `vsdbabysoc.synth.v` is present and readable.  
- ☑ All targeted `.lib` files are in `timing_libs/`.  
- ☑ `vsdbabysoc_synthesis.sdc` contains correct clocks & exceptions.  
- ☑ `STA_OUTPUT/` contains `min_max_*.txt` and aggregated stats.  
- ☑ Screenshots for Output.

---


