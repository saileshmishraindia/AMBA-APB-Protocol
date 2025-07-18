# AMBA-APB-Protocol
This Repository describes the AMBA APB protocol with its flow diagram and operating with cycle graphs.

# Table of Contents

1. [APB Protocol](#1-apb-protocol)  
2. [Signal Description](#2-signal-description)  
3. [State Diagram](#3-state-diagram)  
4. [Real Time Scenario](#4-real-time-scenario)

## APB PROTOCOL

The **Advanced Peripheral Bus (APB)** is a **low-cost interface** that is part of the **AMBA (Advanced Microcontroller Bus Architecture)** family. It is **optimized for minimal power consumption** and **reduced interface complexity**, making it suitable for connecting **low-bandwidth peripheral devices** such as UARTs, GPIOs, timers, and sensors.

### Key Characteristics:
- **Simple, Synchronous Protocol**  
- **Non-pipelined**: Every transfer takes **at least two clock cycles**
- **Read and Write Operations are separated**
- **Handshake-based communication**
- **Typically 32-bit wide data buses**

### System Use & Architecture:

- The APB is **not intended for high-performance burst transfers**.
- It is mainly used to **access control/status registers** of peripheral devices.
- APB peripherals are usually connected to **main memory systems via a bridge**, such as an **AXI-to-APB bridge**.
- This bridge acts as a master, called a **Requester**, while the peripheral is called a **Completer**.

### Roles:
- **Requester**: Initiates the APB transfer (usually the APB bridge)
- **Completer**: Responds to the request (typically the peripheral)

### Purpose:
- **Connects low-bandwidth peripherals** to high-performance cores or memory subsystems
- **Simplifies peripheral design** through its straightforward protocol
- **Reduces dynamic power usage**, ideal for SoCs and embedded systems


 <img width="1095" height="362" alt="image" src="https://github.com/user-attachments/assets/bd65dac6-79f5-4793-96e9-62788c332d52" />

- This block diagram shows a typical ARM-based System-on-Chip (SoC) interconnect structure. Let's Understand the APB Bridge in details.
- So, It's quite understandable from the above block diagram that the APB (Advanced Peripheral Bus) Bridge is connected to a high-speed bus (like AHB or ASB). In a Typical SoC (Silicon on Chip or System on Chip) Architecture, the AHB/ASB (Advanced High-performance Bus or Advanced System Bus) is used for high-speed, high-performance components such as the CPU, on-chip RAM, and DMA controllers. These components require fast and efficient data transfers. However, many peripheral devices like UART, Timers, Keypads, and GPIO/PIO do not demand such high bandwidth. These peripherals are typically low-power and operate at lower speeds. For this reason, the APB (Advanced Peripheral Bus) is used, as it is optimized for simplicity, low power consumption, and minimal bandwidth requirements. Since AHB/ASB and APB operate using different protocols and timings, an APB Bridge is introduced. This bridge acts as a protocol converter or translator, enabling seamless communication between the high-speed AHB/ASB bus and the low-speed APB peripherals.

- So the APB Bridge plays a crucial role in connecting high-performance components with low-speed peripherals in an SoC. It performs protocol conversion, translating AHB/ASB transactions into APB-compatible ones, enabling seamless communication across different bus systems. It also handles clock domain adaptation, allowing the main high-speed system bus to interface reliably with peripherals operating at lower clock speeds. By isolating peripheral logic from the complexity of AHB/ASB, the bridge simplifies peripheral design—peripherals only need to implement the lightweight APB protocol. Additionally, the APB architecture is optimized for low power consumption thanks to its lack of pipelining, straightforward read/write mechanisms, and minimal control signaling. Lastly, APB’s simplicity leads to area efficiency, reducing the silicon footprint required for each peripheral block.

- To understand the role of the APB Bridge in ARM-based SoCs, we can draw a clear analogy from how the 8085 microprocessor handles different operation cycles. In the 8085, when an instruction like LDA 2000H is executed, the processor performs multiple memory read cycles to fetch the 8-bit opcode and the two address bytes from memory. These operations are fast and frequent, much like how an AHB/ASB bus handles high-speed communication between the CPU and RAM in modern systems. Now consider a slower operation like OUT 01H, which sends an 8-bit data value, say 11001010, to an I/O device such as a display or UART. Here, the 8085 enters a slower I/O write cycle, controlled using signals like IO/M̅, WR̅, and the address 01H. This kind of peripheral communication is simple, low-speed, and timing-sensitive — just like how peripherals (GPIO, Timer, UART) operate on the APB (Advanced Peripheral Bus) in ARM-based systems.

  <img width="860" height="683" alt="image" src="https://github.com/user-attachments/assets/23a38fdf-d629-4fce-8323-f8e785df1840" />

  For better Understanding and feeling like a actual electronic surgeon, I provided this Architecture of 8085 Microprocessor

- In this analogy, the APB Bridge plays the same role as the 8085’s control and timing logic that switches the bus behavior from high-speed memory operations to slower I/O cycles. It adapts fast bus protocols to interface with slow peripherals, converting the fast AHB/ASB transactions into lightweight APB transactions — allowing something like that 11001010 data to reach a low-power peripheral correctly and efficiently.

### Benefits:

- **1. Low power consumption** due to non-pipelined nature. What is the concept of Pipelining ? In a pipelined system, multiple operations occur in overlapping stages to improve efficiency. A common analogy is a car wash where different cars are in different stages simultaneously—Car 1 is being washed, Car 2 is being rinsed, and Car 3 is being dried. This overlapping of tasks increases throughput and is similar to how high-speed buses like AHB or AXI operate. These buses can start preparing the next transaction even before the current one is finished, allowing data to flow continuously through the system. On the other hand, APB (Advanced Peripheral Bus) is non-pipelined, meaning each transaction is completed fully before the next one begins. This can be compared to a manual ticket counter, where a person waits in line, the clerk handles them completely—asks their name, writes it, hands the ticket—and only then serves the next person. There is no overlap, making the process simpler but inherently slower. For example, if a system-on-chip needs to send two 8-bit values—say 11001010 to a UART and 00010110 to a GPIO—the APB bus handles this sequentially. It waits for the UART to fully acknowledge the first transaction before beginning the write to the GPIO. This behavior ensures that address, data, and response phases do not overlap. While this makes APB slower compared to pipelined buses, it also reduces the complexity of the peripheral hardware—no need for buffering, arbitration, or pipelined control—making it ideal for simple, low-power devices.

- **2. Reduced hardware complexity** - Now, Since APB is a non-pipelined protocol and only handles one operation at a time, the connected peripheral doesn’t have to worry about multiple requests coming in at once. This means it doesn’t need extra hardware like data buffers, complex timing logic, or multiple registers to manage overlapping signals. For example, if the CPU wants to send data to a UART using APB, the UART just waits for the signal, accepts the data, and responds—then it's ready for the next request. It’s like a shop with only one customer at a time: the shopkeeper finishes with one person before calling the next, so the process is calm and simple. This kind of design reduces the amount of logic needed inside the peripheral, which saves chip area, uses less power, and is easier to design and test.
  
- **3. Easier peripheral integration** - Using APB makes it much easier to add or connect new peripherals like UART, timers, GPIO, or SPI to a system. Why? Because APB follows a very simple and consistent protocol—each transaction happens in a clear step-by-step manner, with no overlapping or advanced timing requirements. This means that when you design a new peripheral, you only need to handle basic control signals like PSEL, PWRITE, PENABLE, and PREADY. You don’t need to build support for pipelining, bursts, or complex arbitration like you would with faster buses. It’s like plugging in a simple USB keyboard to your computer—it just works, without needing to understand how the entire motherboard is functioning. So for designers, APB makes it much faster and less error-prone to connect new blocks to the SoC bus, which is especially helpful when working with many small peripherals. You don’t have to design anything fancy—just follow the simple APB handshake rules, and your peripheral can talk to the CPU.
  
- **4. Cost-effective** compared to AMBA protocols like AHB/AXI because it requires much less hardware to implement. Since APB is non-pipelined, does not support burst transfers, and follows a very basic protocol, it doesn’t need advanced features like data buffering, arbitration, or complex timing control. This translates to fewer logic gates, less silicon area, and lower power consumption, all of which directly reduce the cost of manufacturing the chip. For example, if you're designing an SoC with 20 small peripherals (like GPIOs, timers, or UARTs), using AHB or AXI for each one would add unnecessary complexity and hardware overhead. With APB, you can connect all these low-speed devices using minimal resources. It’s like choosing to build a small single-lane road instead of a six-lane highway when you know only a few bicycles will use it—APB gives you exactly what you need without overspending on performance you don’t require.

### Summary:
| Feature            | Description                                                                 |
|--------------------|-----------------------------------------------------------------------------|
| Type               | Synchronous, non-pipelined bus                                               |
| Typical Use        | Control and status register access                                           |
| Clock Cycles       | Minimum two per transfer (Setup + Enable)                                   |
| Integration        | Connected via bridge to high-performance buses (e.g., AXI → APB bridge)     |

## SIGNAL DESCRIPTION - 

### 1. Basic Signals

| Signal    | Source         | Width         | Description |
|-----------|----------------|---------------|-------------|
| **PCLK**   | Clock           | 1             | Clock signal. All APB signals are timed with the rising edge of `PCLK`. |
| **PRESETn**| System reset    | 1             | Active-low reset signal. Normally connected to the system bus reset. |
| **PADDR**  | Requester       | ADDR_WIDTH    | APB address bus. Can be up to 32 bits wide. |
| **PPROT**  | Requester       | 3             | Indicates protection level (normal, privileged, secure) and access type (data/instruction). |
| **PSELx**  | Requester       | 1             | Select signal. Identifies the target slave and requests a transfer. |
| **PENABLE**| Requester       | 1             | Enables the second and subsequent cycles of an APB transfer. |
| **PWRITE** | Requester       | 1             | Indicates direction: `HIGH` = write, `LOW` = read. |
| **PWDATA** | Requester       | DATA_WIDTH    | Write data bus, driven during write cycles. Can be 8, 16, or 32 bits. |
| **PSTRB**  | Requester       | DATA_WIDTH/8  | Write strobe signal, one bit per byte to indicate active byte lanes. |

### 2. Completer Signals

| Signal    | Source     | Width        | Description |
|-----------|------------|--------------|-------------|
| **PREADY**  | Completer   | 1            | Indicates the slave is ready. Used to extend APB transfers. |
| **PRDATA**  | Completer   | DATA_WIDTH   | Read data bus, driven during read cycles when `PWRITE` is LOW. |
| **PSLVERR** | Completer   | 1            | Optional signal for indicating an error (`HIGH`) on a transfer. |

### 3. Optional / User-Defined Signals

| Signal    | Source     | Width             | Description |
|-----------|------------|-------------------|-------------|
| **PWAKEUP** | Requester   | 1                 | Wake-up indicator showing activity on the APB interface. |
| **PAUSER**  | Requester   | USER_REQ_WIDTH    | User request attribute. Recommended max width: 128 bits. |
| **PWUSER**  | Requester   | USER_DATA_WIDTH   | User-defined write attribute. Recommended width: DATA_WIDTH/2. |
| **PRUSER**  | Completer   | USER_DATA_WIDTH   | User-defined read attribute. Recommended width: DATA_WIDTH/2. |
| **PBUSER**  | Completer   | USER_RESP_WIDTH   | User-defined response attribute. Recommended max width: 16 bits. |

## STATE DIAGRAM

### Here is the State Diagram of APB Protocol, Let's understand it bit-by-bit . . .

 <img width="1004" height="443" alt="image" src="https://github.com/user-attachments/assets/c557aeed-f8b4-4df5-b28a-c5ab214aa874" />

- The APB Protocol transitions through **two main states** for every transfer:

### 1. Setup Phase
- `PSEL` is asserted  
- `PENABLE` is low  
- Control signals and address are valid

### 2. Enable Phase
- `PENABLE` is asserted  
- Actual data transfer occurs  
- `PREADY` signals when the transfer is complete

## REAL-TIME SCENARIO

This section explains how to toggle an LED connected to a GPIO peripheral using the AMBA APB protocol. The microcontroller communicates with the GPIO peripheral through APB-compliant signals like `PADDR`, `PWRITE`, `PWDATA`, and `PSELx`.

## System Setup

| Signal     | Role                                                                 |
|------------|----------------------------------------------------------------------|
| `PADDR`    | Address of GPIO register (e.g., `0x0004` for LED control)            |
| `PWRITE`   | `1` for write, `0` for read                                           |
| `PWDATA`   | Data written to GPIO                                                  |
| `PRDATA`   | Data read from GPIO                                                   |
| `PSELx`    | Selects the GPIO peripheral                                            |
| `PENABLE`  | Moves the transfer to the ACCESS phase                                |
| `PREADY`   | Indicates when the peripheral has completed the operation             |

## Operation 1: Writing to GPIO (Turn ON the LED)

### I. IDLE State
- `PSELx = 0`: No peripheral is selected.
- The processor decides to write `0x01` to turn on the LED at address `0x0004`.

### II. SETUP State
- `PSELx = 1`: GPIO peripheral is selected.
- `PADDR = 0x0004`: Targeting the GPIO control register.
- `PWRITE = 1`: Write operation.
- `PWDATA = 0x01`: Command to turn ON the LED.
- `PENABLE = 0`: Setup phase is still in progress.

### III. ACCESS State
- `PENABLE = 1`: Transition to ACCESS phase.
- The GPIO receives `PWDATA = 0x01` and turns ON the LED.
- `PREADY = 1`: GPIO peripheral indicates the write is complete.

## Operation 2: Reading GPIO Status (Check if LED is ON)

### I. IDLE to SETUP
- Processor initiates read: `PSELx = 1`, `PADDR = 0x0004`.
- `PWRITE = 0`: Read operation.
- `PENABLE = 0`: Setup phase begins.

### II. SETUP to ACCESS
- `PENABLE = 1`: Enters ACCESS state.
- GPIO responds with `PRDATA = 0x01`, confirming LED is ON.
- `PREADY = 1`: Indicates read data is valid.

### III. Return to IDLE
- `PSELx = 0`: Transfer complete, back to IDLE state.
