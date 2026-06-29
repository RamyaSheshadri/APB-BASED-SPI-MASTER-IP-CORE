# APB-BASED-SPI-MASTER-IP-CORE
**Author:** Ramya Sheshadri

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

The **APB Slave Interface** acts as the communication bridge between the **CPU** and the **SPI Master**. It is responsible for receiving configuration commands and data from the processor through the APB bus, storing them in internal registers, and distributing the required control signals to the SPI hardware blocks.

The CPU never communicates directly with the Baud Generator, Shift Register, or SPI Control Logic. Instead, every SPI operation begins with an APB transaction, making the APB Slave Interface the central control block of the design.

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
