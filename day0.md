hw->spice netlist
verilog inspired from c 
chisel, blespec->verilog: advanced, design with higher abstraction.
syntheieze in diff tech nodes
verilog runs in parallel/concurrent. processing power increases.
register tr level

diff abstraction
1. behavior level: algorithms
2. rtl: ff connections, nets and registers
3. gate: logic gates
4. switch level: built in primitives

behavioral level

module mul(input a, output b);
assign b=a*2;
endmodule

RTL

module mul(input a, output b);
assign b=a<<2;
endmodule

GATELEVEL: used built in primitives, and gate

module mul(input a, output b);
and m1(y,a,b);
endmodule


ieee std 1364-2005 version

syntax for module

#adder

ports: after processiing comes out:output port, inout port:bidirectional


module adder(input a, input b, output s);
	assign s=a+b;
endmodule

module adder(a,b,s);
	input a; input b; output s;
	assign s=a+b;
endmodule


--
identifiers
$ escape sequence

legal: uni_3
	ab$
	/fgfg

----


assign: rhs to lhs
default: all ports are of type wire of high impedance default value
assign: for continous assignment

4 logic values
logic 0: 0
logic 1:	1
z: high imp   default value
x: dont care, unknown

fa: top module
ha: sub module

ha
s=a^b
c=a*b

define intermediate wires of data type:wire, net


in:a,b
out: s,c

module ha(a,b,s,c);
input a,b;
output s,c;
assign s= a^b;
assign c=a&&b;//for 1 bit
assign c=a&b; // for contin
endmodule

continous drive, hence we use 


module fa(input a,b,c, output s,c,cout);
	wire w1,w2,w3;
	ha H1(.a(a), .b(b), .c(w2), .s(w1));//named instantiation
	ha H2(w2,c,s,w3);orderd instantiationoutput, input

	or (c,w2,w3);// naming for primitive not mandatory
endmodule




ripple carry adder

module rpc(a0,a1,a2,b0,b1,b2,cin,cout,s0,s1,s2);
	input a0,a1,a2,b0,b1,b2, cin;
	output s0,s1,s2,cout;
	wire c0,c1;

	fa f1(s0,c0,a0,b0,cin);
	fa f2(s1,c1,a1,b1,co);
	fa f3(s2,cout,a2,b2,c1);

endmodule


module rpc(a,b,s,cout,cin);
	input a[2:0],b[2:0], cin;
	output s[2:0], cout;
        wire c[1:0];

	fa f1(s[0],c[0],a[0],b[0],cin);
	fa f2(s[1],c[1],a[1],b[1],c[o]);
	fa f3(s[2],cout,a[2],b[2],c[1]);
endmodule



dont use randowm variables as extra wire causes error during implementation
instantioation order doesnt matter coz of concurrent order
follow named instantiation,


Data types

1.nets:
triwire, 

multi-driven net errors: 
any output port u assign two values
reg : unknown value 
how to prevent: use uwire
define this way: 'default nettype uwire'



vectors:
a[3:0]
little indian format: lsb lower index, msb :3,2,1,0
big indian: lsb= higher index [0:3]a :0,1,2,3



<width>'<radix value>
4'b1111;


Highr bit assigned to lower
wire [3:0]a;
wire [6:0]b=7'hFA;
assign a=b;

output: A


lower bit assigned to higher
wire [3:0]a= 4'hf;
wire [6:0]b;
assign a=b;

output: 0001111 (gets 0)

right to left
high to low will be truncated

can we mix little indian and big indian format





a[2:0] unpacked array used for memory, dont use, column wise
[2:0]a packed array, row wise

takeaway: 
4:1 using 2 2:1 1in rtl, other in gate
vctor reversal










