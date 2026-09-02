# VSD_TCL_PROGRAMMING_WORKSHOP
# Overview
Boltsynth is a TCL-driven automation project developed to explore how scripting can be used to control and automate key stages of a VLSI design flow.
The project takes design information from a CSV-based configuration, processes timing constraints, prepares the required synthesis inputs, runs Yosys for RTL synthesis, and uses OpenTimer for static timing analysis.

The complete flow can be summarized as:

CSV → TCL Processing → SDC Generation → Yosys Synthesis → OpenTimer STA → QoR Extraction

The main objective was not just to execute individual EDA tools, but to understand how TCL can act as the flow-control layer connecting different stages of the design process.

<img width="869" height="596" alt="image" src="https://github.com/user-attachments/assets/944d03b5-74db-4bee-be6f-4cee7dcdc167" />


## DAY 1:  Creating a TCL command and pass .csv file from UNIX shell to tcl script

## Objective

The first stage of Boltsynth establishes the interface between the Linux shell and the TCL automation script.
The objective is to allow the user to launch the flow by supplying a CSV configuration file from the command line and to handle invalid or incomplete inputs gracefully.
The TCL script checks whether the required argument has been supplied and whether the specified CSV file is actually available.

# Input Handling

The flow considers multiple execution conditions:

### Scenario 1: No CSV file supplied
The script identifies that the required input argument is missing and displays an appropriate usage message.

<img width="834" height="491" alt="image" src="https://github.com/user-attachments/assets/fdcba800-fdaa-4450-bcf8-56fd553e6f35" />

### Scenario 2: Invalid CSV path
If the user provides a CSV filename that does not exist, the script reports the issue instead of proceeding with an invalid configuration.

<img width="803" height="384" alt="image" src="https://github.com/user-attachments/assets/f65e63fa-ce21-4ea1-addf-7e5b51b6ef1f" />

### Scenario 3: Help option
A help option is provided to explain the expected CSV format and how the flow should be launched.

<img width="816" height="418" alt="image" src="https://github.com/user-attachments/assets/978b94b9-027c-47f4-8f08-7093fdc1611b" />

## Creating CSV File
The CSV file acts as the initial design descriptor for the flow.
Instead of hard-coding design-specific paths and parameters throughout the TCL scripts, Boltsynth reads the required information from the configuration file.
This makes the automation flow easier to reuse with different designs.

<img width="876" height="361" alt="image" src="https://github.com/user-attachments/assets/910c4071-b9fd-4c1d-8a87-edb27f54ae33" />

## Running the Flow

Source the UNIX shell to tcl script by passing the csv file
```
tclsh boltsynth.tcl <design.csv>

```
<img width="948" height="679" alt="image" src="https://github.com/user-attachments/assets/bc5b60bc-5236-46c9-94ef-a0f0bbfdec74" />

## DAY 2: Design Configuration and Synthesis Setup

### Objective

The objective of Day 2 is to take the information provided in the design CSV file and prepare all the inputs required for the synthesis flow.
The TCL script extracts the design information, validates the required files and directories, processes the constraints, identifies the RTL sources, and finally prepares the synthesis script for Yosys.

## 1. Creating TCL Variables from CSV

The information provided in the design CSV file is extracted and converted into TCL variables.
These variables store important design information such as:

- Design name
- RTL/netlist directory
- Library file
- Constraint file
- Output directory
- Other required paths and parameters

Using variables allows the same TCL flow to work with different designs by changing the CSV configuration instead of modifying the script manually.

<img width="722" height="381" alt="image" src="https://github.com/user-attachments/assets/e8288e00-7dcc-4e21-a401-1a31a4ba599a" />

## 2. Checking Required Directories

After creating the variables, the TCL script checks whether the directories specified in the CSV actually exist.
For example, the script verifies the directory containing the RTL/netlist files before attempting to read the design.
This prevents the flow from continuing with an invalid directory path.

<img width="1224" height="322" alt="image" src="https://github.com/user-attachments/assets/d7eeae06-5f3e-4b75-9d1b-a527ca35b961" />

## 3. Checking Required Files

The same validation is performed for files specified in the CSV configuration. 
The script checks whether required files such as the following are available:

- Standard-cell library file
- Constraint file
- Other design input files

If a required file is missing, the flow can report the problem before proceeding further.

## 4. Converting `openMSP430_design_constraints.csv` into a Matrix Object

