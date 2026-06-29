# APB-BASED-SPI-MASTER-IP-CORE
## Author: Ramya Sheshadri

## Project Overview

This project focuses on the RTL design and verification of an **APB-based SPI Master Controller** using Verilog HDL. The SPI Master is designed as a modular architecture consisting of four major sub-blocks, each responsible for a specific functionality in the SPI communication protocol.

### SPI Master Sub-blocks

#### 1. Baud Rate Generator

* Generates the SPI Serial Clock (**SCLK**) by dividing the system clock (**PCLK**).
* Supports configurable baud rates through programmable divider values.

#### 2. Shift Register

* Performs serial transmission and reception of data.
* Handles data shifting on the **MOSI** line and sampling on the **MISO** line.
* Enables **full-duplex communication**, allowing simultaneous transmission and reception.

#### 3. APB Slave Interface

* Acts as the interface between the CPU and the SPI Master.
* Contains all the configuration registers such as Control Registers, Baud Rate Register, Status Register and Data Register.
* Allows the CPU to configure SPI operating modes, baud rate, data transfer, and monitor status through the APB protocol.

#### 4. Slave Select Controller

* Generates the **Slave Select (SS)** signal.
* Initiates SPI communication by selecting the slave device before data transmission begins.
* Controls the transfer duration based on the programmed baud rate.

---

## Verification

Each RTL sub-block was individually verified using **ModelSim** before integrating them into the complete SPI Master. Functional simulations were performed to verify correct APB transactions, SPI data transfer, clock generation, and slave selection.

---

## Linting

RTL linting was performed using **Synopsys VC SpyGlass** to ensure the design follows recommended coding practices and is free from structural issues before simulation.

The linting process checks for:

* Coding style violations
* Synthesis-related issues
* Unsafe RTL constructs
* Design rule violations

---

## Synthesis

RTL synthesis was performed using **Synopsys Design Compiler** to convert the Verilog RTL into a technology-mapped gate-level netlist.

During synthesis, the following concepts were explored:

* Writing synthesis **TCL scripts**
* Applying timing constraints
* Technology library mapping
* Understanding the conversion from an **unmapped DDC** (generic design representation without timing, area, or delay information) to a **mapped DDC** (technology-specific implementation with complete timing, area, and power information)

---

## Skills Gained

Through this project, I gained practical experience in:

* RTL Design using Verilog HDL
* APB Protocol Implementation
* SPI Master Architecture
* Clock Generation and Baud Rate Configuration
* Full-Duplex SPI Data Transfer
* Functional Verification using ModelSim
* RTL Linting using Synopsys VC SpyGlass
* RTL Synthesis using Synopsys Design Compiler
* Writing and understanding TCL scripts
* Writing SpyGlass Lint project files
* Timing, Area and Gate-Level Analysis

## Introduction

This section provides the background behind the APB-based SPI Master IP Core, explains the motivation for integrating APB with SPI, and describes how the complete communication flow operates within an SoC.

Modern **System-on-Chip (SoC)** designs integrate multiple processing units, memories, and peripherals on a single chip. Efficient communication between these components is achieved using standardized bus protocols. While high-performance buses such as AXI are used for memory-intensive operations, low-speed peripherals like SPI, UART, GPIO, and timers are commonly connected through the Advanced Peripheral Bus (APB), which offers a simpler and lower-power interface.

The **APB Interfaced SPI Master IP Core** is a Verilog-based RTL design that **enables communication between an APB bus and SPI-compatible peripheral devices**. It acts as an SPI Master, allowing a processor to control and exchange data with external SPI slaves through the APB interface.

The APB interface is used to configure the SPI controller, write data for transmission, and read the received data through memory-mapped registers. Internally, the SPI Master generates the serial clock, controls the chip select signal, and performs serial data transmission and reception according to the SPI protocol.

This project demonstrates the integration of an on-chip communication protocol (APB) with an off-chip communication protocol (SPI) using a modular RTL design approach. It is intended to provide a simple and reusable SPI Master IP Core suitable for SoC and embedded system applications.

## Example: CPU Communicating with a Temperature Sensor

To understand the complete communication flow, let us consider a simple example where a **CPU** communicates with an external **temperature sensor** using the **SPI** protocol.

