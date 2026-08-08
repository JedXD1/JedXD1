# Verilog Learning Journey

This repository contains my hands-on Verilog exercises, examples, and mini-projects created while learning digital design and FPGA development.

The files are organized according to the concepts I studied, starting from Verilog fundamentals and progressing toward sequential logic, memory, FSMs, and a simple encryption project.

---

## 1. Verilog Fundamentals

- hello_world.v
- sum_product.v
- easy_verilog_example.v
- literal_values.v
- easy_vectors_example.v

### Operators

- bitwise_operators.v
- reduction_operators.v
- logical_operators.v
- logical_operators_usage.v
- math_operators.v
- shift_operators.v
- relational_operators.v
- equality_operators.v
- conditional_operators.v
- concatention_operator.v
- replication_operator.v
- operators_precedence.v

---

## 2. Basic Combinational Logic

### Gates

- built_in_gates.v
- tb_gates.v

### Multiplexers & Demultiplexers

- mux_1bit.v
- tb_mux.v
- demux_1bit.v
- tb_demux.v
- mux_4x_nbit.v
- demux_nbit_x4.v

### Tri-State Logic

- tristate_buffer_1bit.v
- tb_tristate.v
- mux_tristate.v

### Comparators

- comparator_1bit.v
- comparator_nbit.v
- compare_nbit_func.v

### Encoders & Decoders

- encoder_8to3.v
- prio_enc1_4to2.v
- decoder_nbit.v
- decoder_4to16.v

### Miscellaneous

- some_logic.v
- hex_7seg_decoder.v

---

## 3. Arithmetic Circuits

### Adders

- half_adder_structural.v
- half_adder_dataflow.v
- half_adder_behavioral.v
- full_adder_structural.v
- full_adder_dataflow.v
- ripple_adder_4bit_structural.v
- ripple_adder_4bit_dataflow.v
- adder_4bit_behavioral.v
- adder8bit.v
- adders3.v
- adders3_procedural.v
- adder_nbit.v

### Arithmetic Logic Unit (ALU)

- ALU.v
- ALU (1).v

---

## 4. Testbenches & Simulation

- my_first_testbench.v
- waveforms.v
- procedures_updated.v

---

## 5. Procedural Blocks, Functions & Tasks

### Functions

- function_ex1.v
- function_ex2.v
- function_ex3.v
- function_ex4.v

### Tasks

- task_meters_to_feet.v
- task_control_shift_reg.v

---

## 6. Sequential Logic

### Latches & Flip-Flops

- d_latch.v
- d_latch_rstn.v
- d_ff_sync_rstn.v
- d_ff_async_rstn.v

### Shift Registers

- shift_reg_siso.v
- shift_reg_sipo.v
- shift_reg_piso.v
- shift_reg_pipo.v
- shift_reg_pipo_buggy.v
- shift_left_right_load_reg.v

### Counters

- counter_nbit.v
- counter_modulo_n.v
- counter_up_down_load_nbit.v

### Clocking

- clkgen.v
- clock_div_nbit.v
- clock_div_3.v

### LFSR / Random Number Generation

- lfsr_16.v
- prng.v

---

## 7. Memory

### RAM

- ram_sp_async_read.v
- ram_sp_sync_read.v
- ram_dp_async_read.v
- ram_dp_async_read (1).v
- ram_dp_async_read (2).v

### ROM

- rom.v

---

## 8. Finite State Machines (FSM)

- fsm.v
- fsm_template.v
- semaphore_fsm.v
- seq_det_non_overlap.v
- seq_det_overlap.v
- top_fsm.v
- tb_top_fsm.v
- top_fsm (1).v
- tb_top_fsm (1).v

---

## 9. FIFO Design

- fifo_sync.v
- fifo_sync (1).v

---

## 10. Mini Project - Encryption System

- top_encrypt.v
- tb_encrypt.v
- top_encrypt_golden.v

---

## Learning Progress

✅ Verilog Basics  
✅ Operators  
✅ Combinational Logic  
✅ Arithmetic Circuits  
✅ Testbenches  
✅ Functions & Tasks  
✅ Sequential Logic  
✅ State Machines  
✅ Memory Design  
✅ FIFO Design  
✅ Mini Project Development

---

### Goal

To build a strong foundation in RTL design and digital systems using Verilog, progressing from basic syntax to complete hardware modules and projects.
