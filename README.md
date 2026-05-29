# Traffic Light Controller - Verilog FSM

A finite state machine (FSM) implementation of a traffic light controller in Verilog, with a testbench and netlist visualization using Yosys and netlistsvg.

---

## Overview

This project models a 3-state traffic light controller using a Moore FSM. The controller cycles through Red -> Yellow -> Green with configurable hold times for each state, driven by a clock and active-high reset.

---

## File Structure

```
.
├── traffic_light.v      # Traffic light FSM module
└── testbench.sv         # Simulation testbench
```

---

## Module: `traffic_lights`

### Ports

| Port     | Direction | Type | Description                   |
|----------|-----------|------|-------------------------------|
| `clk`    | input     | wire | System clock                  |
| `reset`  | input     | wire | Active-high synchronous reset |
| `red`    | output    | reg  | Red light signal              |
| `yellow` | output    | reg  | Yellow light signal           |
| `green`  | output    | reg  | Green light signal            |

### Parameters

| Parameter     | Default | Description                 |
|---------------|---------|-----------------------------|
| `Red_Time`    | 5       | Clock cycles to hold RED    |
| `Yellow_Time` | 2       | Clock cycles to hold YELLOW |
| `Green_Time`  | 3       | Clock cycles to hold GREEN  |

---

## State Machine

```
         +----------------------------------+
         |                                  |
         v                                  |
   +-----------+   5 cycles   +------------+|
   |  S0: RED  | -----------> | S1: YELLOW ||
   +-----------+              +------------+|
         ^                          |
         |                          | 2 cycles
         |   3 cycles               v
         |              +------------------+
         +------------- |   S2: GREEN      |
                        +------------------+
```

| State | Encoding | Light  | Duration       |
|-------|----------|--------|----------------|
| S0    | `2'b00`  | Red    | 5 clock cycles |
| S1    | `2'b01`  | Yellow | 2 clock cycles |
| S2    | `2'b10`  | Green  | 3 clock cycles |

---

## Design Details

The FSM is split across four `always` blocks:

1. **Sequential block** — Updates `current_state` and `count` on the positive clock edge. Resets to `S0` on reset. Transitions to `next_state` when `count == limit - 1`.
2. **Next-state logic** — Combinational block: `S0 -> S1 -> S2 -> S0`.
3. **Timer limit logic** — Sets the hold duration (`limit`) based on `current_state`.
4. **Output logic** — Drives `red`, `yellow`, `green` based on `current_state` (Moore outputs).

---

## Simulation

### Tool Requirements

- [Icarus Verilog](http://iverilog.icarus.com/) (`iverilog`, `vvp`)

### Running the Simulation

```bash
# Compile
iverilog -Wall -g2012 traffic_light.v testbench.sv

# Run
vvp a.out
```

### Testbench Behaviour

- Clock period: 10 time units (`#5` half-period toggle)
- Reset asserted at `t=0`, de-asserted at `t=10`
- Simulation runs for 200 time units after reset (`#200 $finish`)
- Outputs a VCD file (`dump.vcd`) and a `$monitor` log to stdout

---

## Sample Output

```
Time = 0    Count = 0 || Red = 1  Yellow = 0  Green = 0
Time = 15   Count = 1 || Red = 1  Yellow = 0  Green = 0
...
Time = 55   Count = 0 || Red = 0  Yellow = 1  Green = 0
Time = 65   Count = 1 || Red = 0  Yellow = 1  Green = 0
Time = 75   Count = 0 || Red = 0  Yellow = 0  Green = 1
...
Time = 105  Count = 0 || Red = 1  Yellow = 0  Green = 0
```

One full cycle (Red -> Yellow -> Green) takes 10 clock cycles (5 + 2 + 3), repeating every 100 time units.

---

## Netlist Visualization

The synthesized netlist is generated using Yosys and rendered to an SVG using netlistsvg.

### Tool Requirements

- [Yosys](https://yosyshq.net/yosys/)
- [netlistsvg](https://github.com/nturley/netlistsvg)

### Steps

```bash
# Navigate to the project folder
cd ~/Desktop

# Open Yosys
yosys

# Inside Yosys:
read_verilog traffic_light.v
prep -top traffic_lights
write_json traffic_light.json
exit

# Generate the SVG netlist diagram
netlistsvg traffic_light.json -o traffic_light.svg

# Open the diagram
open traffic_light.svg
```

This produces `traffic_light.svg`, a visual diagram of the synthesized logic gates and flip-flops that make up the FSM.

---

## Customisation

To change light durations, modify the parameters in `traffic_light.v`:

```verilog
parameter Red_Time    = 5,
          Yellow_Time = 2,
          Green_Time  = 3;
```

---

## Notes

- Only one output (`red`, `yellow`, or `green`) is HIGH at any given time.
- The `count` register is 4 bits wide, supporting hold times up to 15 cycles.
- The `default` branch in all `case` statements prevents latches and handles undefined states.
