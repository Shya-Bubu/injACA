# EE2030 — FPGA Lab 2 Solutions

**Group Number:** 5

---

## Task 1: Control Each LED by the Switch Directly Below It

### Source Code — `task1.v`

```verilog
module task1(
    input  [15:0] sw,
    output [15:0] led
);

    assign led = sw;

endmodule
```

### Constraint File — `task1.xdc`

```tcl
## Switches
set_property -dict { PACKAGE_PIN V17   IOSTANDARD LVCMOS33 } [get_ports {sw[0]}]
set_property -dict { PACKAGE_PIN V16   IOSTANDARD LVCMOS33 } [get_ports {sw[1]}]
set_property -dict { PACKAGE_PIN W16   IOSTANDARD LVCMOS33 } [get_ports {sw[2]}]
set_property -dict { PACKAGE_PIN W17   IOSTANDARD LVCMOS33 } [get_ports {sw[3]}]
set_property -dict { PACKAGE_PIN W15   IOSTANDARD LVCMOS33 } [get_ports {sw[4]}]
set_property -dict { PACKAGE_PIN V15   IOSTANDARD LVCMOS33 } [get_ports {sw[5]}]
set_property -dict { PACKAGE_PIN W14   IOSTANDARD LVCMOS33 } [get_ports {sw[6]}]
set_property -dict { PACKAGE_PIN W13   IOSTANDARD LVCMOS33 } [get_ports {sw[7]}]
set_property -dict { PACKAGE_PIN V2    IOSTANDARD LVCMOS33 } [get_ports {sw[8]}]
set_property -dict { PACKAGE_PIN T3    IOSTANDARD LVCMOS33 } [get_ports {sw[9]}]
set_property -dict { PACKAGE_PIN T2    IOSTANDARD LVCMOS33 } [get_ports {sw[10]}]
set_property -dict { PACKAGE_PIN R3    IOSTANDARD LVCMOS33 } [get_ports {sw[11]}]
set_property -dict { PACKAGE_PIN W2    IOSTANDARD LVCMOS33 } [get_ports {sw[12]}]
set_property -dict { PACKAGE_PIN U1    IOSTANDARD LVCMOS33 } [get_ports {sw[13]}]
set_property -dict { PACKAGE_PIN T1    IOSTANDARD LVCMOS33 } [get_ports {sw[14]}]
set_property -dict { PACKAGE_PIN R2    IOSTANDARD LVCMOS33 } [get_ports {sw[15]}]

## LEDs
set_property -dict { PACKAGE_PIN U16   IOSTANDARD LVCMOS33 } [get_ports {led[0]}]
set_property -dict { PACKAGE_PIN E19   IOSTANDARD LVCMOS33 } [get_ports {led[1]}]
set_property -dict { PACKAGE_PIN U19   IOSTANDARD LVCMOS33 } [get_ports {led[2]}]
set_property -dict { PACKAGE_PIN V19   IOSTANDARD LVCMOS33 } [get_ports {led[3]}]
set_property -dict { PACKAGE_PIN W18   IOSTANDARD LVCMOS33 } [get_ports {led[4]}]
set_property -dict { PACKAGE_PIN U15   IOSTANDARD LVCMOS33 } [get_ports {led[5]}]
set_property -dict { PACKAGE_PIN U14   IOSTANDARD LVCMOS33 } [get_ports {led[6]}]
set_property -dict { PACKAGE_PIN V14   IOSTANDARD LVCMOS33 } [get_ports {led[7]}]
set_property -dict { PACKAGE_PIN V13   IOSTANDARD LVCMOS33 } [get_ports {led[8]}]
set_property -dict { PACKAGE_PIN V3    IOSTANDARD LVCMOS33 } [get_ports {led[9]}]
set_property -dict { PACKAGE_PIN W3    IOSTANDARD LVCMOS33 } [get_ports {led[10]}]
set_property -dict { PACKAGE_PIN U3    IOSTANDARD LVCMOS33 } [get_ports {led[11]}]
set_property -dict { PACKAGE_PIN P3    IOSTANDARD LVCMOS33 } [get_ports {led[12]}]
set_property -dict { PACKAGE_PIN N3    IOSTANDARD LVCMOS33 } [get_ports {led[13]}]
set_property -dict { PACKAGE_PIN P1    IOSTANDARD LVCMOS33 } [get_ports {led[14]}]
set_property -dict { PACKAGE_PIN L1    IOSTANDARD LVCMOS33 } [get_ports {led[15]}]
```

