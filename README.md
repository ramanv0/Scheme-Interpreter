# Scheme-Interpreter
An interpreter for the Scheme language: A dialect of the Lisp family of programming languages.

## Implementation Overview
Here is a brief overview of each of the Read-Eval-Print Loop components of the interpreter:
- **Read**: This step parses user input (a string of Scheme code) into the interpreter's internal Python representation of Scheme expressions (e.g. Pairs).
  - *Lexical analysis* is implemented in the `tokenize_lines` function in `scheme_tokens.py`. This function returns a `Buffer` (from `buffer.py`) of tokens.
  - *Syntactic analysis* happens in `scheme_reader.py`, in the `scheme_read` and `read_tail` functions. Together, these mutually recursive functions parse Scheme tokens into the interpreter's internal Python representation of Scheme expressions.
- **Eval**: This step evaluates Scheme expressions (represented in Python) to obtain values. This step is implemented in the main `scheme.py` file.
  - *Eval* happens in the `scheme_eval` function. If the expression is a call expression, it gets evaluated according to the rules for evaluating call expressions. If the expression being evaluated is a special form, the corresponding `do_?_form` function is called (where `?` is replaced by the name of the special form).
  - *Apply* happens in the `scheme_apply` function. If the function is a built-in procedure, `scheme_apply` calls the `apply` method of that `BuiltInProcedure` instance. If the procedure is a user-defined procedure, `scheme_apply` creates a new call frame and calls `eval_all` on the body of the procedure, resulting in a mutually recursive eval-apply loop.
- **Print**: This step prints the `__str__` representation of the obtained value.
- **Loop**: The logic for the loop is handled by the `read_eval_print_loop` function in `scheme.py`.

## Running the Interpreter
To start an interactive Scheme interpreter session, type:

    python3 scheme.py

The Scheme interpreter can be used to evaluate expressions in an input file by passing the file name as a command-line argument to `scheme.py`:

    python3 scheme.py tests.scm

To exit the Scheme interpreter, press `Ctrl-d` or evaluate the `exit` procedure:

    scm> (exit)
