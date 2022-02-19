# sky130-rtldesign
This github repository holds a report/summary of my experiences participating in the workshop "RTL design using Verilog with SKY130 Technology" under Kunal Ghosh.

## Table of Contents
* Day 1: Introduction to iverilog and Yosys
  - Preliminary Tasks
  - Simulation of 2:1 Mux using iverilog
  - Synthesis of 2:1 Mux using Yosys

* Day 2: .libs, hierarchical vs flat synthesis & flop coding
  - Understanding .lib file
  - Hierarchical and Flat synthesis
  - Optimization using Flip Flops

* Day 3
  - TBD

* Day 4
  - TBD

* Day 5
  - TBD

* Credits
* References

## Day 1
  ### Preliminary Tasks
  I'll start the report by providing readers with one-line descriptions of a few tools:
  
    1. Design: The set of verilog codes which has necessary functionality to meet with the given specs.
    2. Simulator: Tool for simulating a design to check if RTL Design matches with the given specs.
    3. Test Bench: Setup to apply stimulus (test_vectors) to the design to check its functionality.
    4. Synthesizer: Tool which converts RTL to gate-level netlist.
    
  Relevant github repositories (sky130RTLDesignAndSynthesisWorkshop, vsdflow) are first cloned<sup>a</sup> using the following command in a relevant directory:
  ```git clone https://www.github.com/(insert_file_name)```
  
  The repository "vsdflow" contains relevant tools for this workshop and "sky130RTLDesignAndSynthesisWorkshop" contains various pre-written verilog files, model files and .lib file for understanding the design flow.
  
  IMG Capture28
  
  The second part of this day involves simulation of 2:1 Mux. The following images contain verilog behavioral file and testbench of the 2:1 Mux which can be read in linux by running the following command:
  ```
  gvim tb_good_mux -o good_mux
  ```
  
  IMG Capture29
  
  IMG Capture30
  
  ### Simulation of 2:1 Mux using iverilog
  Iverilog is a simulator. It takes in a behavioural model of a design and corresponding testbench as its arguments. The testbench generates necessary stimulus in the design (which is basically changes in primary inputs over time) and paves way for observing primary output changes. Iverilog then makes sense of this and generates an executable file (.out or .vvp file) which upon execution, dumps a .vcd file (Value Change Dump). This can later be read by gtkwave to visually observe the stimulus as waveforms.
  
  This sounds complicated but actually doing this involves only a few commands in linux:
  ```
  iverilog good_mux.v tb_good_mux.v
  ./a.out
  gtkwave tb_good_mux.vcd
  ```
  
  The following images contain the execution of all the above commands and the waveforms in the gtkwave viewer.
  
  IMG Capture31
  
  IMG Capture32
  
  ### Synthesis of 2:1 Mux using Yosys
  Yosys is a popular opensource synthesis tool. Synthesis tools like Yosys specifically require the RTL design file and .lib file. The RTL Design file is remodelled using logic gates (standard cells) provided in the .lib file. The remodelled gate-level version of the design file is called a "netlist". Before delving deep into the implementation in Yosys, let's take our time to check out the gates in a .lib file.
  
  A .lib file consists of various logic gates like AND2, OR2, NOR2, NAND2, XOR2, etc. There are various versions of the same gate as well, some of which are faster and others slower! This begs the question: <ins>what's the point in having different variations?</ins> There are very specific and harsh timing requirements gate-level circuits should meet (for example, setup time and hold time is the amount of time required for the input to a Flip-Flop to be stable before and after a clock edge respectively). If not, metastability might happen. In order to avoid this scenario, both slow and fast gates are required. But then, there are other consequences as well because the speed of a circuit depends on capacitance at the load; the wider the transistors in a gate, the less delay it will have but that'd mean more area and power (the inverse is also true). This is why proper constraint files should be supplied so that the synthesizer does its task properly and efficiently.
  
  Now, as for the implementation, it is pretty straightforward. First type the following in the terminal:
  ```
  yosys
  ```
  This opens up the Yosys prompt. After this, we'll have to execute several commands in the said prompt, all of which will be mentioned below.
  
  > read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib /n
  > read_verilog good_mux.v /n
  > synth -top good_mux

  IMG Capture34

  > abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
  > show

  IMG Capture35
  IMG Capture36
  Note that good_mux is synthesized as a cell named `sky130_fd_sc_hd__mux2_1` which basically is a 2:1 Multiplexer.

  > write_verilog 


## Day 2
  ### Understanding .lib file
  
  ### Hierarchical and Flat synthesis
  
  ### Optimization using Flip Flops

## Day 3

## Day 4

## Day 5

## Credits
[a] Kunalg

## References
