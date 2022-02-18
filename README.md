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
  This day starts out with learning some descriptions like:
  
    1. Design: The set of verilog codes which has necessary functionality to meet with the given specs.
    2. Simulator: Tool for simulating a design to check if RTL Design matches with the given specs.
    3. Test Bench: Setup to apply stimulus (test_vectors) to the design to check its functionality.
    
  Relevant github repositories (sky130RTLDesignAndSynthesisWorkshop, vsdflow) are first cloned<sup>a</sup> using the following command in a relevant directory:
  > ```git clone https://www.github.com/(insert_file_name)```
  
  The repository "vsdflow" contains relevant tools for this workshop and "sky130RTLDesignAndSynthesisWorkshop" contains various pre-written verilog files, model files and .lib file for understanding the design flow.
  
  IMG Capture28
  
  The second part of this day involves simulation of 2:1 Mux. The following images contain verilog behavioral file and testbench of the 2:1 Mux which can be read in linux by running the following command:
  > ```gvim tb_good_mux -o good_mux```
  
  IMG Capture29
  
  IMG Capture30
  
  ### Simulation of 2:1 Mux using iverilog
  Iverilog is a simulator. It takes in a behavioural model of a design and corresponding testbench as its arguments. The testbench generates necessary stimulus in the design (which is basically changes in primary inputs over time) and paves way for observing primary output changes. Iverilog then makes sense of this and generates an executable file (.out or .vvp file) which upon execution, dumps a .vcd file (Value Change Dump). This can later be read by gtkwave to visually observe the stimulus as waveforms.
  
  This sounds complicated but actually doing this involves only a few commands in linux:
  > 
  ```
  iverilog good_mux.v tb_good_mux.v
  ./a.out
  gtkwave tb_good_mux.vcd
  ```
  
  The following images contain the execution of all the above commands and the waveforms in the gtkwave viewer.
  
  IMG Capture31
  
  IMG Capture32
  
  ### Synthesis of 2:1 Mux using Yosys
  Yosys is a popular opensource synthesis tool.

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
