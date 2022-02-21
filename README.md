# sky130-rtldesign
This github repository holds a report/summary of my experiences participating in the workshop "RTL design using Verilog with SKY130 Technology" under Kunal Ghosh.

## Table of Contents
* [Day 1: Introduction to iverilog and Yosys](https://github.com/harishMadhavan1010/sky130-rtldesign#day-1)
  - [Preliminary Tasks](https://github.com/harishMadhavan1010/sky130-rtldesign#preliminary-tasks)
  - [Simulation of 2:1 Mux using iverilog](https://github.com/harishMadhavan1010/sky130-rtldesign#simulation-of-21-mux-using-iverilog)
  - [Synthesis of 2:1 Mux using Yosys](https://github.com/harishMadhavan1010/sky130-rtldesign#synthesis-of-21-mux-using-yosys)

* [Day 2: .libs, hierarchical vs flat synthesis & flop coding](https://github.com/harishMadhavan1010/sky130-rtldesign#day-2)
  - [Understanding .lib file](https://github.com/harishMadhavan1010/sky130-rtldesign#understanding-lib-file)
  - [Hierarchical and Flat synthesis](https://github.com/harishMadhavan1010/sky130-rtldesign#hierarchical-and-flat-synthesis)
  - [Optimization using Flip Flops](https://github.com/harishMadhavan1010/sky130-rtldesign#optimization-using-flip-flops)

* [Day 3: Combinational and Sequential Optimizations](https://github.com/harishMadhavan1010/sky130-rtldesign#day-3)
  - [Overview](https://github.com/harishMadhavan1010/sky130-rtldesign#overview)
  - [Combinational Logic Optimizations](https://github.com/harishMadhavan1010/sky130-rtldesign#combinational-logic-optimizations)
  - [Sequential Logic Optimizations](https://github.com/harishMadhavan1010/sky130-rtldesign#sequential-logic-optimizations)

* [Day 4: Gate-level Simulations](https://github.com/harishMadhavan1010/sky130-rtldesign#day-4)
  - [GLS and Synth-Sim Mismatch](https://github.com/harishMadhavan1010/sky130-rtldesign#gls-and-synth-sim-mismatch)
  - [Missing Sensitivity List](https://github.com/harishMadhavan1010/sky130-rtldesign#missing-sensitivity-list)
  - [Blocking assignments](https://github.com/harishMadhavan1010/sky130-rtldesign#blocking-assignments)

* [Day 5: If, Case, For, Generate](https://github.com/harishMadhavan1010/sky130-rtldesign#day-5)
  - [If statements](https://github.com/harishMadhavan1010/sky130-rtldesign#if-statements)
  - [Case statements](https://github.com/harishMadhavan1010/sky130-rtldesign#case-statements)
  - [For statements](https://github.com/harishMadhavan1010/sky130-rtldesign#for-statements)
  - [Generate statements](https://github.com/harishMadhavan1010/sky130-rtldesign#generate-statements)

* [Credits](https://github.com/harishMadhavan1010/sky130-rtldesign#credits)
* [References](https://github.com/harishMadhavan1010/sky130-rtldesign#references)

## Day 1
  ### Preliminary Tasks
  I'll start the report by providing readers with one-line descriptions of a few tools:
  
    1. Design: The set of verilog codes which has necessary functionality to meet with the given specs.
    2. Simulator: Tool for simulating a design to check if RTL Design matches with the given specs.
    3. Test Bench: Setup to apply stimulus (test_vectors) to the design to check its functionality.
    4. Synthesizer: Tool which converts RTL to gate-level netlist.
    
  Relevant github repositories (sky130RTLDesignAndSynthesisWorkshop, vsdflow) are first cloned using the following command in a relevant directory:
  ```git clone https://www.github.com/(insert_file_name)```
  
  The repository "vsdflow" contains relevant tools for this workshop and "sky130RTLDesignAndSynthesisWorkshop" contains various pre-written verilog files, model files and .lib file for understanding the design flow.
  
  ![Capture28](../main/images/Capture28.PNG)

  The second part of this day involves simulation of 2:1 Mux. The following images contain verilog behavioral file and testbench of the 2:1 Mux which can be read in linux by running the following command:
  ```
  gvim tb_good_mux -o good_mux
  ```
  
  ![This is an image](../main/images/Capture29.PNG)
  
  ![This is an image](../main/images/Capture30.PNG)
    
  ### Simulation of 2:1 Mux using iverilog
  Iverilog is a simulator. It takes in a behavioural model of a design and corresponding testbench as its arguments. The testbench generates necessary stimulus in the design (which is basically changes in primary inputs over time) and paves way for observing primary output changes. Iverilog then makes sense of this and generates an executable file (.out or .vvp file) which upon execution, dumps a .vcd file (Value Change Dump). This can later be read by gtkwave to visually observe the stimulus as waveforms.
  
  This sounds complicated but actually doing this involves only a few commands in linux:
  ```
  iverilog good_mux.v tb_good_mux.v
  ./a.out
  gtkwave tb_good_mux.vcd
  ```
  
  The following images contain the execution of all the above commands and the waveforms in the gtkwave viewer.
  
  ![This is an image](../main/images/Capture31.PNG)
  
  ![This is an image](../main/images/Capture32.PNG)
  
  ### Synthesis of 2:1 Mux using Yosys
  Yosys is a popular opensource synthesis tool. Synthesis tools like Yosys specifically require the RTL design file and .lib file. The RTL Design file is remodelled using logic gates (standard cells) provided in the .lib file. The remodelled gate-level version of the design file is called a "netlist". Before delving deep into the implementation in Yosys, let's take our time to check out the gates in a .lib file.
  
  A .lib file consists of various logic gates like AND2, OR2, NOR2, NAND2, XOR2, etc. There are various versions of the same gate as well, some of which are faster and others slower! This begs the question: <ins>what's the point in having different variations?</ins> There are very specific and harsh timing requirements gate-level circuits should meet (for example, setup time and hold time is the amount of time required for the input to a Flip-Flop to be stable before and after a clock edge respectively). If not, metastability might happen. In order to avoid this scenario, both slow and fast gates are required. But then, there are other consequences as well because the speed of a circuit depends on capacitance at the load; the wider the transistors in a gate, the less delay it will have but that'd mean more area and power (the inverse is also true). This is why proper constraint files should be supplied so that the synthesizer does its task properly and efficiently.
  
  Now, as for the implementation, it is pretty straightforward. First type the following in the terminal:
  
  ```
  yosys
  
  ```
  This opens up the Yosys prompt. After this, we'll have to execute several commands in the said prompt, all of which will be mentioned below.
  
  ```
  read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
  
  read_verilog good_mux.v
    
  synth -top good_mux
  ```

  ![This is an image](../main/images/Capture34.PNG)
  
  ```
  abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

  show
  ```
  
  ![This is an image](../main/images/Capture35.PNG)
  
  ![This is an image](../main/images/Capture36.PNG)
  
  Note that good_mux is synthesized as a cell named `sky130_fd_sc_hd__mux2_1` which basically is a 2:1 Multiplexer.
  
  ```
  write_verilog good_mux_netlist.v

  !gvim good_mux_netlist.v
  ```
  
  ![This is an image](../main/images/Capture37.PNG)
  
  ![This is an image](../main/images/Capture38.PNG)

## Day 2
  ### Understanding .lib file
  We have learnt what .lib has in the previous section. In this section, we'll cover .lib file in detail.
  
  **sky130_fd_sc_hd__tt_025C_1v80.lib - what does this mean?**
  | Symbol | Meaning |
  |--------|---------|
  | sky | skywater |
  | 130 | 180-130nm process node |
  | fd | foundry |
  | sc | standard cell |
  | hd | high density |
  | tt | Typical Process |
  | 025C | (25<sup>o</sup>C) Temperature |
  | 1v80 | (1.8V) Voltage |
  
  Then, we can use vim to open the file: `gvim ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib` to investigate this file. The file describes each gate in detail and lists out parameters like leakage power for all input combinations of each gate, cell spacing, etc.
  
  ![This is an image](../main/images/Capture39.PNG)
  
  The following is the verilog model of the same gate shown in the above image which can be opened using vim separately or typing `:sp ../my_lib/verilog_model/sky130_fd_sc_hd.v` in the same file. Then, we can locate the gate we were examining by using `/sky130_fd_sc_hd__a2111o_1`.
  
  ![This is an image](../main/images/Capture40.PNG)
  
  ### Hierarchical and Flat synthesis
  
  This section is dedicated to explain hierarchical and flat synthesis. Needless to say, Yosys will be used in this section. The file we will be trying to synthesize is `multiple_modules.v`. This file consists of a top module and two submodules (logically equivalent to OR and AND) as shown below. 
  
  ![This is an image](../main/images/Capture41.PNG)
  
  Now, the standard sequence of commands are executed in the Yosys prompt i.e. `read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib`, `read_verilog multiple_modules.v`, `synth -top multiple_modules`, `abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib`, `show multiple_modules` and `write_verilog -noattr multiple_modules_hier.v`. A noticeable difference can be observed in the dot viewer and in `multiple_modules_hier.v`. The submodules are preserved and they don't get flattened to gates by default, leading to hierarchical nature of the modules.
  
  ![This is an image](../main/images/Capture42.PNG)
  
  ![This is an image](../main/images/Capture43.PNG)
  
  ![This is an image](../main/images/Capture47.PNG)
  
  The images below correspond to the gate-level netlist of the hierarchy (which can be checked using dot viewer and vim).
  
  ![This is an image](../main/images/Capture44.PNG)
  
  ![This is an image](../main/images/Capture45.PNG)
  
  ![This is an image](../main/images/Capture46.PNG)
  
  > Trivia: From the above image, it's worth noting how OR logic need not get sythesized to OR gate.
  
  On the other hand, we can use `flatten` in the prompt to eliminate the hierarchies leading to flat synthesis. `show` can be used to view the gate level netlist. `write_verilog -noattr multiple_modules_flat.v` is used to generate verilog files corresponding to the netlist.
  
  ![This is an image](../main/images/Capture48.PNG)
  
  The images below correspond to the gate-level netlist of the flattened module (which can be checked using dot viewer and vim).
  
  ![This is an image](../main/images/Capture49.PNG)
  
  ![This is an image](../main/images/Capture50.PNG)
  
  We can also synthesize just the submodule instead of the topmodule by specifying in the synth command as `synth -top sub_module1` while executing the rest of the commands. The images below illustrate this.
  
  ![This is an image](../main/images/Capture51.PNG)
  
  ![This is an image](../main/images/Capture53.PNG)
  
  ![This is an image](../main/images/Capture52.PNG)
  
  The above approach is especially useful when there are multiple instances of same module. It's convenient to synthesize a module once and replicate it wherever the module is needed instead of directly synthesizing every instance of that module. In a massive design, a synthesizer might fail to synthesize desirably and so synthesizing lower level modules first and then proceeding with the top-level module will let the synthesizer produce the desired netlist.
  
  ### Optimization using Flip Flops
  
  Flip Flops are 1-bit memory elements. They are specifically used in this case to avoid undesirable momentary transitions called "Glitches". While glitches may be negligible in smaller combinational circuits, if there are a lot of stages in a design, glitches will occupy undesirably high amount of time between input transitions. However, we can use D-Flip Flops to capture at the right time via the clock edge and completely circumvent the problem. All of these concepts are illustrated in the following images:
  
  ![This is an image](../main/images/Capture54.PNG)
  
  ![This is an image](../main/images/Capture55.PNG)
  
  ![This is an image](../main/images/Capture56.PNG)
  
  Now that we have learnt why Flip Flops are used, it's time to learn about the various types of sets and resets available. Asynchronous resets reset the output (Q) of the Flip Flop to 0 regardless of input (D) right at the positive edge of the reset pin. They work independent of the clock cycle, whereas synchronous resets are dependent on the clock cycle. Only if there is a positive edge of the clock, synchronous reset works. Sets work the same way but they instead set the output of the Flip Flop (Q) to 1 regardless of input (D). Having both sets and resets make the design vulnerable to race conditions and having asynchronous resets/sets is generally costlier (in terms of power) than having their synchronous counterparts.
  
  The following image depicts the various resets in Flip Flops. 
  > Trivia: To split a tab into multiple sections, `:sp` is used in the editor.
  
  ![This is an image](../main/images/Capture57.PNG)
  
  Now that we have learnt various set/reset configurations of Flip Flops, it's time to simulate and synthesize them!
  As mentioned already, iverilog and gtkwave can be used in tandem to simulate and visualize the waveforms generated by the RTL design file.
  
  ![This is an image](../main/images/Capture58.PNG)
  
  ![This is an image](../main/images/Capture59.PNG)
  
  ![This is an image](../main/images/Capture60.PNG)
  
  > It's worth noting how output transitions to zero right when asyncres signal is high regardless of the clock edge. Similar is the case with asyncset signals. This is not the case with synchronous set/reset signals however. We'll take a look at them eventually.
  
  The following are the waveforms for a few other design files:
  
  ![This is an image](../main/images/Capture61.PNG)
  
  ![This is an image](../main/images/Capture62.PNG)
  
  > In the above image, at the time the marker is pointing, it is clear that even though syncres signal goes LOW and D is HIGH, Q is HIGH only after the next positive edge of the clock. Also, note how the waveform for Q is red initially. It's because synchronous reset/set don't reset or set the output of Flip Flop to 0/1. This makes Q crowbarred for sometime.
  
  Now, these three design files are synthesized using Yosys. After using `synth -top <module>`, `dfflibmap -liberty <lib_file>` must also be used to synthesize D flip flops wherever required. The following images are the results:
  
  ![This is an image](../main/images/Capture63.PNG)
  
  ![This is an image](../main/images/Capture64.PNG)
  
  ![This is an image](../main/images/Capture65.PNG)
  
  > Trivia: Note from above netlist how synchronous reset/set don't even use reset or set pin in D-FF because they are dependent on positive edge of the clock and can easily be simulated as a multiplexer or in this case, an isolation buffer.
  
  ```
  The day ends off with some interesting optimizations made by the synthesizer: Assuming that the inputs are ordered and sorted, multiplication by 2^n is just shifting the number to the right by n inputs and the first n inputs are now zero. This leads to the synthesizer not bothering to include any complex cells like multiplier and instead, just directly wire the output with corresponding inputs or zero. This effectively prevents power from being wasted unnecessarily.
  ```
  
## Day 3
  ### Overview
  
  This section covers the theory for some fundamental optimization techniques. We optimize for a lot of reasons, saving area and power being two important considerations. Combinational Logic can be optimized by Boolean Logic optimizations like K-Map, Quine-McKluskey. When a constant is propagated in the input, we can perform direct optimization. Sequential Logic is optimized by Sequential Constant Propagation or through advanced algorithms like State Optimization, Retiming and Sequential Logic Cloning (Floorplan Aware Synthesis).
  
  <ins>**Combinational Logic Optimization:**</ins>
  
  `Constant Propagation:`
  
  Let's take an abstract example first; let f(x,y) be a boolean function and let's pretend that x is HIGH always. Then f(x,y) = f(1,y) = g(y). Note that since this is a boolean function with respect to a single variable, g can either invert or pass the values of y. It's amazing how a complex two-input function can be simplified and hence optimized to wire/buffer or an inverter. The benefits of this technique is illustrated using an example.
  
  ![This is an image](../main/images/Capture66.PNG)
  
  `Boolean Logic Minimization:`
  
  An example is taken to demonstrate this as well.
  
  ![This is an image](../main/images/Capture67.PNG)
  
  <ins>**Sequential Logic Optimization:**</ins>
  
  `Sequential Constant Propagation:`
  
  This is almost the same as constant propagation in Combination Logic Optimization techniques but this has some more caveats. An example is chosen to illustrate this technique.
  
  ![This is an image](../main/images/Capture68.PNG)
  
  `Advanced Techniques:`
  
  <ins>State Optimization</ins> optimizes any unused states. <ins>Retiming</ins> distributes excess positive slack in one Flip Flop equally to other combinational logic, in order to improve frequency of execution, but keeping the total propagation delay the same. Let's take an arbitrary flip flop having a fanout of two other flip flops. Let's say after floorplanning, the master is separated far away from the slaves. In this case, let's say if the master FF had excess positive slack, <ins>Sequential Logic Cloning</ins> makes use of excess positive slack to physically clone the master into two different flip flops and hence making them fanout-of-one. These techniques will be illustrated in the following image.
  
  ![This is an image](../main/images/Capture69.PNG)
  
  ### Combinational Logic Optimizations
  
  This section is primarily going to revolve around the following files: `opt_check.v`, `opt_check2.v`, `opt_check3.v`, `opt_check4.v`, `multiple_module_opt.v` and `multiple_module_opt2.v`. One can use `gvim` to open and view them.
  
  ![This is an image](../main/images/Capture80.PNG)
  
  ![This is an image](../main/images/Capture71.PNG)
  
  Synthesis is performed using Yosys but this time, we also include `opt_clean -purge` right after `synth -top <file_name>`. The process is shown below.
  
  ![This is an image](../main/images/Capture72.PNG)
  
  ![This is an image](../main/images/Capture73.PNG)
  
  ![This is an image](../main/images/Capture74.PNG)
  
  Since we repeat the exact same steps for the other files, results are shown directly.
  
  ![This is an image](../main/images/Capture75.PNG)
  
  ![This is an image](../main/images/Capture76.PNG)
  
  ![This is an image](../main/images/Capture77.PNG)
  
  ![This is an image](../main/images/Capture78.PNG)
  
  ![This is an image](../main/images/Capture79.PNG)
  
  > Note that I have also used `flatten` to get the results shown in final two images. It's not possible to simplify submodules further after all.
  
  ### Sequential Logic Optimizations
  
  The files we will be using for this section is `dff_const*.v; * = {1,2,3,4,5}`. The images are shown below.
  
  ![This is an image](../main/images/Capture81.PNG)
  
  ![This is an image](../main/images/Capture82.PNG)
  
  Using iverilog and gtkwave, let's simulate every one of them. The results are shown below.
  
  ![This is an image](../main/images/Capture83.PNG)
  
  ![This is an image](../main/images/Capture84.PNG)
  
  ![This is an image](../main/images/Capture85.PNG)
  
  ![This is an image](../main/images/Capture86.PNG)
  
  ![This is an image](../main/images/Capture87.PNG)
  
  After synthesizing using Yosys, the following results are obtained. Note that `dfflibmap -liberty <liberty_file>` should be used after synth as well.
  
  ![This is an image](../main/images/Capture88.PNG)
  
  ![This is an image](../main/images/Capture89.PNG)
  
  ![This is an image](../main/images/Capture90.PNG)
  
  ![This is an image](../main/images/Capture91.PNG)
  
  ![This is an image](../main/images/Capture92.PNG)
  
  The final part of this section discusses about "Unused Outputs Optimization". The following files are used to demonstrate this.
  
  ![This is an image](../main/images/Capture93.PNG)
  
  The synthesis tool only cares about the output mentioned in the design file and there is no need for other unmentioned outputs. So, the synthesizer uses one DFF for the output instead of using three of them just to give the output.
  
  ![This is an image](../main/images/Capture94.PNG)
  
  If we use `opt_clean -purge`, the count variable itself is purged out.
  
  ![This is an image](../main/images/Capture95.PNG)

## Day 4: Gate-level Simulations
  ### GLS and Synth-Sim Mismatch
  
  > Gate-level Simulations exist to verify the design after synthesis. It involves running the testbench with netlist as Design Under Test. Netlist is logically the same as RTL code and consequently, the testbench used for verifying is the same as that for the simulation for RTL design. We could also ensure that the timing of the design is met by doing timing aware GLS.
  
  Synth-Sim Mismatch is when the simulation of the RTL design is not the same as the GLS. This can occur due to a variety of reasons like the improper use of Blocking Assignments. The following sections illustrate this.
  
  ### Missing Sensitivity List
  
  > When we fail to include all of the variables which directly affect output in the sensitivity list, the output only changes when the variables <ins>in the sensitivity list</ins> change. This is equivalent to a latch with some enable input, depending on the variables in the sensitivity list. The worse part is that the synthesis tool might not even detect a latch.

  The following images depict the RTL designs we will use in this section.
  
  ![This is an image](../main/images/Capture96.PNG)
  
  Let's first take a look at ternary_operator_mux.v. We first simulate using iverilog and gtkwave:
  
  ![This is an image](../main/images/Capture97.PNG)
  
  Now, we synthesize this using Yosys:
  
  ![This is an image](../main/images/Capture98.PNG)
  
  It's clear from the above image that this is what we wanted. Now, let's perform the following to do GLS.
  
  ```
  yosys> write_verilog ternary_operator_mux_net.v
  
  yosys> exit
  
  XXX: ~/Desktop/VLSI/sky130RTLDesignAndSynthesisWorkshop/verilog_files$ iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v ternary_operator_mux_net.v tb_ternary_operator.v
  
  XXX: ~/Desktop/VLSI/sky130RTLDesignAndSynthesisWorkshop/verilog_files$ ./a.out
  
  XXX: ~/Desktop/VLSI/sky130RTLDesignAndSynthesisWorkshop/verilog_files$ gtkwave tb_ternary_operator.vcd
  ```
  
  The iverilog arguments now also include the primitives and the sky130 file because the netlist encases gates and the details required for that are provided in those files. Once we do GLS, we get the following results and it's clear that everything works as intended.
  
  ![This is an image](../main/images/Capture99.PNG)
  
  Now, let's take a look at another file (bad_mux.v) which actually WILL result in synth-sim mismatch. `bad_mux.v` doesn't have all the variables in its sensitivity list. The output only responds when the value of sel changes.
  
  Using iverilog, gtkwave and yosys, we simulate the RTL design and after synthesis, we simulate the gate-level netlist. The following two images depict simulation of RTL code and simulation of the netlist respectively.
  
  ![This is an image](../main/images/Capture101.PNG)
  
  ![This is an image](../main/images/Capture102.PNG)
  
  From the above image, we can notice that the synthesized design somehow corrected our design and it now matches the `ternary_operator_mux.v` but unfortunately, the synthesizer doesn't always correct the design and the practice of not including all the necessary variables in the sensitivity list is bad in general and can throw the design cycle to chaos. It's better to keep `@(*)` where * is a wildcard and implies all the variables.
  
  ### Blocking Assignments
  
  > This concept is illustrated using an example.

  In this section, we are going to simulate and synthesize `blocking_caveat.v`. Note that the blocking assignments aren't in the right order in this design.
  
  ![This is an image](../main/images/Capture103.PNG)
  
  The following images hold the simulation of RTL design and GLS.
  
  ![This is an image](../main/images/Capture104.PNG)
  
  ![This is an image](../main/images/Capture106.PNG)
  
  It's happens to be the case that the synthesizer corrected the problem once again. But this doesn't happen always and has the chance to create some messy bugs.

## Day 5: If, Case, For, Generate
  ### If statements
  
  > If statements use priority logic which basically makes them equivalent to a set of stages of multiplexer. This can easily lead to "inferred latches" however if the statements don't handle all the cases i.e. exclusion of else statement especially when other cases are not covered.
  
  The following files are used in this section:
  
  IMG Capture107
  
  The following is the simulation and synthesis results of the first file.
  
  ![This is an image](../main/images/Capture108.PNG)
  
  ![This is an image](../main/images/Capture109.PNG)
  
  ![This is an image](../main/images/Capture110.PNG)
  
  The following is the simulation and synthesis results of the second file.
  
  ![This is an image](../main/images/Capture111.PNG)
  
  ![This is an image](../main/images/Capture112.PNG)
  
  ![This is an image](../main/images/Capture113.PNG)
  
  ### Case statements
  
  > Case statement, when not misused, is very similar to if statement. But unlike if statements, when there are parallel cases, every parallel case is executed as opposed to just the first matching case and then coming out of the cases.
  
  The following files are used in this section, with the first file being the ideal file.
  
  ![This is an image](../main/images/Capture114.PNG)
  
  Simulation, Synthesis of incomplete case statement:

  ![This is an image](../main/images/Capture115.PNG)

  ![This is an image](../main/images/Capture116.PNG)

  ![This is an image](../main/images/Capture117.PNG)
  
  Simulation, Synthesis of complete case statement:

  ![This is an image](../main/images/Capture118.PNG)

  ![This is an image](../main/images/Capture119.PNG)

  ![This is an image](../main/images/Capture120.PNG)
  
  Simulation, Synthesis of partial case statement:

  ![This is an image](../main/images/Capture121.PNG)

  ![This is an image](../main/images/Capture122.PNG)

  ![This is an image](../main/images/Capture123.PNG)
  
  Simulation, Synthesis, GLS of bad case statement:
  
  ![This is an image](../main/images/Capture124.PNG)

  ![This is an image](../main/images/Capture125.PNG)

  ![This is an image](../main/images/Capture126.PNG)
  
  ![This is an image](../main/images/Capture127.PNG)
  
  ### For statements
  
  > For statements evaluate expressions present within the loop, as many times as specified, in a compressed manner. For example, consider multiplexer, demultiplexers, encoders, decoders, etc. (especially ones with a lot of inputs or outputs; say 64:1 Mux or 1:32 Demux). Without a loop, the code will seem unnecessarily big and so, to shrink the code, we can use a blocking assignment alongside `for statements`.
  
  The files we are going to consider are:
  
  ![This is an image](../main/images/Capture128.PNG)
  
  Simulation for `mux_generate.v`:
  
  ![This is an image](../main/images/Capture129.PNG)
  
  Simulation for `demux_generate.v`:
  
  ![This is an image](../main/images/Capture132.PNG)
  
  ### Generate statements
  
  > Generate statements are paired with For statements or if statements to instantiate hardware modules. They are not used in `always` or `initial` blocks. They are used in adders or multipliers. Consider a 8-bit RCA. There are 8 full adders (7 FAs and 1 HA alternatively) in an 8-bit RCA. Instead of instantiating them one by one, we can do them altogether.

  The following is `rca.v`:
  
  ![This is an image](../main/images/Capture135.PNG)
  
  While simulating, note that we also have to include `fa.v` as an argument in the `iverilog` command. The following is the result of the simulation.
  
  ![This is an image](../main/images/Capture136.PNG)

## Credits
  1. [Kunalg github profile](https://github.com/kunalg123)

## References
  1. [Yosys Docs](http://yosyshq.net/yosys/documentation.html)
  2. [nandland](https://www.nandland.com/)
  3. [vsdiat](http://vsdiat.com/)
  4. [VLSI System Design](https://www.vlsisystemdesign.com/)
