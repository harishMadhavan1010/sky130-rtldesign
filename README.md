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

* Day 3: Combinational and Sequential Optimizations
  - Overview
  - Combinational Logic Optimizations
  - Sequential Logic Optimizations

* Day 4: Gate-level Simulations
  - Gate-level Simulations
  - Synth-Sim Mismatch
  - Blocking assignments

* Day 5: If, Case, For, Generate
  - If statements
  - Case statements
  - For statements
  - Generate statements

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
  
  ![Capture28](../tree/main/images/Capture28.PNG?raw=true)

  The second part of this day involves simulation of 2:1 Mux. The following images contain verilog behavioral file and testbench of the 2:1 Mux which can be read in linux by running the following command:
  ```
  gvim tb_good_mux -o good_mux
  ```
  
  ![This is an image](../image/Capture29)
  
  ![This is an image](../image/Capture30)
  
  ![image](https://user-images.githubusercontent.com/54830019/154884275-6f2a9fc1-3900-44f3-be9c-53b3ca14e226.png)
  
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
  
  ```
  read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
  
  read_verilog good_mux.v
    
  synth -top good_mux
  ```

  IMG Capture34
  
  ```
  abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

  show
  ```
  
  IMG Capture35
  
  IMG Capture36
  
  Note that good_mux is synthesized as a cell named `sky130_fd_sc_hd__mux2_1` which basically is a 2:1 Multiplexer.
  
  ```
  write_verilog good_mux_netlist.v

  !gvim good_mux_netlist.v
  ```
  
  IMG Capture37
  
  IMG Capture38

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
  
  IMG Capture39
  
  The following is the verilog model of the same gate shown in the above image which can be opened using vim separately or typing `:sp ../my_lib/verilog_model/sky130_fd_sc_hd.v` in the same file. Then, we can locate the gate we were examining by using `/sky130_fd_sc_hd__a2111o_1`.
  
  IMG Capture40
  
  ### Hierarchical and Flat synthesis
  
  This section is dedicated to explain hierarchical and flat synthesis. Needless to say, Yosys will be used in this section. The file we will be trying to synthesize is `multiple_modules.v`. This file consists of a top module and two submodules (logically equivalent to OR and AND) as shown below. 
  
  IMG Capture41
  
  Now, the standard sequence of commands are executed in the Yosys prompt i.e. `read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib`, `read_verilog multiple_modules.v`, `synth -top multiple_modules`, `abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib`, `show multiple_modules` and `write_verilog -noattr multiple_modules_hier.v`. A noticeable difference can be observed in the dot viewer and in `multiple_modules_hier.v`. The submodules are preserved and they don't get flattened to gates by default, leading to hierarchical nature of the modules.
  
  IMG Capture42
  
  IMG Capture43
  
  IMG Capture47
  
  The images below correspond to the gate-level netlist of the hierarchy (which can be checked using dot viewer and vim).
  
  IMG Capture44
  
  IMG Capture45
  
  IMG Capture46
  
  > Trivia: From the above image, it's worth noting how OR logic need not get sythesized to OR gate.
  
  On the other hand, we can use `flatten` in the prompt to eliminate the hierarchies leading to flat synthesis. `show` can be used to view the gate level netlist. `write_verilog -noattr multiple_modules_flat.v` is used to generate verilog files corresponding to the netlist.
  
  IMG Capture48
  
  The images below correspond to the gate-level netlist of the flattened module (which can be checked using dot viewer and vim).
  
  IMG Capture49
  
  IMG Capture50
  
  We can also synthesize just the submodule instead of the topmodule by specifying in the synth command as `synth -top sub_module1` while executing the rest of the commands. The images below illustrate this.
  
  IMG Capture51
  
  IMG Capture53
  
  IMG Capture52
  
  The above approach is especially useful when there are multiple instances of same module. It's convenient to synthesize a module once and replicate it wherever the module is needed instead of directly synthesizing every instance of that module. In a massive design, a synthesizer might fail to synthesize desirably and so synthesizing lower level modules first and then proceeding with the top-level module will let the synthesizer produce the desired netlist.
  
  ### Optimization using Flip Flops
  
  Flip Flops are 1-bit memory elements. They are specifically used in this case to avoid undesirable momentary transitions called "Glitches". While glitches may be negligible in smaller combinational circuits, if there are a lot of stages in a design, glitches will occupy undesirably high amount of time between input transitions. However, we can use D-Flip Flops to capture at the right time via the clock edge and completely circumvent the problem. All of these concepts are illustrated in the following images:
  
  IMG Capture54
  
  IMG Capture55
  
  IMG Capture56
  
  Now that we have learnt why Flip Flops are used, it's time to learn about the various types of sets and resets available. Asynchronous resets reset the output (Q) of the Flip Flop to 0 regardless of input (D) right at the positive edge of the reset pin. They work independent of the clock cycle, whereas synchronous resets are dependent on the clock cycle. Only if there is a positive edge of the clock, synchronous reset works. Sets work the same way but they instead set the output of the Flip Flop (Q) to 1 regardless of input (D). Having both sets and resets make the design vulnerable to race conditions and having asynchronous resets/sets is generally costlier (in terms of power) than having their synchronous counterparts.
  
  The following image depicts the various resets in Flip Flops. 
  > Trivia: To split a tab into multiple sections, `:sp` is used in the editor.
  
  IMG Capture57
  
  Now that we have learnt various set/reset configurations of Flip Flops, it's time to simulate and synthesize them!
  As mentioned already, iverilog and gtkwave can be used in tandem to simulate and visualize the waveforms generated by the RTL design file.
  
  IMG Capture58
  
  IMG Capture59
  
  IMG Capture60
  
  > It's worth noting how output transitions to zero right when asyncres signal is high regardless of the clock edge. Similar is the case with asyncset signals. This is not the case with synchronous set/reset signals however. We'll take a look at them eventually.
  
  The following are the waveforms for a few other design files:
  
  IMG Capture61
  
  IMG Capture62
  
  > In the above image, at the time the marker is pointing, it is clear that even though syncres signal goes LOW and D is HIGH, Q is HIGH only after the next positive edge of the clock. Also, note how the waveform for Q is red initially. It's because synchronous reset/set don't reset or set the output of Flip Flop to 0/1. This makes Q crowbarred for sometime.
  
  Now, these three design files are synthesized using Yosys. After using `synth -top <module>`, `dfflibmap -liberty <lib_file>` must also be used to synthesize D flip flops wherever required. The following images are the results:
  
  IMG Capture63
  
  IMG Capture64
  
  IMG Capture65
  
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
  
  IMG Capture66
  
  `Boolean Logic Minimization:`
  
  An example is taken to demonstrate this as well.
  
  IMG Capture67
  
  <ins>**Sequential Logic Optimization:**</ins>
  
  `Sequential Constant Propagation:`
  
  This is almost the same as constant propagation in Combination Logic Optimization techniques but this has some more caveats. An example is chosen to illustrate this technique.
  
  IMG Capture68
  
  `Advanced Techniques:`
  
  <ins>State Optimization</ins> optimizes any unused states. <ins>Retiming</ins> distributes excess positive slack in one Flip Flop equally to other combinational logic, in order to improve frequency of execution, but keeping the total propagation delay the same. Let's take an arbitrary flip flop having a fanout of two other flip flops. Let's say after floorplanning, the master is separated far away from the slaves. In this case, let's say if the master FF had excess positive slack, <ins>Sequential Logic Cloning</ins> makes use of excess positive slack to physically clone the master into two different flip flops and hence making them fanout-of-one. These techniques will be illustrated in the following image.
  
  IMG Capture69
  
  ### Combinational Logic Optimizations
  
  This section is primarily going to revolve around the following files: `opt_check.v`, `opt_check2.v`, `opt_check3.v`, `opt_check4.v`, `multiple_module_opt.v` and `multiple_module_opt2.v`. One can use `gvim` to open and view them.
  
  IMG Capture80
  
  IMG Capture71
  
  Synthesis is performed using Yosys but this time, we also include `opt_clean -purge` right after `synth -top <file_name>`. The process is shown below.
  
  IMG Capture72
  
  IMG Capture73
  
  IMG Capture74
  
  Since we repeat the exact same steps for the other files, results are shown directly.
  
  IMG Capture75
  
  IMG Capture76
  
  IMG Capture77
  
  IMG Capture78
  
  IMG Capture79
  
  > Note that I have also used `flatten` to get the results shown in final two images. It's not possible to simplify submodules further after all.
  
  ### Sequential Logic Optimizations
  
  The files we will be using for this section is `dff_const*.v; * = {1,2,3,4,5}`. The images are shown below.
  
  IMG Capture81
  
  IMG Capture82
  
  Using iverilog and gtkwave, let's simulate every one of them. The results are shown below.
  
  IMG Capture83
  
  IMG Capture84
  
  IMG Capture85
  
  IMG Capture86
  
  IMG Capture87
  
  After synthesizing using Yosys, the following results are obtained. Note that `dfflibmap -liberty <liberty_file>` should be used after synth as well.
  
  IMG Capture88
  
  IMG Capture89
  
  IMG Capture90
  
  IMG Capture91

  IMG Capture92
  
  The final part of this section discusses about "Unused Outputs Optimization". The following files are used to demonstrate this.
  
  IMG Capture93
  
  The synthesis tool only cares about the output mentioned in the design file and there is no need for other unmentioned outputs. So, the synthesizer uses one DFF for the output instead of using three of them just to give the output.
  
  IMG Capture94

## Day 4

## Day 5

## Credits
[a] Kunalg

## References
