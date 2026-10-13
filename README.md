# Mini Compiler and VM

A small C++17 project that compiles a C-like language into custom assembly,
loads it into a virtual machine, and lets you execute or debug it.

The pipeline is: source → lexer/tokenizer → abstract syntax tree → assembly
file → loader → virtual CPU. The CPU has 16 registers and separate code,
data, and stack memory regions.

## Build and run

Requires Make and a C++17 compiler (`c++` by default).

```sh
make
```

This builds three executables:

| Executable | Purpose | Usage |
| --- | --- | --- |
| `Compiler` | Parses source code and generates custom assembly in `./exe`. | `./Compiler <source-file>` |
| `Executor` | Loads the generated assembly into virtual memory, runs it on the CPU, and displays diagnostics and final register values. | `./Executor <assembly-file>` |
| `Debugger` | Loads the same assembly and opens an interactive prompt for stepping through instructions and inspecting CPU state and memory. | `./Debugger <assembly-file>` |

Save this example as `example.mc`:

```text
Func int main()
{
    int a = 10;
    int b = (a + 7) * 2;
    return b;
}
```

```sh
./Compiler example.mc
./Executor exe
./Debugger exe
```

The compiler writes a text file named `exe` in the current directory,
overwriting any previous output. It contains a `~AnushFile` header, a function
symbol table, and a `.CODE` section. The executor starts at `main` and prints
loading diagnostics and the final register values; the example computes `34`.

## Debugger commands

Run `./Debugger exe`, then enter commands at the `(dbg)` prompt. Debugging
operates on assembly instructions. In the table, `N` is a numeric index,
`address` is a memory address, and square brackets mark optional arguments.

| Command | Alias | What it does |
| --- | --- | --- |
| `step` | `s` | Executes one instruction and shows the next instruction. |
| `next` | `n` | Performs the same action as `step`, including entering function calls. |
| `continue` | `c` | Runs until the program reaches `EXIT`. |
| `list [N]` | `l [N]` | Shows up to five instructions before and after index `N`, or the current instruction if omitted. Marks the current instruction with `>`. |
| `regs` | `r` | Shows nonzero registers, the instruction pointer, stack pointer, and comparison flag. |
| `print rN` | `p rN` | Shows register `N` (0–15), for example `print r10`. |
| `print #N` | `p #N` | Shows constant-pool entry `N`, for example `print #0`. |
| `print mem address` | `p mem address` | Shows a raw memory cell in decimal and hexadecimal. |
| `mem address` | `m address` | Performs the same action as `print mem address`; accepts decimal or hexadecimal addresses, such as `mem 0x2000`. |
| `info` | `i` | Shows the instruction pointer (`IP`), stack pointer (`SP`), and comparison flag (`cmpFlag`). |
| `info regs` | `i regs` | Shows the same state as `regs`; `registers` and `r` are also accepted as arguments. |
| `help` | `h` | Displays the command help. |
| `quit` | `q` | Ends the debugger session. |

## Language and layout

The language includes `Func` declarations, `int` and `void` return types,
local variables, assignment, arithmetic (`+`, `-`, `*`, `/`), parentheses,
function calls, and `return`. Statements end with semicolons; blocks use braces.
The parser and code generator also contain experimental `if`/`else` and
`while` support.

- `src/`: compiler stages, code generator (`VM.cpp`), loader, CPU, and debugger.
- `includes/`: syntax tree nodes, symbol table, and runtime interfaces.
- `code`: a larger sample source program.
- `Makefile`: build targets; `make clean` removes object files and
  `make fclean` also removes the three executables.

This is a work in progress: comparison parsing and function argument passing
need fixes, and generated register indices are not checked against the CPU's
16-register limit. Keep examples small. Comments, strings, and floating-point
values are not supported.
