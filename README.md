# FIR Filter – Verilog Implementation

This is a parameterized FIR (Finite Impulse Response) filter core implemented in Verilog, using **one multiplier**, **one adder**, and **AXI-compliant interfaces**.

## 📐 Specification

| Parameter     | Value            |
|--------------|------------------|
| `Data_Width` | 32 bits          |
| `Tap_Num`    | 11               |
| `Data_Num`   | Determined by input length |

---

## 🧩 Interface Description

### AXI-Stream Interfaces

- `data_in`  – Input stream (`X[n]`), standard AXI-Stream with `valid/ready` handshake  
- `data_out` – Output stream (`Y[n]`), standard AXI-Stream with `valid/ready` handshake  

### AXI-Lite Control Registers

| Signal     | Direction | Description                             |
|------------|-----------|-----------------------------------------|
| `coef[10:0]` | Write     | FIR tap coefficients (11 total), written via AXI-Lite |
| `len`      | Write     | Number of input samples to process      |
| `ap_start` | Write     | Start signal (asserted for 1 clock cycle) |
| `ap_done`  | Read      | High when FIR processing is complete    |

---

## 🧠 Architecture Overview

- **Computation Logic**:
  - Uses 1 multiplier and 1 adder for multiply-accumulate operations.
- **Shift Register**:
  - Implemented using on-chip SRAM (`Shift_RAM`)
  - Size: 10 words (1 word = 32 bits)
- **Coefficient Storage**:
  - Implemented using on-chip SRAM (`Tap_RAM`)
  - Size: 11 words, initialized via AXI-Lite write

---

## ⚙️ Operation Flow

1. **Initialize Coefficients**:  
   Write 11 FIR tap coefficients to `Tap_RAM` via AXI-Lite.

2. **Start FIR Engine**:  
   Set `len` to the number of samples, then pulse `ap_start` high for one clock.

3. **Streaming Input**:  
   Send input samples `X[n]` via `data_in`. Input rate is controlled via AXI-Stream handshake (`tvalid/tready`).

4. **Filter Processing**:  
   Each input shifts into `Shift_RAM`, and the core computes the output using the tap coefficients from `Tap_RAM`.

5. **Streaming Output**:  
   Filtered outputs `Y[n]` are sent to `data_out` using `tvalid/tready`.

6. **Complete**:  
   When all `len` samples are processed, `ap_done` is set to high.

---

## 📎 Notes

- The output rate is limited by FIR processing throughput.
- This implementation is suitable for synthesis on FPGAs with BRAM-based shift register and coefficient storage.
- Designed for SW/HW co-design environments like Xilinx Vitis or Intel HLS platforms.

---

# FIR Filter – Verilog Specification

## 🔧 Parameters

- **Data_Width**: 32  
- **Tap_Num**: 11  
- **Data_Num**: *TBD – Based on size of data file*

---

## 🔌 Interface

### Stream Interface
- `data_in`: stream (`X[n]`)
- `data_out`: stream (`Y[n]`)

### AXI-Lite Interface
- `coef[Tap_Num-1:0]`: FIR coefficients  
- `len`: number of input data samples  
- `ap_start`: start signal (valid for one clock cycle)  
- `ap_done`: processing complete flag  

---

## ⚙️ Architecture

- **Computation Unit**:  
  Uses **one multiplier** and **one adder**

- **Shift Register**:  
  Implemented with **SRAM**  
  - Module: `Shift_RAM`  
  - Size: 10 data words (10 DW)

- **Tap Coefficient Storage**:  
  Implemented with **SRAM**  
  - Module: `Tap_RAM`  
  - Size: 11 data words (11 DW)  
  - Initialized via **AXI-Lite write**

---

## 🚀 Operation Flow

- Pulse `ap_start` high for one clock to launch FIR engine  
- Stream input `X[n]` via `data_in`  
  - Input rate depends on FIR processing speed  
  - Use AXI-Stream `valid/ready` handshake for flow control  
- Stream output `Y[n]` via `data_out`  
  - Output rate also depends on FIR processing speed


## 📄 License

MIT License © 2024
