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

## The Reader
The reader parses Scheme code into Python values with the following representations:

| Input Example | Scheme Expression Type | Internal Representation |
| :------------ | :--------------------- | :---------------------- |
| `scm> 1` | Numbers | Python's built-in `int` and `float` values |
| `scm> x` | Symbols | Python's built-in `string` values |
| `scm> #t` | Booleans (`#t`, `#f`) | Python's built-in `True`, `False` values |
| `scm> (+ 2 3)` | Combinations | Instances of the `Pair` class, defined in `scheme_reader.py` |
| `scm> nil` | `nil` | The `nil` object, defined in `scheme_reader.py` |

The reader is implemented such that tokens are stored ready to be parsed in `Buffer` instances. For example, a buffer containing the input `(+ (2 3))` would have the tokens `'('`, `'+'`, `'('`, `2`, `3`, `')'`, and `')'`. See the doctests in `buffer.py` for more examples.

The parsing functionality consists of two mutually recursive functions: `scheme_read` and `read_tail`. These functions each take in a single parameter, `src`, which is an instance of `Buffer`.

## The Evaluator
The evaluator consists of the following components:
- `scheme_eval` evaluates a Scheme expression in the given environment.
  - When evaluating a special form, `scheme_eval` redirects evaluation to an appropriate `do_?_form` function found in the Special Forms section in `scheme.py`.
- `scheme_apply` applies a procedure to some arguments.
- The `.apply` methods in subclasses of `Procedure` and the `make_call_frame` function assist in applying built-in and user-defined procedures.
- The `Frame` class implements an environment frame.
- The `LambdaProcedure` class (in the Procedures section) represents user-defined procedures.

These are all of the essential components of the interpreter; the rest of `scheme.py` defines special forms and input/output behavior.

## Running the Interpreter
To start an interactive Scheme interpreter session, type:

    python3 scheme.py

The Scheme interpreter can be used to evaluate expressions in an input file by passing the file name as a command-line argument to `scheme.py`:

    python3 scheme.py tests.scm

To exit the Scheme interpreter, press `Ctrl-d` or evaluate the `exit` procedure:

    scm> (exit)
