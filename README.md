# AMBA-APB-Protocol
This Repository describes the AMBA APB protocol with its flow diagram and operating with cycle graphs.

# Table of Contents
1. [APB Protocol](#apb-protocol)  
2. [Signal Description](#signal-description)  
3. [State Diagram](#state-diagram)  
4. [Real Time Scenario](#real-time-scenario)  
5. [RTL Design Code](#rtl-design-code)  
6. [Testbench Code](#testbench-code)  
7. [Design.v Explained](#designv-explained)  
8. [Testbench.v Explained](#testbenchv-explained)  
9. [Signal Setup and DUT Instantiation](#signal-setup-and-dut-instantiation)  
10. [Simulation Waveform using EPWave](#simulation-waveform-using-epwave)  
11. [EPWave Key Observations](#epwave-key-observations)  
12. [Use Cases of APB Protocol](#use-cases-of-apb-protocol)  
13. [Credits](#credits)  


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

## Signal Description - 

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


## RTL CODE

- **TESTBENCH.v :**
    
  ```bash
  module apb_tb;
    reg PCLK;           // Clock signal
    reg PRESETn;        // Active-low reset signal
    reg PSELx;          // Chip select signal
    reg PENABLE;        // Enable signal
    reg PWRITE;         // Write enable signal
    reg [31:0] PADDR;   // Address for memory access
    reg [31:0] PWDATA;  // Data to be written into memory
    wire [31:0] PRDATA; // Data read from memory
    wire PREADY;        // Ready signal indicating when the module is ready to accept or provide data

    // Instantiate the DUT (Design Under Test)
    apb_slave dut (
        .PCLK(PCLK),        // Connects PCLK to DUT
        .PRESETn(PRESETn),  // Connects PRESETn to DUT
        .PSELx(PSELx),      // Connects PSELx to DUT
        .PENABLE(PENABLE),  // Connects PENABLE to DUT
        .PWRITE(PWRITE),    // Connects PWRITE to DUT
        .PADDR(PADDR),      // Connects PADDR to DUT
        .PWDATA(PWDATA),    // Connects PWDATA to DUT
        .PRDATA(PRDATA),    // Connects PRDATA from DUT
        .PREADY(PREADY)     // Connects PREADY from DUT
    );

    initial begin
        // Initializing signals
        PCLK = 0;
        PRESETn = 0;
        PSELx = 0;
        PENABLE = 0;
        PWRITE = 0;
        PADDR = 32'd0;
        PWDATA = 32'd0;
        $dumpfile("apb_tb.vcd");
        $dumpvars(0, apb_tb);
        #5;
        PRESETn = 1;
    
        // Write cycle
        #5;
        PSELx = 1;
        PENABLE = 1;
        PWRITE = 1;
        PADDR = 32'd0;
        PWDATA = 32'h00001234;

        #10;
        PENABLE = 0;
        PSELx = 0;
        PWRITE = 0;

        // Read cycle
        #5;
        PSELx = 1;
        PENABLE = 1;
        PWRITE = 0;
        PADDR = 32'd0;

        #10;
        PENABLE = 0;
        PSELx = 0;

        #10;
        $finish;
    end

    // Clock generation
    always #5 PCLK = ~PCLK;

    initial begin
        $monitor("Time: %0t | CLK: %b | RESETn: %b | PADDR: %h | PWDATA: %h | PWRITE: %b | PSELx: %b | PENABLE: %b | PRDATA: %h | PREADY: %b",
                 $time, PCLK, PRESETn, PADDR, PWDATA, PWRITE, PSELx, PENABLE, PRDATA, PREADY);
        $dumpfile("apb_tb.vcd");
        $dumpvars(0, apb_tb);
    end

    initial begin
        $display("Starting APB Testbench Simulation");
        $display("=================================");
    end
  endmodule
  ```

- Let's understand the testbench bit-by-bit with reference to the signals description and their corresponding operations.
- So, the goal of this testbench is to verify the basic functionality of an APB slave, particularly focusing on a single write transaction followed by a single read transaction. This is a fundamental setup that helps us understand how the APB protocol behaves in a non-pipelined fashion, where each transaction completes fully before the next one begins.

- We begin by declaring all the standard APB interface signals:

- **`PCLK`**: The clock signal driving the interface.
- **`PRESETn`**: An active-low reset that initializes the system.
- **`PSELx`**: This is the select signal, indicating whether the current slave is targeted by the master.
- **`PENABLE`**: This indicates the second (enable) phase of the APB transaction.
- **`PWRITE`**: This determines the direction — high for write, low for read.
- **`PADDR`**: The 32-bit address bus used to point to memory/register location.
- **`PWDATA`**: The 32-bit data bus used to send data to the slave (during write).
- **`PRDATA`**: The data output from the slave (during read).
- **`PREADY`**: Indicates whether the slave is ready to complete the current transaction.

- Then, We instantiate the DUT (`apb_slave`) and wire up these signals appropriately.  

### I. Initialization Phase

In the `initial` block, we first:

- Drive all the inputs to their default zero state.
- Assert `PRESETn = 0` to hold the design in reset.
- Generate a waveform dump via `$dumpfile` and `$dumpvars` for post-simulation analysis using GTKWave or EPWave.
- After `#5` time units, we deassert the reset (`PRESETn = 1`), allowing the design to begin normal operation.

### II. Read Transaction

Now, we read back from the same address to verify that the write was successful.

#### a. Setup Phase

- Set `PSELx = 1` to select the slave.
- Set `PWRITE = 0` to indicate a **read**.
- Set `PADDR = 32'd0` to read from the same location used in the write.

#### b. Enable Phase

- After a short delay, assert `PENABLE = 1`.
- The slave now places the value stored at `memory[PADDR]` onto `PRDATA`.

#### c. Transaction Completion

- Wait a few time units to simulate bus latency.
- Deassert `PSELx` and `PENABLE`.

### III. Write Transaction

Next, we perform a **write operation** following the standard APB protocol phases.

#### a. Setup Phase

- Set `PSELx = 1` to select the slave.
- Set `PWRITE = 1` to indicate it's a write.
- Provide a valid address: `PADDR = 32'd0`.
- Provide the write data: `PWDATA = 32'h00001234`.

#### b. Enable Phase (After a Clock Edge)

- After a small delay, assert `PENABLE = 1`.
- Now, `PSELx = 1`, `PENABLE = 1`, and `PWRITE = 1` — this initiates the **write transfer**.
- Internally, the slave executes: `memory[PADDR] = PWDATA`.

#### c. Transaction Completion

- After the write has taken place, deassert all control signals:
  - `PSELx = 0`
  - `PENABLE = 0`
  - `PWRITE = 0`

This sequence mimics how a real APB master sets up, enables, and completes a transaction.

So, As all the signals behaved correctly, `PRDATA` will now contain `0x00001234` — confirming the APB slave's read/write functionality.

- **DESIGN.v :**

```bash
  // DESIGN

module apb_slave (
    input wire PCLK,          // Clock signal
    input wire PRESETn,       // Active-low reset signal
    input wire PSELx,         // Chip select signal
    input wire PENABLE,       // Enable signal
    input wire PWRITE,        // Write enable signal
    input wire [31:0] PADDR,  // Address for memory access
    input wire [31:0] PWDATA, // Data to be written into memory
    output reg [31:0] PRDATA, // Data read from memory
    output reg PREADY         // Ready signal indicating when the module is ready to accept or provide data
);

  reg [31:0] memory [0:255];  // An array memory of 256 registers (each 32 bits wide) is used to simulate memory storage.

    // READ and WRITE OPERATIONS : - 
    always @(posedge PCLK or negedge PRESETn) begin
        if (!PRESETn) begin
          
            // Reset state: Clear outputs and set ready signal
            PRDATA <= 32'd0;
            PREADY <= 1'b1;
        end else begin
            // 1. Normal operation
            if (PSELx && PENABLE) begin
                if (PWRITE) begin
                    // 2. WRITE OPERATION : Store PWDATA into memory at address PADDR
                    memory[PADDR] <= PWDATA;
                    PREADY <= 1'b1;
                end else begin
                    // 3. READ OPERATION : Load data from memory at address PADDR into PRDATA
                    PRDATA <= memory[PADDR];
                    PREADY <= 1'b1;
                end
            end else begin
              // 4. IDLE STATE ( when not selected ) : Set ready signal high
                PREADY <= 1'b1;
            end
        end
    end
endmodule
  ```

- The `apb_slave` module implements a simple memory-mapped slave interface that complies with the **AMBA APB (Advanced Peripheral Bus)** protocol. It supports basic **read** and **write** operations to a simulated memory array using standard APB signals.

### Inputs
- `PCLK` — Clock signal
- `PRESETn` — Active-low reset
- `PSELx` — Slave select
- `PENABLE` — Transfer phase enable
- `PWRITE` — Indicates write (`1`) or read (`0`)
- `PADDR` — Address bus
- `PWDATA` — Data to be written

### Outputs
- `PRDATA` — Read data output
- `PREADY` — Ready signal (indicates transaction completion)

- Internally, a **256 × 32-bit memory array** is declared to act as a simple register space or RAM, The behavior is synchronous to the **rising edge of the clock** and includes **asynchronous reset support**.

- During **reset**, the output data and ready signals are cleared.
- When both `PSELx` and `PENABLE` are asserted:
  - If `PWRITE = 1`, the module writes `PWDATA` into `memory[PADDR]`.
  - If `PWRITE = 0`, it reads `memory[PADDR]` into `PRDATA`.
- In both cases, `PREADY` is asserted high to indicate the transaction is complete.
- If the slave is **not selected or not enabled**, the module stays idle but still drives `PREADY = 1`, making it a **zero-wait-state peripheral**.

This minimal design reflects the **fundamental transaction phases** of the APB protocol — `Setup` and `Enable` — and models a **compliant slave peripheral**, useful for:
- Design Verification (DV)
- Extension into more complex peripherals like GPIOs or timers

We use an `always` block that triggers on the **rising edge of the clock** or the **falling edge of reset**.

### I. RESET PHASE

If reset is active (`PRESETn = 0`), then we initialize:
- `PRDATA` to `0`
- `PREADY` to `1` (saying, “Hey I'm ready after reset”)

### II. NORMAL OPERATION

If `PSELx` is high (we're selected) and `PENABLE` is high (enabled to do work), then we check:

### III. WRITE Operation (`PWRITE = 1`)
- Take the data from `PWDATA` and store it in `memory[PADDR]`
- Raise `PREADY` to say “Write done”

### IV. READ Operation (`PWRITE = 0`)
- Fetch data from `memory[PADDR]` and put it in `PRDATA` to send back
- Set `PREADY = 1` to say “Read done”

### V. IDLE PHASE (Not Selected)

- Even if nothing's happening, we still keep `PREADY = 1` so the bus doesn’t hang.

###  Port Descriptions

| Port      | Direction | Width   | Description                                                                 |
|-----------|-----------|---------|-----------------------------------------------------------------------------|
| `PCLK`    | Input     | 1-bit   | System clock signal. All operations are synchronized on its rising edge.    |
| `PRESETn` | Input     | 1-bit   | Active-low reset. Resets internal state and outputs.                        |
| `PSELx`   | Input     | 1-bit   | Slave select signal. Indicates that the slave is being addressed.           |
| `PENABLE` | Input     | 1-bit   | Enable phase signal. Marks the second phase of the APB protocol.            |
| `PWRITE`  | Input     | 1-bit   | Determines transfer direction: `1` for write, `0` for read.                 |
| `PADDR`   | Input     | 32-bit  | Address for memory access.                                                  |
| `PWDATA`  | Input     | 32-bit  | Data to be written to memory.                                               |
| `PRDATA`  | Output    | 32-bit  | Data read from memory.                                                      |
| `PREADY`  | Output    | 1-bit   | Indicates readiness to complete the transfer.        

## RTL SIMULATION 

- EDA Playground (tb.v & design.v)

  <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/7df97200-629a-42bc-82d0-34a11843b165" />

- Output Log

  <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/ac494670-a098-46e5-9ecc-97131cc13caa" />

- EPWave

  <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/8e4fb0a5-892e-427b-aefd-8d6222b992e9" />

  ### From the Above EPWave, We observe

- **Reset Phase:** `PRESETn` is low at the start, initializing the slave.
- **Write Phase:**
  - `PSELx`, `PENABLE`, and `PWRITE` are high.
  - `PWDATA = 0x660` is written at address `PADDR = 0`.
- **Read Phase:**
  - `PWRITE = 0` while `PSELx` and `PENABLE` stay high.
  - `PRDATA` outputs the previously written `0x660` showing data integrity.
- **Memory Verification:**
  - The `memory[0]` location stores the value `0x660`.
- **PREADY** remains high, indicating the slave is always ready for transactions.

- **This waveform confirms that the APB slave correctly handles basic read/write operations according to the AMBA APB protocol specification.**
  