1. The CPU initiates a read or write operation by sending an **AXI** transaction.
2. The **AXI-to-APB Bridge** converts the AXI transaction into an APB transaction.
3. The APB transaction is transferred over the **APB Bus** to the **SPI Master IP**.
4. The SPI Master reads the APB control registers, generates the required **Serial Clock (`SCLK`)**, and asserts the **Chip Select (`SS`)** signal.
5. Using the **MOSI**, **MISO**, **SCLK**, and **SS** lines, the SPI Master communicates with the **SPI Slave** connected to the temperature sensor.
6. The temperature sensor sends the requested data back to the SPI Master through the **MISO** line.
7. The SPI Master stores the received data in its data register.
8. Finally, the CPU reads the received data through the APB interface.


## 1. Baud Generator (Serial Clock Generator)

The **Baud Generator** is responsible for generating the SPI serial clock (`SCLK`) required for communication between the SPI Master and SPI Slave. Since the system clock (`PCLK`) is much faster than the clock supported by most SPI peripherals, the Baud Generator divides the input clock to produce the desired SPI clock frequency.

In addition to generating `SCLK`, it also generates the internal timing signals required for transmitting (`MOSI`) and receiving (`MISO`) data according to the selected SPI mode.

### Functions

- Generates the SPI serial clock (`SCLK`).
- Divides the system clock (`PCLK`) to the required SPI frequency.
- Supports all four SPI modes using **CPOL** and **CPHA**.
- Generates separate timing signals for data transmission and reception.
- Synchronizes MOSI shifting and MISO sampling.

## Block diagram of the Baud Generator block:
<img width="257" height="179" alt="image" src="https://github.com/user-attachments/assets/6af4cc22-1421-4235-bcc8-dc05feec7672" />


### Inputs

| Signal | Description |
|---------|-------------|
| `PCLK` | System clock used for clock generation. |
| `PRESETn` | Active-low reset. |
| `sppr_i[2:0]` | Baud rate prescaler. |
| `spr_i[2:0]` | Baud rate select bits. |
| `cpol_i` | Clock polarity selection. |
| `cpha_i` | Clock phase selection. |
| `ss_i` | Slave Select signal. Clock generation starts when this signal is asserted. |

### Outputs

| Signal | Description |
|---------|-------------|
| `sclk_o` | SPI Serial Clock output. |
| `mosi_send_sclk_o` | Transmit timing signal for MOSI. |
| `mosi_send_sclk0_o` | Alternate transmit timing signal. |
| `miso_receive_sclk_o` | Receive timing signal for MISO. |
| `miso_receive_sclk0_o` | Alternate receive timing signal. |
| `BaudRateDivisor_o` | Final clock divider value used to generate `SCLK`. |


### Working

1. The system clock (`PCLK`) is provided to the Baud Generator.
2. The baud rate configuration (`sppr_i` and `spr_i`) determines the required clock division factor.
3. The divider generates a lower-frequency SPI clock (`SCLK`).
4. The idle state of `SCLK` is determined by **CPOL**.
5. Based on **CPHA**, separate transmit and receive timing signals are generated.
6. These timing signals are used by the Shift Register to control MOSI transmission and MISO sampling.

### Key Features

- Programmable SPI clock generation.
- Supports all SPI modes (Mode 0–3).
- Configurable baud rate.
- Separate timing signals for transmit and receive operations.
- Provides synchronized clocking for reliable SPI communication.

## Output waveform:
Case 1: CPOL=CPHA
<img width="955" height="361" alt="image" src="https://github.com/user-attachments/assets/5a16ab8b-cd5d-4724-806f-f0f60e50626d" />

Case 2: CPOL ≠ CPHA
<img width="955" height="358" alt="image" src="https://github.com/user-attachments/assets/7c3e7e6b-7d9a-4790-80ed-166cc2eb69fc" />

## Inference:
### Waveform Observations

The simulation waveform verifies the correct operation of the **Baud Generator** and its timing signals.

- The generated **SPI Serial Clock (`SCLK`)** is significantly slower than the system clock (`PCLK`), confirming that the baud generator correctly divides the input clock to generate the required SPI clock.

- SPI activity begins only when the **Slave Select (`SS`)** signal is asserted low (`ss_i = 0`). This ensures that clock generation and SPI communication occur only when a slave device is selected.

