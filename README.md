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
| :-----------: | :--------------------: | :---------------------: |
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

## Special Forms
A special form is an expression that follows special evaluation rules. This Scheme interpreter implements the following special forms:
- `define`:
  - The `define` special form in Scheme can be used to either assign a name to the value of a given expression or to create a procedure and bind it to a name:  

        scm> (define a (+ 2 3))  ; Binds the name a to the value of (+ 2 3)
        a
        scm> (define (foo x) x)  ; Creates a procedure and binds it to the name foo
        foo
        
  - The type of the first operand determines what is being defined:
    - If it is a symbol, e.g. `a`, then the expression is defining a name;
    - If it is a list, e.g. `(foo x)`, then the expression is defining a procedure.
  - Implemented by the `do_define_form` function.
- `quote`:
  - In Scheme, you can quote expressions in two ways: With the `quote` special form or with the symbol `'`. The `quote` special form returns its operand expression without evaluating it:

        scm> (quote hello)
        hello
        scm> '(cons 1 2)  ; Equivalent to (quote (cons 1 2))
        (cons 1 2)
  - `'<expr>` translates to `(quote <expr>)` by wrapping the expression following `'` (which is obtained by recursively calling `scheme_read`) into the `quote` special form. For example, `'bagel` is represented as `Pair('quote', Pair('bagel', nil))`. For another example, `'(1 2)` is represented as `Pair('quote', Pair(Pair(1, Pair(2, nil)), nil))`. This functionality is implemented by `scheme_read` in `scheme_reader.py`.
  - Implemented by the `do_quote_form` function.
