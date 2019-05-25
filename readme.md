# Orca Microcode

This is a proposal for a language to define the behaviour of operators in [Orca](https://github.com/hundredrabbits/Orca). 
The motivation behind this is to decouple the specification of the instruction set from the implementation of the virtual machine (i.e. 
[Orca](https://github.com/hundredrabbits/Orca),
[Orca-c](http://github.com/hundredrabbits/Orca-c), and
[Orca Norns](https://github.com/itsyourbedtime/orca/)
).
With this modularity, instruction sets can be distributed as a single text file.
It enables all implementations to easily stay up-to-date with the latest version of Orca's instruction set, and makes possible the creation and distribution of custom instruction sets.
To utilise this, the implementation has to parse the microcode file and bring the instructions in it into a form it can execute. The parsing can be done once during initialisation, or at compile time (for compiled languages).

Feedback and pull-requests welcome ^_^

## rough specification

- `O { expression }` defines an operator, where _O_ has to be a single character
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
- the rest is pretty standard B syntax \*waves hand\*

## example

```
A { [0,1] = [-1,0] + [1,0]; }

B { r = [-1,0] ? [-1,0] : 1;
    m = ([1,0] ? [1,0] : 8) -1;
    k = (frame / r) % (m * 2);
    [0,1] = k <= m ? k : m - (k - m); }

C { r = [-1,0] ? [-1,0] : 1;
    m = [1,0] ? [1,0] : 8;
    [0,1] = (frame / r) % m; }

D { r = [-1,0] ? [-1,0] : 1;
    m = [1,0] ? [1,0] : 8;
    [0,1] = 0 == (frame / r) % m; }

E { @[0,0] = @[1,0] != '.';
    if (!@[0,0]) @[1,0] = 'E'; }

F { [0,1] = [-1,0] == [1,0]; }

G { x = [-3,0];
    y = [-2,0] + 1;
    n = [-1,0];
    for (i=0; i<n; i++) {
        [x+i, y] = [i+1, 0];
    } }

H { h = [0,1]; }

I { s = [-1,0] ? [-1,0] : 1;
    m = [1,0] ? [1,0] : 36;
    [0,1] = ([0,1] + s) % m; }

J { [0,1] = [0,-1]; }

K { n = [-1,0];
    for (i=1; i<=n; i++) {
        if ([i,0]) [i,1] = vars[[i,0]];
    } } 

L { s = [-2,0] ? [-2,0] : 1;
    n = [-1,0];
    for (i=0; i<n; i++) { l[i] = [i+1, 0]; }
    for (i=0; i<n; i++) { [i+1, 0] = l[(i+s)%n]; } }

M { [0,1] = [-1,0] * [1,0]; }

N { @[0,0] = @[0,-1] != '.';
    if (!@[0,0]) @[0,-1] = 'N'; }

O { x = [-2,0] + 1;
    y = [-1,0];
    [0,1] = @[x,y]; }

```