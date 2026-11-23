# Clock-Tree-Characterization-Optimization
## 1.1. Problem Statement:

Current designs use numerous CT cells with custom drive strengths that are set in the flow, increasing utilization and power. By selecting optimal drive strengths and relaxing net length constraints, we aim to reduce CT cell count and improve timing & power efficiency.

These experiments aim to characterize delay and power variation across corners for different CT cell drive strengths (f8, 110, 112) under varying net length scenarios and to recommend the optimal CT cell type and maximum net length for a Low Power Clock Tree Design

## 1.2. Basic Clock Tree Parameters

In Static Timing Analysis (STA), transition time (frequently referred to as slew) is the duration it takes for a signal to change its state from a logic low (0) to a logic high (1), or vice versa.

Transition time is not measured from absolute OV to absolute VDD. Instead, it is calculated as the time difference between the signal crossing two specific voltage thresholds. Common threshold pairs defined in standard cell libraries(lib) include:

Falling edge thresholds: slew lower threshold pct fall:

30.0; slew_upper_threshold_pct_fall: 70.0;

#Rising edge thresholds: slew_lower_threshold_pct_rise:

30.0; slew_upper_threshold_pct_rise: 70.0;

### 3.2.1. Cell delay:

In Static Timing Analysis (STA), cell delay (also known as gate delay or propagation delay) is the amount of time it takes for a signal to propagate from an input pin of a logic gate to its output pin. It represents the intrinsic speed of the logic gate itself.


### 3.2.2. How Is It Measured

Unlike transition time (which measures the time a signal takes to change states), cell delay measures the time difference between the input signal crossing a specific voltage threshold and the output signal crossing a specific voltage threshold.

The 50% Threshold: Most standard cell libraries measure cell delay from the point where the input signal reaches 50% of VDD to the point where the output signal reaches 50% of VDD. Depending on the logic function and the signal transition, cell delay is categorized into two types:

**Cell Rise Delay:** The delay when the output transitions from low to high (0 to 1).

**Cell Fall Delay:** The delay when the output transitions from high to low (1 to 0).


### 3.2.3. What Determines Cell Delay?

Cell delay is not a static number. In timing libraries (like Liberty lib files), the delay of a cell is dynamically calculated based on two primary factors:

**Input Transition Time (Slew):**

How fast the signal arriving at the input pin is changing. A slower input transition means the internal transistors take longer to fully switch on or off, increasing the cell delay

**Output Load Capacitance:**

The total capacitive load the cell's output must drive. This includes the wire capacitance of the net it is driving, plus the input pin capacitances of all the downstream cells (fan-out). A higher load requires the cell to charge or discharge more capacitance, slowing down the output transition and increasing the delay.


### 3.2.4. How Is It Modeled

During STA, the timing tool does not perform SPICE-level circuit simulations. Instead, it looks up the cell delay using predefined models provided by the foundry:

**Non-Linear Delay Model (NLDM):**

Uses 2D look-up tables where the X and Y axes are Input Transition and Output Load. The tool interpolates the exact delay based on these tables.

**Advanced Models (CCS/ECSM):**

Composite Current Source (CCS) and Effective Current Source Model (ECSM) are used in advanced technology nodes (usually below 65nm). Instead of simple voltage tables, they model the cell as a time-varying current source, providing much higher accuracy for ultra-deep submicron effects like the Miller effect and complex wire delays.


### 3.2.5. Cell Delay vs. Net Delay

To calculate the total delay of a path in STA, tools add two components together:

**Cell Delay:** The time taken through the gate.

**Net Delay:** The time taken for the signal to travel across the wire connecting one gate to the next.


### 3.2.6. Clock Distribution Hierarchy:

In clock tree synthesis, the terms "top," "trunk," and "leaf" nets refer to different parts of the clock tree:

**Top Nets:** These are the primary clock distribution paths, typically with the highest fanout. They are routed with special rules to ensure minimal

skew and optimal performance.

**Trunk Nets:** These are intermediate clock paths, connecting top nets to leaf nets. They have a lower fanout compared to top nets.

**Leaf Nets:** These are the final clock paths, directly connected to the clock pins of the flip-flops. They have the lowest fanout and are crucial for meeting timing constraints.

## 3.3. Automation of Clock Tree Parameter Extraction:

Explained the scripts and the usage in the latter part.

Developed an automation script to extract clock tree parameters like load, slew, tran, fanout, netlength, drive strength, for:

1. All the clock tree cells up to first generation points.

2. All the clock drivers in the design.

## 3.4. Optimal Constraints To Be Used

Different experiments had been conducted:

1. Swapping all CTI/CTB cells into different drive strengths (f8, f10, f12 one each time) but the variation across corners is similar in all three cell types when net length of 200 um is set in the flow. There isn't any cross corner criticality for f8 cell type also.

2. Increased max source to sink net length and performed different experiments.

Add below settings for low power clock tree design

in override/innovus_config.tcl after providing CTS options and in override/physical_design.tcl after delete_clock_tree_spec

set_db [get_db clock trees 200M].cts_top_fanout_threshold 30

set_db [get_db clock_trees 400M] .cts_top_fanout_threshold 30

set_db cts_inverter_cells_leaf (cti_f8_dh_126pp cti_f6_dh_126pp cti_f10_dh_126pp)

set_db cts_inverter_cells_top (cti_f10_dh_126pp cti_f12_dh_126pp)

set_db [get_db clock trees 200M] .cts_max_source_to_sink_net_length 240

set_db [get_db clock_trees 400M].cts_max_source_to_sink_net_length 240

More Netlength for top level is not suggested as complete timing will go on toss if there is cross talk. And Higher Drive Strength cells are to be used for Leaf level Cells since there can be high fanout