- When **CPOL = CPHA** (**SPI Mode 0** and **SPI Mode 3**):
  - `mosi_send_sclk_o` is used as the transmit timing signal.
  - `miso_receive_sclk_o` is used as the receive timing signal.
  - These signals correspond to the clock edge defined for Modes 0 and 3.

- When **CPOL ≠ CPHA** (**SPI Mode 1** and **SPI Mode 2**):
  - `mosi_send_sclk0_o` is used as the transmit timing signal.
  - `miso_receive_sclk0_o` is used as the receive timing signal.
  - These signals correspond to the alternate clock edge required for Modes 1 and 2.

- The **MOSI** transmit pulse occurs before the **MISO** receive pulse. This ensures that data is placed on the bus before it is sampled by the receiving device, satisfying SPI timing requirements.

- The waveform confirms that the baud generator correctly switches between the two timing paths (`*_sclk_o` and `*_sclk0_o`) based on the selected **CPOL** and **CPHA** configuration.

### Verification Summary

The simulation verifies the following functionalities of the Baud Generator:

- Correct SPI clock (`SCLK`) generation
- Proper clock division from `PCLK`
- Slave Select (`SS`) controlled SPI operation
- Correct CPOL and CPHA mode selection
- Accurate transmit timing signal generation
- Accurate receive timing signal generation
- Proper shift-before-sample timing sequence for reliable SPI communication

## 2. Shift Register

The **Shift Register** is the primary data-path block of the SPI Master. It is responsible for transmitting data from the Master to the Slave through the **MOSI** line and receiving data from the Slave through the **MISO** line. It performs parallel-to-serial conversion during transmission and serial-to-parallel conversion during reception.

## Block diagram of shift register:
<img width="237" height="167" alt="image" src="https://github.com/user-attachments/assets/cae9e7c4-fd84-4e86-9081-e2c0855415f0" />


### Functions

- Transmits data from the SPI Master to the SPI Slave.
- Receives data from the SPI Slave.
- Converts parallel data into serial format during transmission.
- Converts serial data into parallel format during reception.
- Supports both **MSB-first** and **LSB-first** data transmission.
- Performs shifting and sampling based on timing signals generated by the Baud Generator.

### Data Flow

#### Transmission

```text
CPU
 │
 ▼
APB Interface
 │
 ▼
Shift Register
 │
 ▼
MOSI
 │
 ▼
SPI Slave
```

#### Reception

```text
SPI Slave
 │
 ▼
MISO
 │
 ▼
Shift Register
 │
 ▼
APB Interface
 │
 ▼
CPU
```

### Inputs

| Signal | Description |
|---------|-------------|
| `PCLK` | System clock used for sequential operations. |
| `PRESET_n` | Active-low reset signal. |
| `ss_i` | Slave Select signal. Transmission and reception occur only when `ss_i = 0`. |
| `lsbfe_i` | Selects data transmission order (MSB-first or LSB-first). |
| `cpol_i` | Determines the idle state of the SPI clock. |
| `cpha_i` | Determines the clock edge used for shifting and sampling. |
| `mosi_send_sclk_i` | Positive-edge transmit timing signal from the Baud Generator. |
| `mosi_send_sclk0_i` | Negative-edge transmit timing signal from the Baud Generator. |
| `miso_receive_sclk_i` | Positive-edge receive timing signal from the Baud Generator. |
| `miso_receive_sclk0_i` | Negative-edge receive timing signal from the Baud Generator. |
| `data_mosi_i[7:0]` | Parallel transmit data received from the APB Interface. |
| `miso_i` | Serial data received from the SPI Slave. |
| `send_data_i` | Enables data transmission. |
| `receive_data_i` | Enables data reception. |

### Outputs

| Signal | Description |
|---------|-------------|
| `mosi_o` | Serial transmit data sent to the SPI Slave. |
| `data_miso_o[7:0]` | Parallel data reconstructed from the received serial data. |

### Internal Working

