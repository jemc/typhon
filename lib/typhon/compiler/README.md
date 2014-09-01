Typhon Compiler
===============

This directory contains a Python compiler implemented in ruby
using the Rubinius compiler chain.

Compiling Python
================

The main interface to the Typhon compiler is by using the methods implemented
in `compiler.rb`. `Typhon::Compiler` exposes functions to compile .py
files used as modules and python code string intended to be
evaled. The compiler is implemented in stages.

Stages
======

Compiler stages are implemented in `stages.rb`.

## Parsing ##

The first step is to turn your python program (a string representing
valid python code) into a machine-representable format. This is called
_parsing_, and in current Typhon it's implemented by Python's own
parsing module. The `bin/pyparse.py` script is a tiny Python program
that simply outputs a _sexp_ representation of the source code. The
_sexp_ is made of only basic ruby literals, that is arrays, symbols,
strings, numbers and nil.

`parser.rb` exposes a simple `Typhon::Parser` module that just opens a
pipe to the `pyparse.py` program and returns the evaluated _sexp_.

This stage is implemented by `PyFile` and `PyCode` that essentially
just use `Typhon::Parser`.

## AST ##

The next step is turn the _sexp_ (an array of ruby literals) into
actual AST Nodes. An AST Node is a more logical representation of
what's being done in your program, for example, there's an AST Node
for every string literal, for import statements, for method
invocation, etc.

Once the AST (Abstract Syntax Tree) is completely built, we are ready
to proceed with compilation.

This stage is implemented by `PyAST`

## AST Transformations ##

The next stage is to perform AST transformations, adapting the AST
tree for better suitting the purposes of python semantics.

All python programs are represented by a module object in python,
so the result of parsing any valid python code is a tree
having a root node of type ModuleNode. However, in some cases, like
evaling a python expession with the _eval_ builtin function, we just
don't need nor want to create a new module for the code being
evaluated.  Instead, we want the code to be evaled on the current
context. For this end, Typhon has a `EvalExpr` stage that performs a
very simple transformation for an AST tree intended for eval.

In case of an eval, the `EvalExpr` stage simply removes the top
`ModuleNode` and makes sure the last expression returns a value
(unwraps any final `DiscardNode`).

If the AST tree was not intended for evaling, this stage does nothing.

## Bytecode Generator ##

This stage is where actual compilation takes place, it's implemented
in the `Generator` class, which calls the `bytecode` method on the
root node of the AST tree. Each AST node is responsible for producing
the Rubinius bytecode for it's own behavior.

See the _ast_ directory for the implementaion of all AST nodes.

## Rest of Rubinius compiler chain ##

After the `Generator` stage, all we do is feed the produced generator
to Rubinius Encoder. So we rely on Rubinius doing bytecode analysis
and validation, also on writing compiled .rbc files to the file system
or producing an EvalExpression object for evals.
