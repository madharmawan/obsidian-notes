# A. Verilog
This note is just practice with the syntax of Verilog with explanations on syntax and other things.

## A Verilog Primer
Verilog is a HDL that can describe digital circuits with C-like syntax. It defines circuits at the RTL of abstraction. 

Modules are the building blocks of Verilog designs. They are a means of abstraction and encapsulation for your design. 

- Consists of a port declaration and Verilog code to implement the desired functionality

- Modules are created in Verilog file (.v) where the filename matches the module name. 

Ports allow communication between a module and its environment. Each port has a name and a type

- Input

- Output

- Inout

Ports for a module are declared in the port declaration

```
module full_adder (input x, input y, input cin, output s, output cout);
	// Verilog code here has access to inputs
	// Verilog code here has access to outputs
endmodule
```

Every Verilog design has a top-level module which sits at the highest level of the design hierarchy. The top-level module defines the I/O for the entire digital system. All the modules in your design reside inside the top-level module.


## Verilog Module Instantiation
Modules can be instantiated inside other modules. The syntax used is

```
<module_name> <instance_name> (.port0(wire), .port1(wire), ...)
```

For example, 

```
module top_level (input switch0, 
				  input switch1, 
				  input switch2, 
				  output LED0,
				  output LED1);
	full_adder add (
			.x(switch0), 
			.y(switch1),
			.cin(switch2),
			.s(LED0),
			.cout(LED1)
	);
endmodule
```

## Wire Nets
Wires are analogous to wires in a circuit you build by hand; they are used to transmit values between inputs and outputs. Declare wires before they are used. Wires can be scalar (1 bit), or they can be vectors

```
// 1-bit declaration a and b
wire a;
wire b; 

wire [7:0] d; // 8-bit wire declaration
wire [31:0] e; // 32-bit wire declaration
```

We can declare signals that are more than 1 bit wide in Verilog with this general syntax:

```
[MSB bit index : LSB bit index]
```

now we can imagine how it deals with type annotations:

```
module two_bit_adder (input [1:0] x, input [1:0] y, output [2:0] sum);
	wire [1:0] partial_sum;
	wire carry_out;
endmodule
```

## Gate Primitives
The following gate primitives exist in Verilog: and, or, xor, not, nand, nor, xnor. In general, the syntax is

```
<operator> (output, input1, input2) // two input gate
<operator> (output, input); // for not gate
```

For example, if we want to implement the Boolean equation $f = a + b$:

```
wire a, b, f;
or (f, a, b);
```

## Example: Full Adder
Here is a gate-level circuit schematic

![[A.1 fulladder.png]]

In Verilog:

```
module full_adder (input a, input b, input cin, output s, output cout);
	xor(s, a, b, cin);

	wire xor_a_b;
	wire cin_and;
	wire and_a_b;

	xor(xor_a_b, a, b);
	and(cin_and, cin, xor_a_b);
	and(and_a_b, a, b);
	or(cout, and_a_b, cin_and);
endmodule
```

## Behavioral Verilog
The full adder using structural Verilog can be a pain to write, but you will never have to write it that way. Behavioral Verilog constructs allow you to describe what you want a circuit to do at the RTL level of abstraction.

### Assign Statements
Wires can be assigned to logic equations, other wires, or operations performed on wires. This is accomplished using `assign` statement. The left argument of the assign statement must be a wire, and cannot be an input wire. The right argument of the assign statement can be any expression created from Verilog operators and wires

```
module copy (input a, output b);
	assign b = a;
endmodule
```

Verilog contains operators that can be used to perform arithmetic, form logic expressions, perform reductions/shifts, and check equality between signals

![[A.2 operators.png]]

![[A.3 operators2.png]]

![[A.4 operators3.png]]

So a behavioral Verilog version of the full adder might look like

```
module full_adder (input x, input y, input cin, output s, output cout);
	assign s = x ^ y ^ cin;
	assign cout = (a && b) || (cin && (a ^ b));
endmodule
```

It gets even better

```
wire operand1 [31:0];
wire operand2 [31:0];
wire result [32:0];
assign result = operand1 + operand2
```

which described the 32-bit adder, but didn't specify the architecture or logic. The Verilog synthesizer wil turn the generalized addder into a FPGA specific netlist or an ASIC gate-level netlist.

## Conditional Operator ?
The conditional operator allows us to define if-else logic in an assign statement

```
<condition> ? <expr_if_true> : <expr_if_false>
```

```
assign out = a > 10 ? 10 : a;
```

Example: output `0` if the two inputs are equal, output `1` if input `a` is less than input `b`, and output `2` if input `a` is greater than input `a`:

```
module equality_checker(input a [31:0], input b[31:0], output c [1:0]);
	assign c = a == B ? 2'd0 : (a < b ? 2'd1 : 2'd2);
endmodule
```

## Verilog Literals
Verilog defines a particular way of specifying literals: 

```
[bit width]'[radix][literal]
```

Radix can be d (decimal), h (hex), o (octal), b (binary). It is critical to always match bit widths for operators and module connections, do not ignore these warnings from the tools

```
2'd1 
16'hAD14
8'b01011010
```
### Macros
Macros are similar to those in C. `` `include``
is a preprocessor command to inject a Verilog source file at its location; often used to bring in a Verilog header file (.vh)

`` `define <Constant Name> `` is used to declare a synthesis-time constant; use these instead of putting "magic values" in RTL

`` `define <Macro function name>(ARGS) <Function body>`` is used to define a macro function that can generate RTL based on ARGS

`` `ifndef <NAME>, `define <NAME>, `endif`` is an include guard

Here is an example. In constants.vh:

```
`ifndef CONSTANTS // guard prevents header file from being included more than once
`define CONSTANTS

`define ADDR_BITS 16
`define NUM_WORDS 32
`define LOG2(x) \ 
	(x <= 2) ? 1 : \ 
	(x <= 4) ? 2 : \ 
	(x <= 8) ? 3 : \ 
	(x <= 16) ? 4 : \ 
	(x <= 32) ? 5 : \ 
	(x <= 64) ? 6 : \
	-1
`endif
```

Then in design.v:

```
`include "constants.vh"
module memory (input [`ADDR_BITS - 1:0] address, output [`LOG2(`NUM_WORDS) - 1:0] data);
	//implementation
endmodule
```

## Reg Nets
There are two types of nets: `wire` and `reg`

Reg nets are required whenever a net must preserve state (i.e. in an always block). Wires are used for structural verilog (to connect inputs to outputs) and for continuous assignmnet:

```
reg x;
```