---

## Task 2: Gate-Level Circuit Implementation

**Group Number:** 5 → k = mod(5 / 6) = **5** → Gate (a) = **NOR**, Gate (b) = **AND**

### Circuit Description

- Inputs **A** and **B** feed into gate **(b)** = AND gate
- Input **C** goes through a **NOT** gate (inverter)
- The output of the AND gate and the inverted C feed into gate **(a)** = NOR gate
- Output: **Z = ~( (A & B) | ~C )**

### Truth Table

| A | B | C | AND(A,B) | NOT(C) | Z = NOR(AND(A,B), NOT(C)) |
|---|---|---|----------|--------|---------------------------|
| 0 | 0 | 0 |    0     |   1    |             0             |
| 0 | 0 | 1 |    0     |   0    |             1             |
| 0 | 1 | 0 |    0     |   1    |             0             |
| 0 | 1 | 1 |    0     |   0    |             1             |
| 1 | 0 | 0 |    0     |   1    |             0             |
| 1 | 0 | 1 |    0     |   0    |             1             |
| 1 | 1 | 0 |    1     |   1    |             0             |
| 1 | 1 | 1 |    1     |   0    |             0             |

### Source Code — `task2.v`

```verilog
module task2(
    input  A,
    input  B,
    input  C,
    output Z
);

    wire and_out;
    wire not_c;

    // Gate (b): AND gate with inputs A and B
    and g_b(and_out, A, B);

    // Inverter on input C
    not g_inv(not_c, C);

    // Gate (a): NOR gate with inputs and_out and not_c
    nor g_a(Z, and_out, not_c);

endmodule
```

### Constraint File — `task2.xdc`

```tcl
## Switches as inputs
set_property -dict { PACKAGE_PIN V17   IOSTANDARD LVCMOS33 } [get_ports {A}]
set_property -dict { PACKAGE_PIN V16   IOSTANDARD LVCMOS33 } [get_ports {B}]
set_property -dict { PACKAGE_PIN W16   IOSTANDARD LVCMOS33 } [get_ports {C}]

## LED as output
set_property -dict { PACKAGE_PIN U16   IOSTANDARD LVCMOS33 } [get_ports {Z}]
```

---

## Task 3: 1-Bit Full Adder (Gate Level)

### Source Code — `task3.v`

```verilog
module full_adder_1bit(
    input  A,
    input  B,
    input  Cin,
    output Sum,
    output Cout
);

    wire axb;
    wire a_and_b;
    wire axb_and_cin;

    // Sum = A XOR B XOR Cin
    xor g1(axb, A, B);
    xor g2(Sum, axb, Cin);

    // Cout = (A AND B) OR ((A XOR B) AND Cin)
    and g3(a_and_b, A, B);
    and g4(axb_and_cin, axb, Cin);
    or  g5(Cout, a_and_b, axb_and_cin);

endmodule
```

### Constraint File — `task3.xdc`

```tcl
## Switches as inputs
set_property -dict { PACKAGE_PIN V17   IOSTANDARD LVCMOS33 } [get_ports {A}]
set_property -dict { PACKAGE_PIN V16   IOSTANDARD LVCMOS33 } [get_ports {B}]
set_property -dict { PACKAGE_PIN W16   IOSTANDARD LVCMOS33 } [get_ports {Cin}]

## LEDs as outputs
set_property -dict { PACKAGE_PIN U16   IOSTANDARD LVCMOS33 } [get_ports {Sum}]
set_property -dict { PACKAGE_PIN E19   IOSTANDARD LVCMOS33 } [get_ports {Cout}]
```

---

## Task 4: 4-Bit Full Adder (Using 1-Bit Full Adder Submodules)

### Source Code — `task4.v`