The constraint information is stored in a separate `openMSP430_design_constraints.csv` file.
Instead of processing the CSV as plain text line by line, the file is read and converted into a **matrix object** in TCL.
This provides a structured representation of the constraint data, where individual rows and columns can be accessed programmatically.
The matrix can then be used to extract specific constraint information required by the flow.

<img width="1204" height="699" alt="image" src="https://github.com/user-attachments/assets/53550211-c3fc-4eb9-be54-deddb75c761f" />

## 5. Computing Row Numbers Using Matrix Processing

Once the constraint CSV has been converted into a matrix, the script performs further processing to determine the required row numbers.
The matrix is searched and processed to locate specific constraint entries.  
This allows the script to identify the position of relevant information without relying on fixed row numbers.
The computed row numbers are then used to access the corresponding constraint information from the matrix.
This makes the constraint-processing stage more flexible when the CSV contents change.

<img width="539" height="147" alt="image" src="https://github.com/user-attachments/assets/96228367-cb62-4a7c-8706-d7396409f294" />

## 6. Extracting Constraint Information

Using the matrix and the computed row positions, the required constraint values are extracted from the `openMSP430_design_constraints.csv` file.
The extracted information can then be used by the subsequent stages of the TCL flow to generate or apply the required timing constraints.
This separates the constraint data from the main TCL script and makes the flow easier to configure for different designs.

## DAY 3: Processing Clock and Input Constraints

### Objective

Day 3 focuses on processing clock and input information from `openMSP430_design_constraints.csv` and generating the corresponding SDC constraints.

### 1. Identifying Clock Constraint Columns

The constraint matrix is searched to locate the columns corresponding to clock latency and slew parameters. This allows the script to dynamically identify the required data instead of depending on fixed column positions.

### 2. Generating Clock Latency Constraints

The identified clock latency values are processed and written into the SDC file using the appropriate `set_clock_latency` constraints.

### 3. Adding Clock Slew Constraints

Clock slew information is also extracted from the matrix and added to the SDC file using clock transition constraints.

### 4. Creating Clock Constraints

The clock period and duty-cycle information are used to generate the `create_clock` constraint. The waveform values are calculated from the period and duty cycle before being written to the SDC file.

### 5. Differentiating Bit and Bus Inputs

The next step is to identify whether an input port is a single bit or part of a bus. The RTL/netlist is examined using pattern matching so that constraints can be applied correctly to individual ports as well as bus signals.

<img width="1568" height="837" alt="image" src="https://github.com/user-attachments/assets/0b8bf700-57d7-49ff-b141-ac1ab7579ce1" />

<img width="1584" height="817" alt="image" src="https://github.com/user-attachments/assets/b6ae957b-f179-49c2-83fd-1266d072897f" />

### 6. Generating Input Constraints

After identifying bit and bus inputs, the required input timing constraints are generated and written into the SDC file. The process includes identifying the relevant input delay and transition values and applying them to the appropriate ports.

<img width="1567" height="837" alt="image" src="https://github.com/user-attachments/assets/7fdaf5e6-58d7-42db-af03-55a2341c2ace" />

### Day 3 Flow

`Constraint Matrix` → `Clock Column Search` → `Clock Latency/Slew` → `Clock Period & Duty Cycle` → `Bit/Bus Identification` → `Input Constraints` → `SDC File`

## DAY 4: Output Constraints and Yosys Synthesis

### Objective

Day 4 completes the SDC generation and connects the generated constraints and RTL design to Yosys for synthesis.

### 1. Generating Output Constraints

The output-port information is processed and the required `set_output_delay` and `set_load` constraints are generated in the SDC file.

### 2. Passing SDC and RTL to Yosys

The generated SDC information, RTL files, and standard-cell library are prepared as inputs for the Yosys synthesis flow.

<img width="1506" height="804" alt="image" src="https://github.com/user-attachments/assets/f0f7fcb8-5fe7-4ab0-a57d-bef0f13af250" />

### 3. Memory Synthesis with Yosys

A memory design is used to understand how Yosys processes RTL and converts it into a synthesized gate-level representation. The synthesized design is examined to understand its components and gate-level structure.

<img width="1501" height="825" alt="image" src="https://github.com/user-attachments/assets/afd2241d-5232-4f90-8957-80780747b772" />

### 4. Understanding the Synthesized Netlist