1. The CPU writes transmit data through the APB Interface.
2. The parallel data is loaded into the Shift Register.
3. The Baud Generator provides timing signals based on the selected SPI mode.
4. On each transmit clock edge, one bit is shifted out through the `MOSI` line.
5. Incoming serial data on the `MISO` line is sampled on the appropriate receive clock edge.
6. After all bits have been received, the Shift Register reconstructs the complete parallel data word.
7. The received data is transferred back to the APB Interface for the CPU to read.

### Data Order Selection

The transmission order is controlled by the **LSB First Enable (`lsbfe_i`)** configuration bit.

- **`lsbfe_i = 0`** : Data is transmitted **MSB First**.
- **`lsbfe_i = 1`** : Data is transmitted **LSB First**.

The Shift Register does not determine the transmission order itself. It simply follows the configuration provided by the APB control register.
## Output waveform:
<img width="959" height="407" alt="image" src="https://github.com/user-attachments/assets/f000ad88-35dd-40a9-84ad-7daefad8576a" />

## Inference:
### Waveform Observations

The simulation waveform verifies the correct operation of the **Shift Register** during both transmission and reception.

- The transmit data `data_mosi_i` is loaded with the value **`0xA5`** (`10100101`) before the SPI transaction begins.

- During transmission, the loaded data is temporarily stored in the internal **`shift_register_s`** register. This register shifts one bit at a time onto the **`MOSI`** line whenever a transmit timing pulse is generated.

- On the receive side, serial data arriving on the **`MISO`** line is sampled and temporarily stored in the internal **`temp_reg_s`** register. After all eight bits are received, the reconstructed byte is transferred to **`data_miso_o`**.

- Since **LSB First Enable (`lsbfe_i`)** is asserted, both transmission and reception occur in **LSB-first** order.

- The internal counters **`count_s`** and **`count2_s`** operate as **up counters**, incrementing from **0** to **7** to keep track of the number of transmitted and received bits.

- The waveform shows that:
  - `shift_register_s` contains the transmit data (`0xA5`) before shifting begins.
  - `temp_reg_s` gradually accumulates the received bits.
  - `data_miso_o` is updated only after all eight bits have been sampled.
  - The transmit and receive operations are synchronized using the timing signals generated by the Baud Generator.

### Verification Summary

The waveform confirms the following:

- Correct loading of transmit data into `shift_register_s`.
- Correct serial transmission of data through the `MOSI` line.
- Correct sampling of incoming serial data from the `MISO` line.
- Proper reconstruction of the received byte in `temp_reg_s`.
- Successful transfer of the received byte to `data_miso_o`.
- Correct LSB-first data transfer.
- Proper operation of the transmit (`count_s`) and receive (`count2_s`) bit counters.

## 3. APB Slave Interface

- The **APB Slave Interface** acts as the communication bridge between the **CPU** and the **SPI Master**. It is responsible for receiving configuration commands and data from the processor through the APB bus, storing them in internal registers, and distributing the required control signals to the SPI hardware blocks.

- The CPU never communicates directly with the Baud Generator, Shift Register, or SPI Control Logic. Instead, every SPI operation begins with an APB transaction, making the APB Slave Interface the central control block of the design.

- APB Slave Interface contains all the CPU Configuration settings. The CPU communicates everything via this block. For shifting, CPU writes to APB Slave’s Data register, which is copied onto the Shift register and sent to slave via MOSI.
While receiving, slave writes to shift register, which is sampled , via MISO and then sent to APB Slave, which is read by CPU.


## Block diagram of APB Slave Interface:
<img width="345" height="234" alt="image" src="https://github.com/user-attachments/assets/66b2cfe7-4a3e-4895-9f24-fcc71a15a1d4" />


### Responsibilities

The APB Slave Interface performs the following functions:

- Implements the APB protocol for register read and write operations.
- Stores SPI configuration parameters in memory-mapped registers.
- Generates control signals required by the Baud Generator and Shift Register.
- Transfers transmit data from the CPU to the Shift Register.
- Stores received data from the Shift Register for CPU access.
- Updates SPI status information during data transfer.
- Generates interrupt requests whenever SPI events occur.

## Internal Registers

The APB Slave Interface contains five memory-mapped registers.

### i. Control Register 1 (CR1)

Stores the primary SPI configuration parameters such as:

- Clock Polarity (**CPOL**)
- Clock Phase (**CPHA**)
- LSB First Enable (**LSBFE**)
- SPI Enable (**SPE**)

