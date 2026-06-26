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