- `begin`:
  - A `begin` expression is evaluated by evaluating all sub-expressions in order. The value of the `begin` expression is the value of the final sub-expression.

        scm> (begin (+ 2 3) (+ 5 6))
        11
        scm> (define x (begin (display 3) (newline) (+ 2 3)))
        3
        x
        scm> (+ x 3)
        8
        scm> (begin (print 3) '(+ 2 3))
        3
        (+ 2 3)
  - Implemented by the `do_begin_form` function, which calls `eval_all`.
  
- User-Defined Procedures:
  - User-defined procedures are represented as instances of the `LambdaProcedure` class. A `LambdaProcedure` instance has three instance attributes:
    - `formals` is a Scheme list of the formal parameters (symbols) that name the arguments of the procedure.
    - `body` is a Scheme list of expressions; the body of the procedure.
    - `env` is the environment in which the procedure was defined.
  - `lambda`:
    - Creates and returns a `LambdaProcedure` instance. 
    - In Scheme, it is legal to place more than one expression in the body of a procedure (there must be at least one expression). Hence, the body attribute of a `LambdaProcedure` instance is a Scheme list of body expressions. This list of expressions has the same semantics as if it were the body of a single `begin` special form.
    
  - `define`:
    - Using only the `lambda` special form, the interpreter is able to bind symbols to user-defined procedures in the following manner:

          scm> (define f (lambda (x) (* x 2)))
          f
      However, the `define` special form handles a shorthand form of defining named procedures:
      
          scm> (define (f x) (* x 2))
    - Implemented by the `do_define_form` function, by:
      - Using the variables `target` and `expressions` to find the defined function's name, formals, and body;
      - Creating a `LambdaProcedure` instance using the formals and body;
      - Binding the name to the `LambdaProcedure` instance.
      
- Logical Special Forms:
  - Includes `if`, `and`, `or`, and `cond`. These expressions are special because not all of their sub-expressions may be evaluated.
  - `if`:

        (if <predicate> <consequent> [alternative])
    - Evaluates `predicate`. If true, the `consequent` is evaluated and returned. Otherwise, the `alternative`, if it exists, is evaluated and returned (if no `alternative` is present in this case, the return value is undefined).
    - Implemented by the `do_if_form` function.
  - `and`:
    - A *short-circuiting* logical form. The interpreter evaluates each sub-expression from left to right, and if any of these evaluates to a false value, then `#f` is returned. Otherwise, it returns the value of the last sub-expression. If there are no sub-expressions in an `and` expression, it evaluates to `#t`.
    
          scm> (and)
          #t
          scm> (and 4 5 6)  ; all operands are true values
          6
          scm> (and 4 5 (+ 3 3))
          6
          scm> (and #t #f 42 (/ 1 0))  ; short-circuiting behavior of and
          #f
    - Implemented by the `do_and_form` function.
  - `or`:
    - Another *short-circuiting* logical form. The interpreter evaluates each sub-expression from left to right. If any sub-expression evaluates to a true value, returns that value. Otherwise, returns `#f`. If there are no sub-expressions in an `or` expression, it evaluates to `#f`.
    
          scm> (or)
          #f
          scm> (or 5 2 1)  ; 5 is a true value
          5
          scm> (or #f (- 1 1) 1)  ; 0 is a true value in Scheme
          0
          scm> (or 4 #t (/ 1 0))  ; short-circuiting behavior of or
          4
    - Implemented by the `do_or_form` function.
  - `cond`:
    - Returns the value of the first result sub-expression corresponding to a true predicate, or the result sub-expression corresponding to `else`. Some special cases:
      - When the true predicate does not have a corresponding result sub-expression, returns the predicate value.
      - When a result sub-expression of a `cond` case has multiple expressions, evaluates them all and returns the value of the last expression.  
    
            scm> (cond ((= 4 3) 'nope)
                   ((= 4 4) 'hi)
                   (else 'wait))
            hi
            scm> (cond ((= 4 3) 'wat)
                       ((= 4 4))
                       (else 'hm))
            #t
            scm> (cond ((= 4 4) 'here (+ 40 2))
                       (else 'wat 0))
            42
    - The value of a `cond` is `undefined` if there are no true predicates and no `else`. In such a case, `do_cond_form` returns `None`. If there is only an `else`, returns its sub-expression. If it doesn't have one, returns `#t`.
    
          scm> (cond (False 1) (False 2))
          scm> (cond (else))
          #t
    - Implemented by the `do_cond_form` function.
    
- `let`:
  - The `let` special form binds symbols to values locally, giving them their initial values. For example:
  
        scm> (define x 5)
        x
        scm> (define y 'bye)
        y
        scm> (let ((x 42)
                   (y (* x 10)))  ; x refers to the global value of x, not 42
               (list x y))
        (42 50)
        scm> (list x y)
        (5 bye)
  - Implemented by the `do_let_form` function, which calls `make_let_frame`. `make_let_frame` returns a child frame of `env` that binds the symbol in each element of bindings to the value of its corresponding expression. The `bindings` Scheme list contains pairs that each contain a symbol and a corresponding expression.
  
- `mu`:
  - All of the above Scheme special forms use **lexical scoping**: The parent of the new call frame is the environment in which the procedure was defined. Another type of scoping, which is not standard in Scheme, is called **dynamic scoping**: The parent of the new call frame is the environment in which the procedure was evaluated. With dynamic scoping, calling the same procedure in different parts of code can lead to different results (because of varying parent frames).
  - The `mu` special form, a non-standard Scheme expression type, is a procedure that is **dynamically scoped**.
  - In the example below, the `mu` special form is used instead of `lambda` to define a dynamically scoped procedure `f`:

        scm> (define f (mu () (* a b)))
        f
        scm> (define g (lambda () (define a 4) (define b 5) (f)))
        g
        scm> (g)
        20
    The procedure `f` does not have an `a` or `b` defined; however, because `f` gets called within the procedure `g`, it has access to the `a` and `b` defined in `g`'s frame.
  - A `mu` expression is similar to a `lambda` expression, but it evaluates to a `MuProcedure` instance, which is **dynamically scoped**, instead of a `LambdaProcedure` instance, which is *lexically scoped*.
  - Implemented by the `do_mu_form` function.

## Running the Interpreter
To start an interactive Scheme interpreter session, type:

    python3 scheme.py

The Scheme interpreter can be used to evaluate expressions in an input file by passing the file name as a command-line argument to `scheme.py`:

    python3 scheme.py tests.scm

To exit the Scheme interpreter, press `Ctrl-d` or evaluate the `exit` procedure:

    scm> (exit)