The values stored in this register are forwarded to the Baud Generator and Shift Register to configure the SPI communication mode.

### ii.Control Register 2 (CR2)

Stores additional SPI control information, including:

- Master/Slave configuration
- Interrupt enable bits
- Mode fault detection
- Wait mode configuration (`SPISWAI`)

These settings determine the operating mode of the SPI controller.

### iii.Baud Rate Register (BR)

Stores the baud-rate configuration values.

- Baud Rate Prescaler (**SPPR**)
- Baud Rate Select (**SPR**)

These values are provided to the **Baud Generator**, which calculates the SPI serial clock frequency.

### iv.Status Register (SR)

Contains status information generated during SPI communication.

Typical status flags include:

- Transfer In Progress (**TIP**)
- Transfer Complete (**SPIF**)
- Transmit Buffer Empty (**SPTEF**)
- Mode Fault (**MODF**)

Since this register contains status information generated by hardware, it is **read-only**.

### v.Data Register (DR)

The Data Register is used for both transmission and reception.

During transmission, the CPU writes an 8-bit data value into the Data Register. The APB Slave Interface forwards this data to the Shift Register, which serializes it and transmits it through the MOSI line.

During reception, the Shift Register reconstructs the received serial data into an 8-bit value and stores it back into the Data Register. The CPU can then read this value through the APB interface.

### APB Transaction

Every APB transaction follows three phases:

- **IDLE** – The interface waits for a valid APB transaction.
- **SETUP** – The address and control signals are decoded.
- **ENABLE** – The actual read or write operation is performed.

For a **write operation**, the selected register is updated with the value present on `PWDATA`.

For a **read operation**, the contents of the selected register are placed on `PRDATA` and returned to the CPU.

### Interaction with Other Blocks

The APB Slave Interface communicates with the remaining blocks of the SPI Master as follows:

- Sends **CPOL**, **CPHA**, **SPPR**, and **SPR** values to the **Baud Generator**.
- Sends **LSBFE**, transmit data, and control signals to the **Shift Register**.
- Receives completed data from the **Shift Register** and stores it in the Data Register.
- Updates the Status Register based on transfer status.
- Generates interrupt requests whenever an SPI event such as transfer completion or mode fault occurs.
  
## Output waveform:
<img width="952" height="460" alt="image" src="https://github.com/user-attachments/assets/e8d2bcc0-432c-481e-8ae6-cf3abd9da159" />
<img width="946" height="113" alt="image" src="https://github.com/user-attachments/assets/c56e39a3-1fd8-4185-aeab-ff19cdebf4fb" />


## Inference:
### Waveform Observations

The simulation waveform verifies the correct operation of the **APB Slave Interface** and confirms successful APB communication with the SPI Master.

- **Reset Operation**
  - `PRESET_n` is initially asserted low.
  - All internal registers are reset to their default values before any APB transaction begins.

- **APB Write Transactions**
  - The `PSEL_i` and `PENABLE_i` signals correctly follow the APB transaction sequence (**SETUP → ENABLE**).
  - Configuration data is successfully written into the SPI registers.

- **Control Register 1 (CR1)**
  - The value **`0x5C`** (`01011100`) is successfully written into CR1.
  - The waveform confirms:
    - `mstr_o = 1` (Master Mode enabled)
    - `cpol_o = 1` (Clock Polarity = 1)
    - `cpha_o = 1` (Clock Phase = 1)
    - `lsbfe_o = 0` (MSB First transmission)

- **Control Register 2 (CR2)**
  - The value **`0x12`** (`00010010`) is successfully written into CR2.
  - The waveform confirms:
    - `MODFEN = 1` (Mode Fault Detection enabled)
    - `spiswai_o = 1` (SPI Stop in Wait Mode enabled)

- **Baud Rate Register (BR)**
  - The value **`0x75`** (`01110101`) is successfully written.
  - The generated baud-rate configuration is:
    - `sppr_o = 111`
    - `spr_o = 101`
  - These values are forwarded to the Baud Generator for SPI clock generation.

- **Data Register (DR)**
  - The transmit data **`0xA5`** (`10100101`) is successfully written into the Data Register.
  - The data is forwarded to the Shift Register for serial transmission.

