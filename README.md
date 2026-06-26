# APB-BASED-SPI-MASTER-IP-CORE

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

## Block diagram:
<img width="512" height="337" alt="image" src="https://github.com/user-attachments/assets/a0fa8319-543d-4006-bf91-5d56389bece7" />

## 1. Baud Generator (Serial Clock Generator)

The **Baud Generator** is responsible for generating the SPI serial clock (`SCLK`) required for communication between the SPI Master and SPI Slave. Since the system clock (`PCLK`) is much faster than the clock supported by most SPI peripherals, the Baud Generator divides the input clock to produce the desired SPI clock frequency.

In addition to generating `SCLK`, it also generates the internal timing signals required for transmitting (`MOSI`) and receiving (`MISO`) data according to the selected SPI mode.

### Functions

- Generates the SPI serial clock (`SCLK`).
- Divides the system clock (`PCLK`) to the required SPI frequency.
- Supports all four SPI modes using **CPOL** and **CPHA**.
- Generates separate timing signals for data transmission and reception.
- Synchronizes MOSI shifting and MISO sampling.

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

Case 2: CPOL!=CPHA
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

## Block diagram:
<img width="512" height="325" alt="image" src="https://github.com/user-attachments/assets/08529a51-ee57-40a4-968e-3ff37fb41eea" />

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