The generated gate-level netlist is inspected to understand how the original memory/RTL structure is represented using synthesized cells and logic components.

<img width="314" height="852" alt="image" src="https://github.com/user-attachments/assets/3e5a08f6-d37b-4294-857e-11e34216d6d7" />

> **Take:** Gate-level netlist or Yosys output showing the synthesized components.

### 5. Hierarchy Check

A TCL/Yosys script is used to verify that the referenced modules are properly connected and that the design hierarchy is valid. A successful check confirms that the required modules are present and correctly linked.

<img width="1561" height="312" alt="image" src="https://github.com/user-attachments/assets/cb3237bb-c804-4d5b-a0da-ca8ac4cc42fd" />

### 6. Hierarchy Error Handling

Error handling is added to detect hierarchy failures and prevent the flow from silently continuing with an invalid design. The error status is checked and useful debugging information is reported when a hierarchy problem occurs.

<img width="1588" height="267" alt="image" src="https://github.com/user-attachments/assets/48a9e541-36c8-46d7-af9b-5d9c865fc9a4" />

### Day 4 Flow

`Output Constraints` → `SDC Completion` → `RTL + Library + SDC` → `Yosys` → `Memory Synthesis` → `Hierarchy Check` → `Error Handling`

## DAY 5: OpenTimer Integration and QoR Analysis

### Objective

Day 5 connects the synthesized netlist and processed SDC constraints to OpenTimer, performs STA, and generates a concise QoR report. 

### 1. Synthesis Script and Netlist Processing

The main synthesis script is created and the Yosys-generated netlist is cleaned and modified so it can be used by OpenTimer.

<img width="1431" height="314" alt="image" src="https://github.com/user-attachments/assets/a3e5ac1a-994e-4403-b523-c7771482fcca" />

### 2. Using TCL Procs

Reusable TCL `proc` commands are created for handling library paths, netlist paths, and SDC processing and also converting all bussed constraints to bit-blasted.

<img width="732" height="843" alt="image" src="https://github.com/user-attachments/assets/9621930e-52d3-407f-a47d-08d6bec32166" />

### 3. Processing the SDC File

The SDC file is read and special characters such as square brackets are replaced to make the constraints easier to process. The clock period, clock port, and duty cycle are then extracted and converted into OpenTimer-compatible clock constraints.

<img width="1364" height="873" alt="image" src="https://github.com/user-attachments/assets/28421586-b3bf-4cec-aa8f-3f4ab52c6d53" />

### 4. Converting SDC to OpenTimer Format

The required clock, input, and output constraints are converted from SDC syntax into the format expected by OpenTimer. Bussed input constraints are also expanded into bit-level constraints where required.

### 5. OpenTimer Configuration

An OpenTimer `.conf` file is generated with the required library, netlist, timing constraints, and analysis settings.

<img width="1132" height="326" alt="image" src="https://github.com/user-attachments/assets/94eb37ff-f7a3-4f63-b6a5-6a578fbe3fc7" />

<img width="1578" height="863" alt="image" src="https://github.com/user-attachments/assets/568c33d8-14f2-45ab-85f9-0d4c8cf147bf" />

<img width="808" height="401" alt="image" src="https://github.com/user-attachments/assets/b16c3430-0f19-43d0-83fe-3b7e63b708dd" />

<img width="1290" height="845" alt="image" src="https://github.com/user-attachments/assets/b0b91536-d5e9-4e57-98f5-f55cbb21d10f" />



### 6. Running STA and Extracting QoR

OpenTimer is executed and the generated results are parsed to obtain key QoR parameters, including:

- STA runtime
- Instance count
- WNS
- FEP
- Setup violations
- Hold violations
- Reg-to-out violations

### 7. Report Formatting

The extracted timing and synthesis information is formatted into a concise QoR report for easy interpretation.

<img width="1336" height="649" alt="image" src="https://github.com/user-attachments/assets/8359429d-04fe-4ed3-93b7-747681cf6729" />

### Day 5 Flow

`Yosys Netlist` → `Netlist Processing` → `TCL Procs` → `SDC Processing` → `OpenTimer Format` → `.conf` → `STA` → `QoR Report`

### Conclusion

By the end of Day 5, the complete **CSV → SDC → Yosys → OpenTimer → QoR** flow is automated, with TCL handling data processing, tool execution, STA extraction, and final report generation. 