- **SPI Mode FSM**
  - The SPI Mode FSM transitions correctly between:
    - `spi_run`
    - `spi_wait`
    - `spi_run`
  - This verifies proper SPI operating mode and wait-mode control.

- **Transmit Operation**
  - `send_data_o` generates a valid transmit request pulse.
  - `data_mosi_o` carries the expected transmit data (`0xA5`).
  - This confirms successful transfer of data from the Data Register to the Shift Register.

- **APB Read Transactions**
  - Register contents are correctly returned through `PRDATA_o`.
  - The following values are successfully read back:
    - `CR1 = 0x5C`
    - `CR2 = 0x12`
    - `BR = 0x75`
  - After the read operation completes, `PRDATA_o` returns to zero, confirming that it is driven only during an active read transaction.

### Verification Summary

The waveform confirms the following functionalities of the APB Slave Interface:

- Correct reset operation
- Successful APB write transactions
- Successful APB read transactions
- Correct register decoding
- Proper loading of CR1, CR2, BR, and DR registers
- Correct SPI configuration signal generation
- Proper baud-rate configuration
- Correct SPI Mode FSM operation
- Successful transmit data handling
- Proper APB protocol implementation

## 4. SPI Slave Control Select

The **SPI Slave Control Select** block is responsible for controlling the overall SPI transaction. It receives configuration and control signals from the APB Slave Interface and generates the control signals required to start, monitor, and terminate an SPI transfer.

Unlike the Baud Generator and Shift Register, this block does **not** generate the SPI clock or transfer data. Instead, it acts as a transaction controller by selecting the slave, monitoring the transfer duration, and indicating when a transfer is in progress or completed.

## Block diagram:
<img width="485" height="203" alt="image" src="https://github.com/user-attachments/assets/db39efac-e03d-4533-9266-a4baed180e74" />

### Functions

- Initiates an SPI transaction upon receiving a transmit request.
- Generates the active-low Slave Select (`ss_o`) signal.
- Indicates transfer status using the `tip_o` (Transfer In Progress) signal.
- Generates a receive-complete flag (`receive_data_o`) after the transaction finishes.
- Uses the programmed baud-rate divisor to determine the transaction duration.
- Supports different SPI operating modes through the `spi_mode_i` input.

### Working

When the APB Slave Interface asserts `send_data_i`, the Slave Control Select block begins an SPI transaction by asserting `ss_o` low, selecting the target SPI slave.

During the transaction:

- `tip_o` remains high, indicating that a transfer is in progress.
- An internal counter tracks the transfer duration based on the programmed baud-rate divisor.
- Once the required number of clock cycles has elapsed, the block asserts `receive_data_o` to indicate that valid received data is available.
- Finally, `ss_o` is deasserted, `tip_o` returns low, and the SPI Master returns to the idle state.

### Inputs

| Signal | Description |
|---------|-------------|
| `PCLK` | System clock. |
| `PRESET_n` | Active-low reset. |
| `mstr_i` | Master/Slave configuration input. |
| `send_data_i` | Requests the start of an SPI transfer. |
| `spiswai_i` | Controls SPI operation during wait mode. |
| `spi_mode_i` | Selects the SPI operating mode. |
| `baudratedivisor_i` | Determines the transfer duration. |

### Outputs

| Signal | Description |
|---------|-------------|
| `ss_o` | Active-low Slave Select signal. |
| `tip_o` | Indicates that an SPI transfer is in progress. |
| `receive_data_o` | Indicates that the received data is ready to be stored by the APB Interface. |

## Output waveform:
- The counter starts only when send_data flag goes high. Slave select is pulled low, and TIP is high (TIP=~SS)
<img width="952" height="341" alt="image" src="https://github.com/user-attachments/assets/fea9f157-3628-41a7-8da8-9e5db4b9bfc2" />

- It counts upto specified target, and then resets. Here receive_data flag goes high, indicating it has received the data.
<img width="955" height="337" alt="image" src="https://github.com/user-attachments/assets/273c58dc-5d0a-45d6-b8d4-c4017bfdf497" />

### Waveform Observations

The simulation waveform verifies the correct operation of the **SPI Slave Control Select** block.

