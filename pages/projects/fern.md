---
title: Fern
subtitle: Multi-stage optimizing compiler for a statically-typed imperative programming language.
date: 2024-11-01
draft: false
featured: true
---

Fern is a multi-stage optimizing compiler for a statically-typed imperative programming language.

This project exists primarily as a learning exercise in compiler design and implementation.

## Compilation Pipeline

The compiler follows a modern compilation pipeline with several optimization stages:

1. Source code is parsed into an Abstract Syntax Tree (AST)
2. A symbol table is generated to track variable and function scopes
3. The AST is converted to Three-Address Code (TAC) for optimization
4. Dead code elimination removes unreachable code paths
5. A Control Flow Graph is built for analysis
6. The program is converted to Static Single Assignment (SSA) form
7. LLVM Intermediate Representation is generated
8. Finally, LLVM compiles the IR to a native executable

## Language Features

The compiler supports core imperative programming constructs including static typing with `int`, `bool`, and `void` types, function definitions with typed parameters, arithmetic and logical operations, control flow statements, and recursion.

## Language Syntax

Fern uses a Lua-inspired syntax with static typing. Here's a quick overview:

Function definitions use the `def` keyword, followed by the function name and typed parameters. The return type comes after the parameter list:

```
def add(x: int, y: int): int
    return x + y
end
```

Variables must be declared with the `var` keyword and require explicit type annotations. They can be initialized at declaration or will be zero-initialized:

```
var x: int = 42    // initialized to 42
var y: int         // zero-initialized
var z: bool
```

Control flow uses `if-then-end` blocks with optional `else` clauses:

```
if x > y then
    return x
else 
    return y
end
```

### Example: Fibonacci Sequence

```
def fib(n: int): int
    if n <= 1 then
        return n
    end
    return fib(n - 1) + fib(n - 2)
end

def main(): int
    return fib(10)
end
```

## Source

The source code for Fern is available on [GitHub](https://github.com/kkestell/fern).

## License

Fern uses the [Zero-Clause BSD](https://opensource.org/license/0bsd/) license.
