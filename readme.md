# Orca Microcode

This is a proposal for a language to define the behaviour of operators in [Orca](https://github.com/hundredrabbits/Orca). 
The motivation behind this is to decouple the specification of the instruction set from the implementation of the virtual machine (i.e. 
[Orca](https://github.com/hundredrabbits/Orca),
[Orca-c](http://github.com/hundredrabbits/Orca-c), and
[Orca Norns](https://github.com/itsyourbedtime/orca/)).

With this modularity, instruction sets can be distributed as a single text file.
It enables all implementations to easily stay up-to-date with the latest version of Orca's instruction set, and makes possible the creation and distribution of custom instruction sets.

To utilise this, the implementation has to parse the microcode file and bring the instructions in it into a form it can execute. The parsing can be done once during initialisation, or at compile time (for compiled languages).

feedback and pull-requests welcome ^\_^

## specification

- `[x,y]` addresses a cell, _x_ and _y_ being relative offsets
- assigning to a cell marks it as output port, and locks it
- reading from a cell marks it as input port, and locks it
- `@[x,y]` addresses a cell without locking it
- the data types are: **integer**, **boolean**, and **character**
- integer assignments to a cell are in **mod 36**
- boolean assingments to a cell are `*` (true) or `.` (false)
- character assignments are the character itself
- local variables don't have to be declared, type is inferred
- implicit conversion from character (in base36) to integer
- some global variables exist, such as **frame** and the array **vars[]**
- `O { definition }` defines an operator, where _O_ has to be a single character
- a _definition_ is composed of:
    - `op { statement; statement; ... }` (the actual behaviour of the operator)
    - _\*optional\*_ `name "text"`
    - _\*optional\*_ `info "text"`
    - _\*optional\*_ `labels { [x,y]: "text", [x,y]: "text", ... }`
- the rest is pretty standard B syntax \*waves hand\*

## example

```
A { name "add"
    info "Outputs sum of inputs"
    op { [0,1] = [-1,0] + [1,0]; } }

B { name "bounce"
    info "Outputs values between inputs"
    op { r = [-1,0] ? [-1,0] : 1;
         m = ([1,0] ? [1,0] : 8) -1;
         k = (frame / r) % (m * 2);
         [0,1] = k <= m ? k : m - (k - m); }
    labels { [-1,0]: "rate", [1,0]: "mod" } }

C { name "clock"
    info "Outputs modulo of frame"
    op { r = [-1,0] ? [-1,0] : 1;
         m = [1,0] ? [1,0] : 8;
         [0,1] = (frame / r) % m; }
    labels { [-1,0]: "rate", [1,0]: "mod } }

D { name "delay"
    info "Bangs on modulo of frame"
    op { r = [-1,0] ? [-1,0] : 1;
         m = [1,0] ? [1,0] : 8;
         [0,1] = 0 == (frame / r) % m; }
    labels { [-1,0]: "rate", [1,0]: "mod" } }

E { name "east"
    info "Moves eastward, or bangs"
    op { @[0,0] = @[1,0] != '.';
         if (!@[0,0]) @[1,0] = 'E'; } }

F { name "if"
    info "Bangs if inputs are equal"
    op { [0,1] = [-1,0] == [1,0]; } }

G { name "generator"
    info "Writes operands with offset"
    op { x = [-3,0];
         y = [-2,0] + 1;
         n = [-1,0];
         for (i=0; i<n; i++) {
             [x+i, y] = [i+1, 0];
         } }
    labels { [-3,0]: "x", [-2,0]: "y", [-1,0]:"len" } }

H { name "halt"
    info "Halts southward operand"
    op { h = [0,1]; } }

I { name "increment"
    info "Increments southward operand"
    op { s = [-1,0] ? [-1,0] : 1;
         m = [1,0] ? [1,0] : 36;
         [0,1] = ([0,1] + s) % m; }
    labels { [-1,0]: "step", [1,0]: "mod" } }

J { name "jump"
    info "Outputs northward operand"
    op { [0,1] = [0,-1]; } }

K { name "konkat"
    info "Reads multiple variables"
    op { n = [-1,0];
         for (i=1; i<=n; i++) {
             if ([i,0]) [i,1] = vars[[i,0]];
         } }
    labels { [-1,0]:"len" } }

L { name "loop"
    info "Moves eastward operands"
    op { s = [-2,0] ? [-2,0] : 1;
         n = [-1,0];
         for (i=0; i<n; i++) { l[i] = [i+1, 0]; }
         for (i=0; i<n; i++) { [i+1, 0] = l[(i+s)%n]; } }
    labels { [-2,0]: "step", [-1,0]: "len" } }

M { name "multiply"
    info "Outputs product of inputs"
    op { [0,1] = [-1,0] * [1,0]; } }

N { name "north"
    info "Moves Northward, or bangs"
    op { @[0,0] = @[0,-1] != '.';
         if (!@[0,0]) @[0,-1] = 'N'; } }

O { name "read"
    info "Reads operand with offset"
    op { x = [-2,0] + 1;
         y = [-1,0];
         [0,1] = @[x,y]; }
    labels { [-2,0]: "x", [-1,0]: "y" } }
```