- After reset is released and **Master Mode** (`mstr = 1`) is enabled, a `send_data` pulse initiates the SPI transaction by asserting `ss_o = 0`, resetting the internal counter, and setting `tip_o = 1` to indicate that a transfer is in progress.

- During the transaction, the internal counter increments on every `PCLK` cycle while `ss_o` remains low and `tip_o` remains high. The transfer duration is determined by the programmed baud-rate divisor (`target = baudrate_divisor × 16`).

- When the counter reaches **`target - 1`**, the `receive_data_o` signal is asserted, indicating that valid received data is available in the Shift Register and can be transferred to the Data Register.

- When the counter reaches the programmed **target** value, the SPI transaction is completed. The slave is deselected (`ss_o = 1`), `tip_o` is deasserted, and the counter returns to its idle value.

### Verification Summary

The waveform confirms:

- Correct generation of the Slave Select (`ss_o`) signal.
- Proper assertion and deassertion of the Transfer In Progress (`tip_o`) signal.
- Correct counter-based transaction timing.
- Proper generation of the `receive_data_o` flag after transfer completion.
- Successful SPI transaction control for multiple SPI operating modes.

## Top Module

The **Top Module** integrates all the individual RTL blocks to form the complete **APB Interfaced SPI Master IP Core**. Each block performs a specific function and communicates with the others through well-defined control and data signals.

Although the SPI architecture supports both **Master** and **Slave** modes through the **MSTR** configuration bit, this project implements **Master Mode only**. The SPI Slave functionality is not included, as no slave-side interface has been implemented.

## Block diagram of SPI Master IP Core:
<img width="412" height="188" alt="image" src="https://github.com/user-attachments/assets/ba5709d6-9d06-4809-8956-ff1dff009d8d" />

The interaction between the blocks is as follows:

- The **APB Slave Interface** receives configuration and data from the CPU through the APB bus.
- The **Baud Rate Register** inside the APB Slave Interface provides the baud-rate configuration (`SPPR` and `SPR`) to the **Baud Generator**.
- The **Baud Generator** divides the system clock (`PCLK`) and generates the required **SPI Serial Clock (`SCLK`)** along with the transmit and receive timing signals.
- The **Shift Register** uses the generated timing signals together with the configured **CPOL** and **CPHA** values to determine the correct clock edge for transmitting (`MOSI`) and sampling (`MISO`) data.
- The **Slave Control Select** block controls the SPI transaction by asserting the Slave Select signal (`SS`), indicating the transfer status (`TIP`), and generating the receive-complete signal once the transfer is finished.

Together, these blocks provide a complete APB-controlled SPI Master capable of configuring, transmitting, and receiving SPI data.

## Waveform:
<img width="905" height="415" alt="image" src="https://github.com/user-attachments/assets/b8b700c9-09fe-4d88-92ce-6a16ba623a35" />

### Waveform Observations

The simulation waveform verifies the successful integration and operation of the complete **APB Interfaced SPI Master IP Core**.

- After `PRESET_n` is deasserted, the SPI Core comes out of reset and the CPU configures the SPI registers through the APB interface.
- The APB address sequence correctly accesses **CR1**, **CR2**, **Baud Rate Register**, and **Data Register**, while `PSEL` and `PENABLE` follow the standard APB protocol.
- The SPI Core is configured in **Master Mode** (`MSTR = 1`), the baud-rate settings are loaded into the Baud Generator, and the transmit data (`0xAA`) is written into the Data Register.
- The SPI transaction begins by asserting **Slave Select (`SS`)** low and setting **Transfer In Progress (`TIP`)** high. The Baud Generator starts producing the SPI serial clock (`SCLK`).
- The Shift Register transmits one bit on the **MOSI** line during each clock cycle, completing a full **8-bit SPI frame**.
- The generated **SCLK** toggles only while the SPI transfer is active and remains idle once the transaction is complete.
- After all eight bits are transmitted, **SS** returns high, **TIP** is deasserted, and the SPI transaction terminates successfully.
- During APB accesses, `PREADY` is asserted to indicate successful transactions. The observed `PSLVERR` pulse is due to the RTL implementation (`PSLVERR = ~tip_i` during the ENABLE phase) and does not indicate an actual SPI communication error.
- The `spi_interrupt_request` signal is asserted after the transfer, indicating successful completion of the SPI transaction.

