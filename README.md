# Overview

Schnell is ...

# Modules

The user creates a *module* and adds various *module objects* to it. Examples of such module objects are type factories and generic functions.

Each module object has a *name* (a symbol). Objects added to a module can be looked up using this name. Each kind of module object lives in its own namespace.

# Type factories

- `name`
- `constructor`: a Scheme procedure used to create a type
- `parents`: a list of type factories (used to establish a type hierarchy)

A *type expression* is either a symbol or a list starting with a symbol. In both cases, the symbol serves as the *type name*. If the type expression is a list and it has elements after the initial type name, those elements become *type arguments*. Type arguments must be numbers or type expressions.

A type expression is *resolved* by looking up the corresponding type factory (by type name), then applying the constructor of this factory to the resolved type arguments (if any). The resulting *type* is cached in the type factory so that future lookups of the same type expression result in the same type.

Types "remember" which type factory they are coming from. With the help of the `parents` attribute, type factories - and thus types themselves - can be organized into a type hierarchy. The *parents* of type factory `tf` are those type factories which appear in the `parents` list of `tf`. The *ancestors* of type factory `tf` are those type factories which are either parents of `tf` or ancestors of a parent of `tf`.

Every object referred to by the program has an associated type. In the source code these types are denoted by type expressions. Type expressions are resolved into types when the program gets compiled.

If `t1` and `t2` are types, `(isa? t1 t2)` returns #t if all operations supported by an object of type `t2` are also supported by an object of type `t1`. The isa? function works as follows:

1. Let `(tf t)` denote the type factory of type `t`.
2. Let `tf1` be `(tf t1)` and `tf2` be `(tf t2)`.
3. If `tf2` is an ancestor of `tf1`, return #t
4. If `tf2` is not equal to `tf1`, return #f
5. If there is a specialization of isa? for the `tf1`==`tf2` case, apply it to `t1` and `t2`, return the result
6. return #f

# Special forms

- `name`: a symbol
- `compiler`: a Scheme procedure which compiles uses of this special form

When the compiler processes an application whose operator is the name of a special form, it applies the associated compiler procedure to the (unevaluated) operands of the special form. The compiler procedure can do any changes it wishes to the current function or module. If it returns a truthy value (not #f), the value is substituted in place of the original special form and processed as if it originally stood there.

# Generic functions

- `name`
- `methods`

A generic function is a collection of methods under a single name.

Each method has its own *type signature* which specifies what kind of arguments are accepted by the method.

For every function call, the compiler looks up the generic function (by name), then searches for the method whose type signature is the closest match for the actual types of arguments in the call. A call to this method will then be generated at the call site. In other words, method resolution happens at compile time.

# Methods

- `name`
- `ret`: return type
- `args`: arguments (names and types)
- `include`: the include file which declares this function
- `body`: a list of forms
- `compiler`: a Scheme procedure which compiles calls to this function

If a function is externally declared (typically in a C include file) then `include` is set to the name of the include file. If the value is a string, it will be generated into the C code as `#include "value"`. If it's a list containing a single string, the C code will be `#include <value>`.

If the function is both declared and defined, either `body` or `compiler` should be set. `Body` is a list of forms which should be compiled as the body of the function. `Compiler` is a Scheme procedure which compiles calls made to this function. By default, this is a procedure which generates a C call to the corresponding function.

# Typical use

The user creates one or more modules and populates then with module objects:

- type factories
- generic functions and their methods
- specials

The modules are linked together through their import lists to ensure proper resolution of external references.

When everything is ready, the user asks the system to create an executable file by invoking `(compile <module> <entry-point>)` where `<entry-point>` is a symbol which resolves to a thunk (a method with zero arguments and no return value) in `<module>`. The result of the `compile` call is an `<executable>` object. The user then calls `(run <executable> <arguments>)` which causes the program to run in a subprocess. The `run` call returns the exit status of the program.
