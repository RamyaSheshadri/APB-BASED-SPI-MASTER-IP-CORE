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
<img width="512" height="259" alt="image" src="https://github.com/user-attachments/assets/d69373e4-bc97-414c-8fae-3e9cd1fd0ff5" />

<img width="512" height="94" alt="image" src="https://github.com/user-attachments/assets/8162a8d8-0aed-4328-aa9d-218ab09e504f" />

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