### Verification Summary

The waveform confirms:

- Correct APB configuration and register programming.
- Successful baud-rate configuration and SPI clock generation.
- Proper Slave Select (`SS`) and Transfer In Progress (`TIP`) control.
- Correct transmission of one complete 8-bit SPI frame.
- Successful interaction between all RTL blocks.
- Correct end-to-end APB-to-SPI communication.

# Synthesis using SDC:

After successful synthesis using **Synopsys Design Compiler**, the RTL design is converted into a technology-mapped gate-level implementation.

### Unmapped Design

Before technology mapping, the synthesized design exists as an **unmapped DDC (Design Compiler Database)**. At this stage:

* The design consists of generic logic cells (GTECH cells).
* No standard-cell library has been applied.
* Cell delay, area, and power information are not available.
* The design only represents the logical functionality of the RTL.
---
### Mapped Design

After linking the target technology library and compiling the design, the unmapped design is converted into a **mapped DDC**.

During technology mapping:

* Generic logic is replaced with technology-specific standard cells.
* Timing, area, and power information are assigned.
* Flip-flops, multiplexers, buffers, and combinational logic are mapped to cells available in the target library.
* The synthesized design becomes suitable for timing analysis and physical implementation.

The mapped hierarchical schematic below shows the interconnection between the four SPI Master sub-blocks:

* **APB Slave Interface**
* **Slave Select Controller**
* **Baud Generator**
* **Shift Register**

**Mapped Hierarchical Schematic**
<img width="716" height="278" alt="Screenshot 2026-06-29 112722" src="https://github.com/user-attachments/assets/0738a99e-2f8c-49db-8f50-b39c7cfc37d0" />

<img width="927" height="263" alt="Screenshot 2026-06-29 112801" src="https://github.com/user-attachments/assets/6e0219ee-30e5-483e-b130-a880eefa8053" />


### Outcome of Synthesis

Through synthesis, the following were obtained:

* Technology-mapped gate-level netlist
* Area report
* Timing report
* Power report
* Hierarchical mapped schematic

The synthesis process also provided practical understanding of:

* Technology library mapping
* Standard-cell based implementation
* Timing constraints using TCL scripts
* Gate-level optimization for area and timing

### RTL Linting: 

The RTL design was verified using **Synopsys VC SpyGlass** before simulation and synthesis to identify coding issues and ensure compliance with industry-standard RTL design guidelines.

The generated **Summary Report** classifies messages into four categories:

* **Info** – Informational messages about the design.
* **Warning** – Potential coding issues that may affect design quality or portability.
* **Error** – RTL issues that must be corrected before proceeding.
* **Fatal** – Critical issues that prevent successful analysis.

## Linting report:
<img width="665" height="482" alt="image" src="https://github.com/user-attachments/assets/c1f3a99b-df47-469f-b025-2d55c2744dcf" />

The report shown above contains:

* **2 Informational Messages**

  * **DetectTopDesignUnits** – Successfully identified the top-level design unit.
  * **ElabSummary** – Successfully elaborated the RTL design.

* **1 Warning**

  * **STARC05-1.3.1.3** – Indicates that the asynchronous reset signal (`PRESET_n`) is also used as a normal combinational signal. According to the STARC RTL coding guidelines, an asynchronous reset should only be used for reset logic and not for combinational or synchronous logic.

No **Errors** or **Fatal** issues were reported, indicating that the design is functionally suitable for simulation and synthesis.

**Key Learning:**

* Performed RTL linting using Synopsys VC SpyGlass.
* Understood industry-standard RTL coding guidelines (STARC Rules).
* Identified and analyzed coding quality issues before synthesis.
* Learned to interpret SpyGlass summary reports and rule violations.

## Conclusion

This project successfully demonstrates the design and implementation of an **APB-interfaced SPI Master IP Core** using Verilog HDL. The design was modularly developed, functionally verified using **ModelSim**, analyzed using **Synopsys VC SpyGlass** for RTL linting, and synthesized using **Synopsys Design Compiler**. Through this project, I gained hands-on experience in RTL design, APB-SPI protocol integration, functional verification, linting, synthesis, TCL scripting, and technology mapping, providing a strong foundation in digital VLSI design and design flow.
