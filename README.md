# Red Pitaya Lock-in + PID Control System (Incubating)

> **Note**: This README specifically describes the specialized Lock-in + PID control system located in the `modeling/` and `rp_web/` directories. It is **not** the documentation for the upstream `RedPitaya-FPGA` project.

This project is an independent effort to build a high-performance, general-purpose control module for quantum optics experiments. While it currently resides within this repository for development, the long-term goal is to evolve this into a standalone project that is decoupled from the standard Red Pitaya FPGA ecosystem, allowing for easier adaptation to custom high-speed hardware.

## Project Vision

Modern quantum optics projects often require precise frequency stabilization and noise cancellation. While platforms like Red Pitaya and projects like PyRPL provide a wide range of functionalities, they are often too general or limited by the hardware's native SNR and sampling rates for specialized applications (e.g., frequency combs).

This project focuses on a streamlined, high-performance Lock-in + PID implementation that:
- **Improves SNR**: Uses a 6-stage CIC filter to handle the inherent noise of the RP 125-14.
- **Simplifies Adaptation**: Provides a modular core that is easily portable to custom PCBs and different ADC/DAC hardware.
- **Focuses on Utility**: Strips away redundant features to provide a robust foundation for specific control tasks.

## System Architecture

The system consists of three main layers: FPGA Core Logic, Web Visualization, and Configuration Scripts.

### 1. FPGA Core (RTL)
The heart of the system is the `feedback_controller_top` module, which includes:
- **Signal Model**: Simulates a physical system with built-in disturbance generators:
    - **0.5 Hz**: Ambient temperature drift simulation.
    - **330 Hz**: Mechanical vibration simulation.
    - **1.2 kHz**: Power supply noise simulation.
- **CIC Filter**: A 6-stage Cascaded Integrator-Comb filter with 125x decimation, reducing the 125 MSps input to a filtered 1 MSps stream for control.
- **PID Controller**: A standard PI control algorithm operating in the decimated 1 MHz clock domain, using Q0.16 fixed-point math with anti-windup.

### 2. Web Visualization (`rp_web`)
A real-time monitoring interface built with:
- **Backend (Rust)**: Maps physical memory (`/dev/mem`) to read the Red Pitaya scope buffers and serve data via a REST API.
- **Frontend (Vue 3 + uPlot)**: A high-performance dashboard for real-time wave observation.

### 3. Configuration & Scripts
Parameters such as PID gains, setpoints, and noise levels are managed via AXI4-Lite registers.
- **Base Address**: `0x40900000`
- **Control Scripts**: Located in `modeling/scripts/`, these scripts allow for quick switching between Open Loop, Closed Loop, and Bypass modes.

## Performance Demo & Setup

### Hardware Setup
For the standard demo, the **OUT1** and **IN1** ports on the Red Pitaya are connected via an SMA cable (Loopback). This allows the FPGA to inject simulated environment noise into the input path and then attempt to stabilize it using the PID controller.

### Results
In our testing on the Red Pitaya 125-14:
- **Stability**: The system can stabilize the output within **+/- 5mV** of the setpoint even under severe simulated noise conditions.
- **Noise Suppression**: The combination of CIC filtering and tuned PI control effectively suppresses mechanical and electrical noise, achieving excellent Allan deviation results for this hardware class. Even on a board with limited native SNR, the filtered control loop provides stable performance.

## Getting Started

### Prerequisites
- Red Pitaya (Z10 or Z20)
- Vivado (for FPGA synthesis)
- Rust toolchain (for web backend)
- Node.js (for web frontend)

### Build & Run
1. **Generate FPGA Bitstream**: Use the provided `.tcl` scripts to generate the Vivado project and bitstream for your specific board model.
2. **Deploy Web Server**: Navigate to `rp_web/` and follow the instructions to build and run the backend and frontend.
3. **Configure Control**: Use `modeling/script/setup_sim_waveform_tuned.sh` to initialize the control parameters.

## Future Roadmap
- [ ] **Interactive Web Control**: Move PID parameter tuning from shell scripts to the web interface.
- [ ] **Ramp Generation**: Add a frequency ramp feature to support Kerr Comb and other scanning-based projects.
- [ ] **Hardware Porting**: Further decoupling of the core logic to simplify migration to custom high-speed ADC/DAC boards.

## Acknowledgments
This implementation is based on several real-world quantum optics projects, aiming to provide a "prototype-to-production" path for control systems in the lab.