```verilog
module full_adder_1bit(
    input  A,
    input  B,
    input  Cin,
    output Sum,
    output Cout
);

    wire axb;
    wire a_and_b;
    wire axb_and_cin;

    xor g1(axb, A, B);
    xor g2(Sum, axb, Cin);

    and g3(a_and_b, A, B);
    and g4(axb_and_cin, axb, Cin);
    or  g5(Cout, a_and_b, axb_and_cin);

endmodule


module full_adder_4bit(
    input  [3:0] A,
    input  [3:0] B,
    input        Cin,
    output [3:0] Sum,
    output       Cout
);

    wire c1, c2, c3;

    full_adder_1bit FA0(.A(A[0]), .B(B[0]), .Cin(Cin), .Sum(Sum[0]), .Cout(c1));
    full_adder_1bit FA1(.A(A[1]), .B(B[1]), .Cin(c1),  .Sum(Sum[1]), .Cout(c2));
    full_adder_1bit FA2(.A(A[2]), .B(B[2]), .Cin(c2),  .Sum(Sum[2]), .Cout(c3));
    full_adder_1bit FA3(.A(A[3]), .B(B[3]), .Cin(c3),  .Sum(Sum[3]), .Cout(Cout));

endmodule
```

### Constraint File — `task4.xdc`

```tcl
## Switches for input A (sw[0] to sw[3])
set_property -dict { PACKAGE_PIN V17   IOSTANDARD LVCMOS33 } [get_ports {A[0]}]
set_property -dict { PACKAGE_PIN V16   IOSTANDARD LVCMOS33 } [get_ports {A[1]}]
set_property -dict { PACKAGE_PIN W16   IOSTANDARD LVCMOS33 } [get_ports {A[2]}]
set_property -dict { PACKAGE_PIN W17   IOSTANDARD LVCMOS33 } [get_ports {A[3]}]

## Switches for input B (sw[4] to sw[7])
set_property -dict { PACKAGE_PIN W15   IOSTANDARD LVCMOS33 } [get_ports {B[0]}]
set_property -dict { PACKAGE_PIN V15   IOSTANDARD LVCMOS33 } [get_ports {B[1]}]
set_property -dict { PACKAGE_PIN W14   IOSTANDARD LVCMOS33 } [get_ports {B[2]}]
set_property -dict { PACKAGE_PIN W13   IOSTANDARD LVCMOS33 } [get_ports {B[3]}]

## Switch for Carry-in (sw[8])
set_property -dict { PACKAGE_PIN V2    IOSTANDARD LVCMOS33 } [get_ports {Cin}]

## LEDs for Sum output (led[0] to led[3])
set_property -dict { PACKAGE_PIN U16   IOSTANDARD LVCMOS33 } [get_ports {Sum[0]}]
set_property -dict { PACKAGE_PIN E19   IOSTANDARD LVCMOS33 } [get_ports {Sum[1]}]
set_property -dict { PACKAGE_PIN U19   IOSTANDARD LVCMOS33 } [get_ports {Sum[2]}]
set_property -dict { PACKAGE_PIN V19   IOSTANDARD LVCMOS33 } [get_ports {Sum[3]}]

## LED for Carry-out (led[4])
set_property -dict { PACKAGE_PIN W18   IOSTANDARD LVCMOS33 } [get_ports {Cout}]
```

---

## Task 5: 3-Judge Majority Voting (Gate Level)

**Rule:** 3 judges, output is HIGH if **2 or more** judges vote YES (majority = 2/3).

**Boolean Expression:** `Z = (J0 & J1) | (J0 & J2) | (J1 & J2)`

### Source Code — `task5.v`

```verilog
module judge_majority(
    input  J0,
    input  J1,
    input  J2,
    output Z
);

    wire j0_and_j1;
    wire j0_and_j2;
    wire j1_and_j2;
    wire or_temp;

    // All pairwise AND combinations
    and g1(j0_and_j1, J0, J1);
    and g2(j0_and_j2, J0, J2);
    and g3(j1_and_j2, J1, J2);

    // OR all pairs together
    or  g4(or_temp, j0_and_j1, j0_and_j2);
    or  g5(Z, or_temp, j1_and_j2);

endmodule
```

### Constraint File — `task5.xdc`

```tcl
## Switches as Judge inputs
set_property -dict { PACKAGE_PIN V17   IOSTANDARD LVCMOS33 } [get_ports {J0}]
set_property -dict { PACKAGE_PIN V16   IOSTANDARD LVCMOS33 } [get_ports {J1}]
set_property -dict { PACKAGE_PIN W16   IOSTANDARD LVCMOS33 } [get_ports {J2}]

## LED as output
set_property -dict { PACKAGE_PIN U16   IOSTANDARD LVCMOS33 } [get_ports {Z}]
```
