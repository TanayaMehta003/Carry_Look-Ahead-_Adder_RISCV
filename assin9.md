## Icarus Verilog-Based Simulation Flow

A simulator is used to validate whether a design aligns with its intended specifications by running simulations on the code. It monitors changes in input signals, and whenever these inputs vary, the corresponding outputs are recalculated. RTL (Register Transfer Level) design refers to the Verilog code that describes the behavior and structure of a digital circuit. To ensure correctness, a testbench is developed to test the design through simulation using Icarus Verilog.

During the simulation, a VCD (Value Change Dump) file is generated, which logs signal transitions. This file is analyzed using GTKWave, a waveform viewer that assists in debugging and verifying circuit behavior. GTKWave allows users to explore signal interactions, inspect timing relationships, and validate the overall operation of the design by visualizing the waveforms created during simulation.

## Setup

Enter the below command in the Ubuntu terminal:
	  
```
sudo -i
sudo apt-get install git
ls
cd Desktop
mkdir ASIC
cd ASIC
git clone https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop.git
cd sky130RTLDesignAndSynthesisWorkshop/verilog_files
ls
```

We observe the following list of files present in the directory. 

![Screenshot from 2024-10-21 00-30-37](https://github.com/user-attachments/assets/ff904975-c47a-4500-9228-0c8362e08bf9)

## DAY 1
		  

### Introduction to Icarus Verilog and GTKWave

This tutorial covers the basics of simulating a 2x1 multiplexer design along with its testbench using Icarus Verilog (iverilog). Additionally, it demonstrates how to view the generated waveforms using GTKWave for better analysis and verification of the design.

   Run the following commands
```
iverilog good_mux.v tb_good_mux.v
./a.out
gtkwave tb_good_mux.vcd

 ```	
![Screenshot from 2024-10-21 00-51-22](https://github.com/user-attachments/assets/750fc0e4-2cfe-4de2-a4dc-1d004bcddaa6)


  #### MUX.V

```
  module good_mux (input i0, input i1, input sel, output reg y);
	  always@(*)
	  begin
	  	if(sel)
			y<=i1;
		else
			y<=i0;
	  end
  endmodule
```

  #### MUX_TB.V

  ```
  module tb_good_mux;
	reg i0,i1,sel;
	wire y;

     	good_mux uut(.sel(sel),.i0(i0),.i1(i1),.y(y));

	initial begin
		$dumpfile("tb_good_mux.vcd");
		$dumpvars(0,tb_good_mux);
		sel=0;
		i0=0;
		i1=0;
		#300 $finish;
	end
	always #75 sel = ~sel;
	always #10 i0 = ~i0;
	always #55 i1 = ~i1;
  endmodule
  ```

 
### Introduction to Yosys:

This tutorial demonstrates how to synthesize a Verilog design using Yosys, explore the generated netlists, and examine the cells used for building the circuit. The following commands were utilized throughout the process:

### Yosys 

1. Launches the Yosys tool.
 ```
yosys
```
![Screenshot from 2024-10-21 01-40-30](https://github.com/user-attachments/assets/adbbd872-5a0a-4b95-81be-65e7d6eb61d3)

2. **Load the Technology Library**
   Load the technology library file (in Liberty format) required for synthesis from the designated path.
   ```sh
   read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
   ```

3. **Import the Verilog File**
   Import the Verilog file `good_mux.v` for synthesis purposes.
   ```sh
   read_verilog good_mux.v
   ```

4. **Execute Synthesis**
   Execute synthesis on the design, using `good_mux` as the top-level module.
   ```sh
   synth -top good_mux
   ```

5. **Enhance the Design**
   Utilize the ABC tool to enhance the synthesized design with the provided technology library.
   ```sh
   abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
   ```

6. **Display the Synthesized Design**
   Show the synthesized design in the form of a schematic.
   ```sh
   show
   ```
![Screenshot from 2024-10-21 01-19-13](https://github.com/user-attachments/assets/9f582f40-1115-419e-82ec-69a9eb27d97e)

7. **Output the Synthesized Netlist**
   Write the synthesized netlist to the file `good_mux_netlist.v`, omitting attributes.
   ```sh
   write_verilog -noattr good_mux_netlist.v
   ```

8. **Open the Netlist File**
   Open the netlist file `good_mux_netlist.v` in the gvim text editor.
   ```sh
   !gvim good_mux_netlist.v

### Netlist generated

![Screenshot from 2024-10-21 01-20-27](https://github.com/user-attachments/assets/c5f94c7b-75c8-4be1-aebb-33c76a1e3864)
  
#### Yosys screenshots

![Screenshot from 2024-10-21 01-44-12](https://github.com/user-attachments/assets/e657c68f-4b36-4b35-aeec-06ade4035573)
![Screenshot from 2024-10-21 01-44-38](https://github.com/user-attachments/assets/81babc42-afdb-4cba-af14-efc23967fa77)
![Screenshot from 2024-10-21 01-45-04](https://github.com/user-attachments/assets/b87e8608-76a5-44dd-82b5-1e63551477b7)
 
 ## Day 2: Timing libs, hierarchical vs flat synthesis and efficient flop coding styles

Run the commands mentioned to view the contents inside the .lib file:
```
cd ..
ls
cd lib
vim sky130_fd_sc_hd__tt_025C_1v80.lib
```

The (.lib) files contain PVT parameters—Process, Voltage, and Temperature. Variations in these parameters can have a substantial impact on circuit performance. Factors such as manufacturing inconsistencies, voltage fluctuations, and temperature shifts contribute to these effects.

![Screenshot from 2024-10-21 13-05-37](https://github.com/user-attachments/assets/53c8af6f-105b-421b-b055-cf879c8394b5)


### Hierarchical vs. Flat Synthesis

**Hierarchical synthesis** breaks down a complex design into multiple sub-modules. 
Each sub-module is synthesized separately into gate-level netlists, which are later 
integrated to form the complete design. This approach promotes better organization, 
enables the reuse of modules, and allows for incremental changes without impacting the entire system.

In contrast, **flat synthesis** treats the entire design as a single, unified block 
during the synthesis process, ignoring any hierarchical relationships. While this 
method can optimize certain designs, it can make the design more challenging to maintain, 
analyze, and modify due to the absence of modular structure.

Consider the Verilog file `multiple_modules.v` located in the `verilog_files` directory 
for a practical example of hierarchical synthesis.
  
### Yosys Synthesis for Multiple Modules

Execute the below mentioned yosys commands for multiple modules

```
yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog multiple_modules.v
synth -top multiple_modules
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show multiple_modules
write_verilog -noattr multiple_modules_netlist.v
!gvim multiple_modules_netlist.v
```


#### multiple_modules.v

```

module sub_module2 (input a, input b, output y);
	assign y = a | b;
endmodule

module sub_module1 (input a, input b, output y);
	assign y = a&b;
endmodule

module multiple_modules (input a, input b, input c, output y);
	wire net1;
	sub_module1 u1(.a(a),.b(b),.y(net1)); //net1 = a&b
	sub_module2 u2(.a(net1),.b(c),.y(y)); //y = netic,ie y = a&b + c;
endmodule
```

#### Hierarchical generated Netlist
```
/* Generated by Yosys 0.23 (git sha1 7ce5011c24b) */

module multiple_modules(a, b, c, y);
  input a;
  wire a;
  input b;
  wire b;
  input c;
  wire c;
  wire net1;
  output y;
  wire y;
  sub_module1 u1 (
    .a(a),
    .b(b),
    .y(net1)
  );
  sub_module2 u2 (
    .a(net1),
    .b(c),
    .y(y)
  );
endmodule

module sub_module1(a, b, y);
  wire _0_;
  wire _1_;
  wire _2_;
  input a;
  wire a;
  input b;
  wire b;
  output y;
  wire y;
  sky130_fd_sc_hd__and2_0 _3_ (
    .A(_1_),
    .B(_0_),
    .X(_2_)
  );
  assign _1_ = b;
  assign _0_ = a;
  assign y = _2_;
endmodule

module sub_module2(a, b, y);
  wire _0_;
  wire _1_;
  wire _2_;
  input a;
  wire a;
  input b;
  wire b;
  output y;
  wire y;
  sky130_fd_sc_hd__or2_0 _3_ (
    .A(_1_),
    .B(_0_),
    .X(_2_)
  );
  assign _1_ = b;
  assign _0_ = a;
  assign y = _2_;
endmodule
```

![Screenshot from 2024-10-21 01-53-12](https://github.com/user-attachments/assets/9180fbe6-6180-4ce6-a4ce-ef6f094555c0)

Yosys screenshot

![Screenshot from 2024-10-21 01-47-16](https://github.com/user-attachments/assets/6175c1ba-594f-43b1-8d36-474f891ffd7e)
![Screenshot from 2024-10-21 01-48-35](https://github.com/user-attachments/assets/07488850-0aa4-4d79-b7e0-ccc79f2676d7)
![Screenshot from 2024-10-21 01-49-06](https://github.com/user-attachments/assets/9e13a965-1026-454f-8bf9-fb55962e0ee1)
![Screenshot from 2024-10-21 01-54-08](https://github.com/user-attachments/assets/758e640c-138a-4498-bc30-4e5b970b4189)

To perform **flat synthesis** on the `multiple_modules.v` file, use the following commands:

 Merges all hierarchical modules in the design into a single module to create a flat netlist.
 
 ```
1. yosys
2. read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
3. read_verilog multiple_modules.v
4. synth -top multiple_modules
5. abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
6. flatten
7. show
8. write_verilog -noattr multiple_modules_netlist.v
9. !gvim multiple_modules_flat.v
```

![Screenshot from 2024-10-21 15-38-20](https://github.com/user-attachments/assets/fe06f2c7-ccb0-469f-afe8-35a4e5ef176a)


#### Flat synthesis generated Netlist
```
/* Generated by Yosys 0.7 (git sha1 61f6811, gcc 6.2.0-11ubuntu1 -O2 -fdebug-prefix-map=/build/yosys-OIL3SR/yosys-0.7=. -fstack-protector-strong -fPIC -Os) */

module multiple_modules(a, b, c, y);
  wire _00_;
  wire _01_;
  wire _02_;
  wire _03_;
  wire _04_;
  wire _05_;
  wire _06_;
  wire _07_;
  input a;
  input b;
  input c;
  wire net1;
  wire \u1.a ;
  wire \u1.b ;
  wire \u1.y ;
  wire \u2.a ;
  wire \u2.b ;
  wire \u2.y ;
  output y;
  sky130_fd_sc_hd__and2_2 _08_ (
    .A(_00_),
    .B(_01_),
    .X(_02_)
  );
  sky130_fd_sc_hd__clkinv_1 _09_ (
    .A(_03_),
    .Y(_06_)
  );
  sky130_fd_sc_hd__clkinv_1 _10_ (
    .A(_04_),
    .Y(_07_)
  );
  sky130_fd_sc_hd__nand2_1 _11_ (
    .A(_06_),
    .B(_07_),
    .Y(_05_)
  );
  assign \u1.a  = a;
  assign \u1.b  = b;
  assign net1 = \u1.y ;
  assign _00_ = \u1.b ;
  assign _01_ = \u1.a ;
  assign \u1.y  = _02_;
  assign \u2.a  = net1;
  assign \u2.b  = c;
  assign y = \u2.y ;
  assign _03_ = \u2.b ;
  assign _04_ = \u2.a ;
  assign \u2.y  = _05_;
endmodule
```

![Screenshot from 2024-10-21 15-46-44](https://github.com/user-attachments/assets/1b0b580f-b478-493a-aef6-18b18a55df57)
![Screenshot from 2024-10-21 15-47-09](https://github.com/user-attachments/assets/913ee2f5-3d7d-41c5-87b1-38012444b84c)

### Various Flip-Flop Coding Styles and Optimizations

Flip-flops play a crucial role in sequential logic circuits. They are used to store intermediate values, ensuring that inputs to combinational circuits remain stable until the next clock edge. This stability helps prevent glitches, ensuring proper operation of the circuit. By capturing and holding data, flip-flops help maintain the integrity of signals and facilitate the correct sequencing of operations in digital systems.

Simulations were conducted for three types of D-Flip-Flops:  
1. **Asynchronous Reset**  
2. **Asynchronous Set**  
3. **Synchronous Reset**  

These simulations demonstrate the behavior of each type, highlighting how they respond to different reset and set conditions, ensuring accurate storage and control of data during circuit operation.

### Asynchronous Reset

#### Verilog code:

```
module dff_asyncres(input clk, input async_reset, input d, output reg q);
	always@(posedge clk, posedge async_reset)
	begin
		if(async_reset)
			q <= 1'b0;
		else
			q <= d;
	end
endmodule
```

#### Testbench
```
module tb_dff_asyncres; 
	reg clk, async_reset, d;
	wire q;
	dff_asyncres uut (.clk(clk),.async_reset (async_reset),.d(d),.q(q));

	initial begin
		$dumpfile("tb_dff_asyncres.vcd");
		$dumpvars(0,tb_dff_asyncres);
		// Initialize Inputs
		clk = 0;
		async_reset = 1;
		d = 0;
		#3000 $finish;
	end
		
	always #10 clk = ~clk;
	always #23 d = ~d;
	always #547 async_reset=~async_reset; 
endmodule
```
Run the below commands for simulation:

```
iverilog dff_asyncres.v tb_dff_asyncres.v
./a.out
gtkwave tb_dff_asyncres.vcd
```
Output:

![Screenshot from 2024-10-21 16-44-26](https://github.com/user-attachments/assets/ea076a05-17d4-47aa-a94d-5f576a2ca9d4)

From the waveform, it can be observed that the **Q** output transitions to zero when the **asynchronous reset** is set high, regardless of the positive or negative clock edge. This demonstrates the independent nature of the asynchronous reset, allowing it to override other inputs to reset the output immediately.

Run the below commands to view netlist:

1. yosys
2. read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
3. read_verilog dff_asyncres.v
4. synth -top dff_asyncres
5. dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
7. show
8. write_verilog -noattr dff_asyncres_netlist.v
9. !gvim dff_asyncres_netlist.v

Netlist:

![Screenshot from 2024-10-21 16-48-44](https://github.com/user-attachments/assets/1ba4e1d6-c534-4a0e-8446-4998649e2635)

Netlist code:
```
/* Generated by Yosys 0.23 (git sha1 7ce5011c24b) */

module dff_asyncres(clk, async_reset, d, q);
  wire _0_;
  input async_reset;
  wire async_reset;
  input clk;
  wire clk;
  input d;
  wire d;
  output q;
  wire q;
  assign _0_ = ~async_reset;
  sky130_fd_sc_hd__dfrtp_1 _2_ (
    .CLK(clk),
    .D(d),
    .Q(q),
    .RESET_B(_0_)
  );
endmodule
```
Yosys screenshot			

![Screenshot from 2024-10-21 16-47-08](https://github.com/user-attachments/assets/29c16cd6-2b2a-4c0d-8633-281a4d956aea)

### Synchronous Reset

#### Verilog code:
```
module dff_syncres(input clk, input sync_reset, input d, output reg q);
	always@(posedge clk)
	begin
		if(sync_reset)
			q <= 1'b0;
		else
			q <= d;
	end
endmodule
```

#### Testbench

```
module tb_dff_syncres; 
	reg clk, syncres, d;
	wire q;
	dff_asyncres uut (.clk(clk),.sync_reset (sync_reset),.d(d),.q(q));

	initial begin
		$dumpfile("tb_dff_syncres.vcd");
		$dumpvars(0,tb_dff_syncres);
		// Initialize Inputs
		clk = 0;
		sync_reset = 1;
		d = 0;
		#3000 $finish;
	end
		
	always #10 clk = ~clk;
	always #23 d = ~d;
	always #547 sync_reset=~async_reset; 
endmodule
```

Run the below commands for simulation:
```
iverilog dff_syncres.v tb_dff_syncres.v
./a.out
gtkwave tb_dff_syncres.vcd
```
Output

![Screenshot from 2024-10-21 17-46-28](https://github.com/user-attachments/assets/f394c882-9dd1-4d79-8ae1-af4c75bf69ba)

From the waveform, it can be observed that the **Q** output transitions to zero when the **synchronous reset** is set high, but only at the **positive clock edge**. This behavior illustrates that the synchronous reset affects the output in alignment with the clock, ensuring controlled and predictable operation.

Run the below commands to view netlist:

1. yosys
2. read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
3. read_verilog dff_syncres.v
4. synth -top dff_syncres
5. dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
6. abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
7. show
8. write_verilog -noattr dff_syncres_netlist.v
9. !gvim dff_syncres_netlist.v

Netlist:

![Screenshot from 2024-10-21 17-54-04](https://github.com/user-attachments/assets/268b6e9a-c895-41e4-b28f-fdddb9f2b33b)


Netlist code:
```
/* Generated by Yosys 0.23 (git sha1 7ce5011c24b) */

module dff_syncres(clk, async_reset, sync_reset, d, q);
  wire _0_;
  wire _1_;
  wire _2_;
  wire _3_;
  input async_reset;
  wire async_reset;
  input clk;
  wire clk;
  input d;
  wire d;
  output q;
  wire q;
  input sync_reset;
  wire sync_reset;
  sky130_fd_sc_hd__nor2b_1 _4_ (
    .A(_2_),
    .B_N(_1_),
    .Y(_0_)
  );
  sky130_fd_sc_hd__dfxtp_1 _5_ (
    .CLK(clk),
    .D(_3_),
    .Q(q)
  );
  assign _1_ = d;
  assign _2_ = sync_reset;
  assign _3_ = _0_;
endmodule
```
Yosys screenshot			

![Screenshot from 2024-10-21 17-52-00](https://github.com/user-attachments/assets/e15240f9-8398-4104-b092-ff22cbd67dde)

      
### Asynchronous Set

#### Verilog code
```
module dff_async_set(input clk, input async_set, input d, output reg q);
	always@(posedge clk, posedge async_set)
	begin
		if(async_set)
			q <= 1'b1;
		else
			q <= d;
	end
endmodule
```

#### Testbench
```
module tb_dff_async_set; 
	reg clk, async_set, d;
	wire q;
	dff_async_set uut (.clk(clk),.async_set (async_set),.d(d),.q(q));

	initial begin
		$dumpfile("tb_dff_async_set.vcd");
		$dumpvars(0,tb_dff_async_set);
		// Initialize Inputs
		clk = 0;
		async_set = 1;
		d = 0;
		#3000 $finish;
	end
		
	always #10 clk = ~clk;
	always #23 d = ~d;
	always #547 async_set=~async_set; 
endmodule
```
Run the below commands for simulation:

```
iverilog dff_async_set.v tb_dff_async_set.v
./a.out
gtkwave tb_dff_async_set.vcd
```

Output:

![Screenshot from 2024-10-21 17-59-15](https://github.com/user-attachments/assets/97adade9-b985-4314-86b8-445beaf752dc)

From the waveform, it can be observed that the **Q** output transitions to one when the **asynchronous set** is set high, regardless of the positive or negative clock edge. This demonstrates the independent nature of the asynchronous set, allowing it to override other inputs and immediately set the output to one.

Run the below commands to view netlist:

1. yosys
2. read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
3. read_verilog dff_async_set.v
4. synth -top dff_async_set
5. dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
6. abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
7. show
8. write_verilog -noattr dff_async_set_netlist.v
9. !gvim dff_syncres_netlist.v

Netlist:

![Screenshot from 2024-10-21 18-04-46](https://github.com/user-attachments/assets/9fd6ec4e-3f43-4940-b150-b357df4ed341)


Netlist code:
```
/* Generated by Yosys 0.23 (git sha1 7ce5011c24b) */

module dff_syncres(clk, async_reset, sync_reset, d, q);
  wire _0_;
  wire _1_;
  wire _2_;
  wire _3_;
  input async_reset;
  wire async_reset;
  input clk;
  wire clk;
  input d;
  wire d;
  output q;
  wire q;
  input sync_reset;
  wire sync_reset;
  sky130_fd_sc_hd__nor2b_1 _4_ (
    .A(_2_),
    .B_N(_1_),
    .Y(_0_)
  );
  sky130_fd_sc_hd__dfxtp_1 _5_ (
    .CLK(clk),
    .D(_3_),
    .Q(q)
  );
  assign _1_ = d;
  assign _2_ = sync_reset;
  assign _3_ = _0_;
endmodule
```
Yosys screenshot			

![Screenshot from 2024-10-21 18-03-10](https://github.com/user-attachments/assets/21046dca-e89f-45e9-b1fc-973d67cb1433)


### Optimization

#### Multiplication by 2

In this tutorial, we learn that dedicated multiplier hardware is not necessary to multiply a number by 2. This operation can be efficiently performed by **concatenating a zero to the Least Significant Bit (LSB)** of the number, effectively shifting all bits to the left by one position.

verilog code 
```
module mul2(input [2:0]a, output [3:0]y);
	assign y=a*2;
endmodule
```

Run the below commands to view netlist:

```
1. yosys
2. read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
3. read_verilog mult_2.v
4. synth -top mul2
5. abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
6. show
7. write_verilog -noattr mul2_net.v
8. !gvim mul2_net.v
```

Netlist:

![Screenshot from 2024-10-21 18-19-39](https://github.com/user-attachments/assets/f0277ae9-b267-46b3-91db-da16e6d20259)


Netlist code:
```
/* Generated by Yosys 0.23 (git sha1 7ce5011c24b) */

module mul2(a, y);
  input [2:0] a;
  wire [2:0] a;
  output [3:0] y;
  wire [3:0] y;
  assign y = { a, 1'h0 };
endmodule
```
Yosys screenshot			

![Screenshot from 2024-10-21 18-18-24](https://github.com/user-attachments/assets/f39d7cb5-f0a4-414d-bfa7-2ed9ef73ccd6)


#### Multiplication by 9

In this tutorial, we learn that dedicated multiplier hardware is not needed to multiply a number by 9. This can be efficiently achieved by **concatenating the number with itself**, effectively performing the multiplication through bit manipulation.

verilog code 
```
module mult8 (input [2:0] a , output [5:0] y);
	assign y = a * 9;
endmodule
```

Run the below commands to view netlist:

```
1. yosys
2. read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
3. read_verilog mult_8.v
4. synth -top mult8
5. abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
6. show
7. write_verilog -noattr mult_8_net.v
8. !gvim mult_8_net.v
```

Netlist:

![Screenshot from 2024-10-21 18-32-58](https://github.com/user-attachments/assets/24bd8001-1ae4-4539-adb3-6419da480f9a)


Netlist code:
```
/* Generated by Yosys 0.23 (git sha1 7ce5011c24b) */

module mult8(a, y);
  input [2:0] a;
  wire [2:0] a;
  output [5:0] y;
  wire [5:0] y;
  assign y = { a, a };
endmodule
```
Yosys screenshot			

![Screenshot from 2024-10-21 18-30-32](https://github.com/user-attachments/assets/450c54c9-e7f3-41ba-b42a-b6a476e6ecd7)

 

## Day 3: Combinational and sequential Optimizationsof Various Designs

### Types of Optimizations: Combinational and Sequential

These optimizations aim to create designs that are efficient in terms of **area**, **power**, and **performance**.

#### Combinational Optimization  
The techniques used in this type of optimization include:

- **Constant Propagation (Direct Optimization):**  
  Simplifies logic by directly substituting known constant values, reducing unnecessary gates.

- **Boolean Logic Optimization:**  
  Minimizes logic expressions using methods such as **Karnaugh Maps (K-Map)** or the **Quine-McCluskey algorithm**, leading to fewer gates and improved efficiency.


 #### 2 input AND Gate:

Verilog code: 
```
module opt_check(input a, input b, output y);
	assign y = a?b:0;
endmodule
```
Commands for netlist:

```
1. yosys
2. read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
3. read_verilog opt_check.v
4. synth -top opt_check
5. abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
6. opt_clean -purge
7. show
8. write_verilog -noattr opt_check.v
9. !gvim opt_check.v
```
Netlist:

![Screenshot from 2024-10-21 22-17-00](https://github.com/user-attachments/assets/a80371af-ef59-4bf0-b66c-2b0d3aa627c9)

Netlist Code:
```
/* Generated by Yosys 0.23 (git sha1 7ce5011c24b) */

module opt_check(a, b, y);
  input a;
  wire a;
  input b;
  wire b;
  output y;
  wire y;
  sky130_fd_sc_hd__and2_0 _0_ (
    .A(b),
    .B(a),
    .X(y)
  );
endmodule
```

Yosys screenshot

![Screenshot from 2024-10-21 22-15-31](https://github.com/user-attachments/assets/b0790e92-f1a5-422f-b06f-136ca85d5727)


#### 2 input OR Gate:

Verilog code:
```
module opt_check2(input a, input b, output y);
	assign y = a?1:b;
endmodule
```
Commands for netlist:

```
1. yosys
2. read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
3. read_verilog opt_check2.v
4. synth -top opt_check2
5. abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
6. opt_clean -purge
7. show
7. write_verilog -noattr opt_check2.v
8. !gvim opt_check2.v
```
Netlist:

![Screenshot from 2024-10-21 22-37-20](https://github.com/user-attachments/assets/e2a830de-7377-43c8-80a7-f7b0ba08a211)

Netlist Code:
```
/* Generated by Yosys 0.23 (git sha1 7ce5011c24b) */

module opt_check2(a, b, y);
  input a;
  wire a;
  input b;
  wire b;
  output y;
  wire y;
  sky130_fd_sc_hd__or2_0 _0_ (
    .A(a),
    .B(b),
    .X(y)
  );
endmodule
```

Yosys screenshot

![Screenshot from 2024-10-21 22-35-49](https://github.com/user-attachments/assets/8585191a-9453-4aed-9660-b3327e18a8e8)

#### 3 input AND Gate:

Verilog code:
```
module opt_check2(input a, input b, input c, output y);
	assign y = a?(b?c:0):0;
endmodule
```
Commands for netlist:

```
1. yosys
2. read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
3. read_verilog opt_check3.v
4. synth -top opt_check3
5. abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
6. opt_clean -purge
7. show
8. write_verilog -noattr opt_check3.v
8. !gvim opt_check3.v
```
Netlist:

![Screenshot from 2024-10-21 22-45-42](https://github.com/user-attachments/assets/14451574-cbb5-4aa9-a01f-a68a75ef7d28)


Netlist Code:
```
/* Generated by Yosys 0.23 (git sha1 7ce5011c24b) */

module opt_check3(a, b, c, y);
  input a;
  wire a;
  input b;
  wire b;
  input c;
  wire c;
  output y;
  wire y;
  sky130_fd_sc_hd__and3_1 _0_ (
    .A(b),
    .B(c),
    .C(a),
    .X(y)
  );
endmodule
```

Yosys screenshot

![Screenshot from 2024-10-21 22-44-13](https://github.com/user-attachments/assets/62d23c95-4675-4faf-9309-695a82b2ca46)


### 2 input XNOR Gate

Verilog code:
```
module opt_check2(input a, input b, input c, output y);
	assign y = a ? (b ? ~c : c) : ~c;
endmodule
```
Commands for netlist:

```
1. yosys
2. read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
3. read_verilog opt_check4.v
4. synth -top opt_check4
5. abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
6. opt_clean -purge
7. show
8. write_verilog -noattr opt_check4.v
8. !gvim opt_check4.v

```
Netlist:

![Screenshot from 2024-10-21 22-53-27](https://github.com/user-attachments/assets/9d5ec42c-bde1-49a6-86ee-7a7a5e415509)

Netlist Code:
```
/* Generated by Yosys 0.23 (git sha1 7ce5011c24b) */

module opt_check4(a, b, c, y);
  input a;
  wire a;
  input b;
  wire b;
  input c;
  wire c;
  output y;
  wire y;
  sky130_fd_sc_hd__xnor2_1 _0_ (
    .A(c),
    .B(a),
    .Y(y)
  );
endmodule
```

Yosys screenshot:
![Screenshot from 2024-10-21 22-52-41](https://github.com/user-attachments/assets/eb555790-04b1-4668-8857-598df51826dd)


#### Multiple Module Optimization-1

Verilog code:
```
module sub_module1(input a, input b, output y);
	assign y = a & b;
endmodule

module sub_module2 (input a, input b output y);
	assign y = a^b;
endmodule

module multiple_module_opt(input a, input b input c, input d output y);
	wire n1,n2, n3;

	sub_module1 U1 (.a(a), .b(1'b1), .y(n1));
	sub_module2 U2 (.a(n1), .b(1'b0), .y(n));
	sub_module2 U3 (.a(b), .b(d), .y(n3));

	assign y = c | (b & n1);
endmodule
```
Commands for netlist:
```
1. yosys
2. read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
3. read_verilog multiple_module_opt.v
4. synth -top multiple_module_opt
5. abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
6. opt_clean -purge
7. flatten
8. show
9. write_verilog -noattr multiple_module_opt.v
10. !gvim multiple_module_opt.v
```
Netlist:

![Screenshot from 2024-10-21 23-05-35](https://github.com/user-attachments/assets/15d75143-e1b3-48e5-ba30-15a69aff9b9a)

Netlist Code:
```
/* Generated by Yosys 0.23 (git sha1 7ce5011c24b) */

module multiple_module_opt(a, b, c, d, y);
  wire \U1.a ;
  wire \U1.b ;
  wire \U1.y ;
  input a;
  wire a;
  input b;
  wire b;
  input c;
  wire c;
  input d;
  wire d;
  wire n1;
  output y;
  wire y;
  sky130_fd_sc_hd__a21o_1 _0_ (
    .A1(n1),
    .A2(b),
    .B1(c),
    .X(y)
  );
  sky130_fd_sc_hd__and2_0 _1_ (
    .A(\U1.b ),
    .B(\U1.a ),
    .X(\U1.y )
  );
  assign \U1.a  = a;
  assign \U1.b  = 1'h1;
  assign n1 = \U1.y ;
endmodule
```

Yosys screenshot:
![Screenshot from 2024-10-21 23-01-06](https://github.com/user-attachments/assets/9a4684b9-78d6-43e2-b3ad-21f8ad9c43e6)

#### Multiple Module Optimization-2

Verilog code:

```
module sub_module(input a input b output y);
	assign y = a & b;
endmodule

module multiple_module_opt2(input a, input b input c, input d, output y);
	wire n1,n2, n3;

	sub_module U1 (.a(a), .b(1'b0), y(n));
	sub_module U2 (.a(b), .b(c), .y(n2));
	sub_module U3 (.a(n2), .b(d), .y(n));
	sub_module U4 (.a(n3), .b(n1), .y(y));
endmodule
```

Commands for netlist:
```
1. yosys
2. read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
3. read_verilog multiple_module_opt2.v
4. synth -top multiple_module_opt2
5. abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
6. opt_clean -purge
7. flatten
8. show
9. write_verilog -noattr multiple_module_opt2_net.v
10. !gvim multiple_module_opt2.v
```
Netlist:
![Screenshot from 2024-10-21 23-14-52](https://github.com/user-attachments/assets/061138ea-29c0-47c5-862d-6567af7ad2ca)

Netlist code:
```
module sub_module(input a , input b , output y);
 assign y = a & b;
endmodule

module multiple_module_opt2(input a , input b , input c , input d , output y);
wire n1,n2,n3;

sub_module U1 (.a(a) , .b(1'b0) , .y(n1));
sub_module U2 (.a(b), .b(c) , .y(n2));
sub_module U3 (.a(n2), .b(d) , .y(n3));
sub_module U4 (.a(n3), .b(n1) , .y(y));

endmodule
```

Yosys screenshot:
![Screenshot from 2024-10-21 23-13-01](https://github.com/user-attachments/assets/a08cded3-dafe-4798-a073-fd347db8ed21)


### Sequential Logic Optimizations


#### D-Flipflop Constant 1 with Asynchronous Reset (active low)

Verilog code:
```
module dff_const1(input clk, input reset, output reg q); 
always @(posedge clk, posedge reset)
begin
	if(reset)
		q <= 1'b0;
	else
		q <= 1'b1;
end
endmodule
```

Testbench:
```
module tb_dff_const1; 
	reg clk, reset;
	wire q;

	dff_const1 uut (.clk(clk),.reset(reset),.q(q));

	initial begin
		$dumpfile("tb_dff_const1.vcd");
		$dumpvars(0,tb_dff_const1);
		// Initialize Inputs
		clk = 0;
		reset = 1;
		#3000 $finish;
	end

	always #10 clk = ~clk;
	always #1547 reset=~reset;
endmodule
```
Simulation:

```
iverilog dff_const1.v tb_dff_const1.v
./a.out
gtkwave tb_dff_const1.vcd
```
![Screenshot from 2024-10-22 00-26-40](https://github.com/user-attachments/assets/06c70939-8c36-4324-b52b-932f9ecc8b42)

Commands for netlist:
   
```
1. yosys
2. read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
3. read_verilog dff_const1.v
4. synth -top dff_const1
5. dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
7. show
8. write_verilog -noattr dff_const1_net.v
9. !gvim dff_const1_net.v
```

Netlist:
![Screenshot from 2024-10-22 00-28-41](https://github.com/user-attachments/assets/b72fe889-02fd-4492-ac6d-6cca10b54220)


Netlist code:

```
/* Generated by Yosys 0.23 (git sha1 7ce5011c24b) */

module dff_const1(clk, reset, q);
  wire _0_;
  input clk;
  wire clk;
  output q;
  wire q;
  input reset;
  wire reset;
  assign _0_ = ~reset;
  sky130_fd_sc_hd__dfrtp_1 _2_ (
    .CLK(clk),
    .D(1'h1),
    .Q(q),
    .RESET_B(_0_)
  );
endmodule
```

Yosys screenshot:
![Screenshot from 2024-10-22 00-28-02](https://github.com/user-attachments/assets/ea89c56d-6276-42ec-9041-d0c287531353)


### D-Flipflop Constant 2 with Asynchronous Reset (active high)

Verilog code:

```
module dff_const2(input clk, input reset, output reg q); 
always @(posedge clk, posedge reset)
begin
	if(reset)
		q <= 1'b1;
	else
		q <= 1'b1;
end
endmodule
```
Testbench
```
module tb_dff_const2; 
	reg clk, reset;
	wire q;

	dff_const2 uut (.clk(clk),.reset(reset),.q(q));

	initial begin
		$dumpfile("tb_dff_const1.vcd");
		$dumpvars(0,tb_dff_const1);
		// Initialize Inputs
clk = 0;
		reset = 1;
		#3000 $finish;
	end

	always #10 clk = ~clk;
	always #1547 reset=~reset;
endmodule
```
Simulation:

```
iverilog dff_const2.v tb_dff_const2.v
./a.out
gtkwave tb_dff_const2.vcd
```
![Screenshot from 2024-10-22 00-30-10](https://github.com/user-attachments/assets/9c5bfad3-0575-49db-b73d-e36a671779cd)

Commands for netlist:

```
1. yosys
2. read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
3. read_verilog dff_const2.v
4. synth -top dff_const2
5. dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
7. show
8. write_verilog -noattr dff_const2_net.v
9. !gvim dff_const2_net.v
```
Netlist:
![Screenshot from 2024-10-22 00-32-35](https://github.com/user-attachments/assets/da3ce576-1178-4de9-837f-278656e72ec4)

Netlist code:
```
/* Generated by Yosys 0.23 (git sha1 7ce5011c24b) */

module dff_const2(clk, reset, q);
  input clk;
  wire clk;
  output q;
  wire q;
  input reset;
  wire reset;
  assign q = 1'h1;
endmodule
```
Yosys screenshot:
![Screenshot from 2024-10-22 00-31-40](https://github.com/user-attachments/assets/84dd7353-74e4-4ca3-9d0a-f7512a29287d)


### D-Flipflop Constant 3 with Asynchronous Reset (active low)

Verilog code:
```
module dff_const3(input clk, input reset, output reg q); 
	reg q1;

	always @(posedge clk, posedge reset)
	begin
		if(reset)
		begin
			q <= 1'b1;
			q1 <= 1'b0;
		end
		else
		begin	
			q1 <= 1'b1;
			q <= q1;
		end
	end
endmodule
```
Simulation:
```
iverilog dff_const3.v tb_dff_const3.v
./a.out
gtkwave tb_dff_const3.vcd
```
![Screenshot from 2024-10-22 00-34-15](https://github.com/user-attachments/assets/014d8a8d-ae10-4144-943f-7636258b2459)

Commands for netlist:
```
1. yosys
2. read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
3. read_verilog dff_const3.v
4. synth -top dff_const3
5. dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
7. show
8. write_verilog -noattr dff_const3_net.v
9. !gvim dff_const3_net.v
```
Netlist:
![Screenshot from 2024-10-22 00-35-53](https://github.com/user-attachments/assets/548d896d-7faa-4753-a333-364dd0c014e0)

Netlist code:
```
/* Generated by Yosys 0.23 (git sha1 7ce5011c24b) */

module dff_const3(clk, reset, q);
  wire _0_;
  wire _1_;
  input clk;
  wire clk;
  output q;
  wire q;
  wire q1;
  input reset;
  wire reset;
  assign _0_ = ~reset;
  assign _1_ = ~reset;
  sky130_fd_sc_hd__dfstp_2 _4_ (
    .CLK(clk),
    .D(q1),
    .Q(q),
    .SET_B(_0_)
  );
  sky130_fd_sc_hd__dfrtp_1 _5_ (
    .CLK(clk),
    .D(1'h1),
    .Q(q1),
    .RESET_B(_1_)
  );
endmodule
```

Yosys screenshot:
![Screenshot from 2024-10-22 00-35-11](https://github.com/user-attachments/assets/6d054b53-6717-4b2a-9db8-c50afebf7246)


### D-Flipflop Constant 4 with Asynchronous Reset (active high)

Verilog code:
```
module dff_const4(input clk, input reset, output reg q); 
	reg q1;

	always @(posedge clk, posedge reset)
	begin
		if(reset)
		begin
			q <= 1'b1;
			q1 <= 1'b1;
		end
		else
		begin	
			q1 <= 1'b1;
			q <= q1;
		end
	end
endmodule
```
Simulation:
```
iverilog dff_const4.v tb_dff_const4.v
./a.out
gtkwave tb_dff_const4.vcd
```
Commands for netlist:
```
1. yosys
2. read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
3. read_verilog dff_const4.v
4. synth -top dff_const4
5. dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
7. show
8. write_verilog -noattr dff_const4_net.v
9. !gvim dff_const4_net.v
```
Netlist:

Netlist code:

Yosys screenshot:



#### D-Flipflop Constant 5 with Asynchronous Reset

Verilog code:
```
module dff_const5(input clk, input reset, output reg q); 
	reg q1;

	always @(posedge clk, posedge reset)
	begin
		if(reset)
		begin
			q <= 1'b0;
			q1 <= 1'b0;
		end
		else
		begin	
			q1 <= 1'b1;
			q <= q1;
		end
	end
endmodule
```
Simulation:
```
iverilog dff_const5.v tb_dff_const5.v
./a.out
gtkwave tb_dff_const5.vcd
```
Commands for netlist:
```
1. yosys
2. read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
3. read_verilog dff_const5.v
4. synth -top dff_const5
5. dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
7. show
8. write_verilog -noattr dff_const5_net.v
9. !gvim dff_const5_net.v
```
Netlist:

Netlist code:

Yosys screenshot:

### Counter Optimization 1:

Verilog code:
```
module counter_opt (input clk, input reset, output q);
	reg [2:0] count;
	assign q = count[0];
	
	always @(posedge clk,posedge reset)
	begin
		if(reset)
			count <= 3'b000;
		else
			count <= count + 1;
	end
endmodule
```
Simulation:

```
iverilog counter_opt.v tb_counter_opt.v
./a.out
gtkwave tb_counter_opt.vcd
```
Commands for netlist:
```
1. yosys
2. read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
3. read_verilog counter_opt.v
4. synth -top counter_opt
5. dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
7. show
8. write_verilog -noattr counter_opt.v
9. !gvim counter_opt.v

```

Netlist:

Netlist code:

Yosys screenshot:

#### Counter Optimization 2:

Verilog code:
```
module counter_opt2 (input clk, input reset, output q);
	reg [2:0] count;
	assign q = (count[2:0] == 3'b100);
	
	always @(posedge clk,posedge reset)
	begin
		if(reset)
			count <= 3'b000;
		else
			count <= count + 1;
	end
endmodule
```
Simulation:
```
iverilog counter_opt.v tb_counter_opt.v
./a.out
gtkwave tb_counter_opt.vcd
```
Commands for netlist:
```
1. yosys
2. read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
3. read_verilog counter_opt2.v
4. synth -top counter_opt2
5. dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
7. show
8. write_verilog -noattr counter_opt_net.v
```

![Screenshot from 2024-10-20 14-04-56](https://github.com/user-attachments/assets/233723bf-fbd6-4b38-bd7a-33eadf112094)

![Screenshot from 2024-10-20 14-05-45](https://github.com/user-attachments/assets/06d71651-0967-4069-8f51-6f0606e2456d)
 
</li>

</li>

  </details>

  <details>
	  <summary>Day 4:</summary>
<li>
	Design of 2x1 MUX using Ternary Operator:
	
```
//Design
module ternary_operator_mux(input i0, input i1, input sel, output y);
	assign y = sel?i1:i0;
endmodule
```

```
iverilog ternary_operator_mux.v tb_ternary_operator_mux.v
./a.out
gtkwave tb_ternary_operator_mux.vcd
```

These commands perform iverilog and GTKWave simulation.

![Screenshot from 2024-10-20 14-47-41](https://github.com/user-attachments/assets/135af422-39af-4181-96cb-4cd570be26cb)

![Screenshot from 2024-10-20 14-48-07](https://github.com/user-attachments/assets/d9311642-5568-4b86-b387-06313afb4866)

```
1. yosys
2. read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
3. read_verilog ternary_operator_mux.v
4. synth -top ternary_operator_mux
5. abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
6. opt_clean -purge
7. write_verilog -noattr ternary_operator_mux_net.v
8. !gvim ternary_operator_mux_net.v
9. show
```

```
//Generated Netlist
module ternary_operator_mux(i0, il, sel, y);
	wire _0_;
	wire _1_;
	wire _2_;
	wire _3_;
	input i0; wire i0;
	input il; wire il;
	input sel; wire sel;
	output y; wire y;
	
	sky130_fd_sc_hd_mux2_1 _4_ (.AO(_0_),.A1(_1_),.S(_2_),.X(_3_));

	assign _0_ = i0;
	assign _1_ = il;
	assign _2_ = sel;
	assign y = _3_;
endmodule
```

![Screenshot from 2024-10-20 14-51-09](https://github.com/user-attachments/assets/ee242f1e-b4be-4490-9eec-c41c64a52603)

```
iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v ternary_operator_mux.v tb_ternary_operator_mux.v
./a.out
gtkwave tb_ternary_operator_mux.vcd
```

![Screenshot from 2024-10-20 14-52-00](https://github.com/user-attachments/assets/24ec3204-b9b1-401a-8923-bf9d3bab757c)

![Screenshot from 2024-10-20 15-13-10](https://github.com/user-attachments/assets/84d3783d-83bb-41de-bba5-9d47657ef1dd)

These waveforms correspond to the GATE LEVEL SYNTHESIS for the Ternary Operator MUX.

</li>	

<li>
	Design of a Bad 2x1 MUX:

```
//Design
module bad_mux(input i0, input i1, input sel, output reg y);
	always@(sel)
	begin
		if(sel)
			y <= i1;
		else
			y <= i0;
	end
endmodule
```

```
iverilog bad_mux.v tb_bad_mux.v
./a.out
gtkwave tb_bad_mux.vcd
```

![Screenshot from 2024-10-20 15-03-49](https://github.com/user-attachments/assets/aee93f19-1fe0-4271-93a5-86bbada7acd9)
![Screenshot from 2024-10-20 15-03-22](https://github.com/user-attachments/assets/009ae054-238f-41af-82db-5541626878ac)

From the waveform it can be observed that the output y changes only when there is a change in select line, completely ignoring the change in i0 and i1, which should also change the output y. Thus, this design is that of a bad MUX.

```
1. yosys
2. read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
3. read_verilog bad_mux.v
4. synth -top bad_mux
5. abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
6. opt_clean -purge
7. write_verilog -noattr bad_mux_net.v
8. !gvim bad_mux_net.v
9. show
```

```
//Generated Netlist
module bad_mux(i0, il, sel, y);
	wire _0_;
	wire _1_;
	wire _2_;
	wire _3_;
	input i0; wire i0;
	input il; wire il;
	input sel; wire sel;
	output y; wire y;
	
	sky130_fd_sc_hd_mux2_1 _4_ (.AO(_0_),.A1(_1_),.S(_2_),.X(_3_));

	assign _0_ = i0;
	assign _1_ = il;
	assign _2_ = sel;
	assign y = _3_;
endmodule
```

![Screenshot from 2024-10-20 15-07-21](https://github.com/user-attachments/assets/acf4336d-398d-4366-883f-ae9d121bd55f)

```
iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v bad_mux.v tb_bad_mux.v
./a.out
gtkwave tb_bad_mux.vcd
```

![Screenshot from 2024-10-20 15-49-52](https://github.com/user-attachments/assets/013350b6-9761-41b4-a0e4-245e15dc4418)

![Screenshot from 2024-10-20 15-09-13](https://github.com/user-attachments/assets/850e946b-813e-4038-af0b-714b38a46173)

These waveforms correspond to the GATE LEVEL SYNTHESIS for the Bad MUX.

</li>

<li>
	Blocking Caveat:

```
//Design
module blocking_caveat(input a, input b, input c, output reg d);
	reg x;

	always@(*)
	begin
		d = x & c;
		x = a | b;
	end
endmodule
```

```
iverilog blocking_caveat.v tb_blocking_caveat.v
./a.out
gtkwave tb_blocking_caveat.vcd
```

![Screenshot from 2024-10-20 15-57-26](https://github.com/user-attachments/assets/1c3acb0f-915f-4212-87db-6535cb8f8705)

![Screenshot from 2024-10-20 16-02-54](https://github.com/user-attachments/assets/fbbf0035-d8e2-4935-9be4-d9adb49efe21)

![Screenshot from 2024-10-20 16-02-47](https://github.com/user-attachments/assets/e3b2aa99-cd89-4a5c-b428-26d2f6880780)

As depicted in the waveform, when A and B go zero, the OR gate output should be zero (X equal to zero), and the AND gate output should also be zero (same as D output). But, the AND gate input of X takes the previous value of A|B equal to one, based on the design created by the blocking statement, hence the discrepancy in the output.

```
1. yosys
2. read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
3. read_verilog blocking_caveat.v
4. synth -top blocking_caveat
5. abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
6. opt_clean -purge
7. write_verilog -noattr blocking_caveat_net.v
8. !gvim blocking_caveat_net.v
9. show
```

```
//Generated Netlist
module blocking_caveat(a,b,c,d);
	wire _0_;
	wire _1_;
	wire _2_;
	wire _3_;
	wire _4_;
	input a; wire a;
	input b; wire b;
	input c; wire c;
	input d; wire d;
	output d; wire d;
	
	sky130_fd_sc_hd__o21a_1 _5_ (.A1(_2_),.A2(_1_),.B1(_3_),.X(_4_));

	assign _2_ = b;
	assign _1_ = a;
	assign _3_ = c;
	assign d = _4_;
endmodule
```

![Screenshot from 2024-10-20 16-07-43](https://github.com/user-attachments/assets/14860dcb-b550-4679-8ebf-512dbff3820f)
```
iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v blocking_caveat.v tb_blocking_caveat.v
./a.out
gtkwave tb_blocking_caveat.vcd
```

![Screenshot from 2024-10-20 16-04-20](https://github.com/user-attachments/assets/0a41a5f7-738c-448d-b88f-b548474cc14b)

![Screenshot from 2024-10-20 16-05-32](https://github.com/user-attachments/assets/3e9cde6d-af32-4c44-aaa6-4bb96397c244)
These waveforms correspond to the GATE LEVEL SYNTHESIS for the Blocking Caveat.

</li>

  </details>
</details>







