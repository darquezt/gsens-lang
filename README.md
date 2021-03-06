# GSens

GSens is a functional, expression-based programming languages with gradual sensitivity typing.

## Install

```bash
npm i -g @gsens-lang/cli
```

## Usage

```bash
gsens <file>
```

or

```bash
gsens run <file>
```

To see other available commands run

```bash
gsens --help
```

## Features

GSens is a statically-typed language with support for gradual sensitivity information.
A well-typed GSens expression has a type-and-effect which corresponds to a type (as in the sumply typed lambda calculues) and a sensitivity effect that describes how the result of the expression may change given a variation on inputs (the variables it depends on). We present some examples:

- The expression `2 + 3` is not sensitive because it does not depend on any variable. Thus, no matter how many times we reduce the expression to a value, its result will not change. In GSens, this expression has the type-and-effect `Number@[]`, i.e. type `Number` and an empty sensitivity effect. Basically, all literal expression (numbers, booleans, nil) are non-sensitive.

- The expression `x + 2y + 3` is 1-sensitive in `x` and 2-sensitive in `y`, because any variation on `x` will not be magnified by a factor greater than 1, and any variation on `y` will not be magnified by a factor greater than 2. The `+ 3` part does not contribute in the sensitivity as its a constant operation. Assuming that both `x` and `y` are numbers and directly-sensitive variables (more on this later), the type-and-effect of the expression will be `Number@[x + 2y]`, where `x + 2y` is the sensitivity effect. A sensitivity effect (or environment) is a mapping between variable names and sensitivities, and a sensitivity is any positive real number (including zero) or $\infty$. For readaibility purposes, sensitivity effects are written with polynomial notation rather than objects or mappings notation.

- The expression `x * y` is $\infty$-sensitive in `x` and `y` because any variation on a particular variable can be indetereminably magnified by the other one. Thus, the only safe sensitivity is $\infty$. The resulting type-and-effect is `Number@[???x + ???y]`.

- The expression `x <= 5` (read "`x` less or equal than 5") is $\infty$-sensitive in `x`. Sensitivity is all about distance between inputs and outputs. Distance between numbers is generally defined as the L1-norm, i.e. `d(n, m) = |n - m|`. In the case of booleans we consider `true` and `false` to be infinite apart, i.e. `d(true, false) = ???`. Therefore, given that a small input variation of `x` can change the result from `true` to `false` (or viceversa), the resulting type-and-effect of `x <= 5` is `Bool@[???x]`.

- The expression `fun (x : Number) { x + x }`, as literals, is not sensitive because functions are considered to be pure values. However, its _latent effect_ is 2-sensitive in `x`. This is the sensitivity effect of executing or reducing the body of the function. The type of this expression is written as `(x:Number -> Number@[2x])`. First, we need to annotate the name of the argument (`x`) in the arrow type because it can appear in the codomain of the type. This type represents a function, denoted as `(_ -> _)`, that takes an argument `x` of type `Number` and returns a type-and-effect `Number@[2x]`. Finally, since that functions are pure values, the resulting type-and-effect is `(x:Number -> Number@[2x])@[]`. Notice that parenthesis are used to dissamiguate whether an effect `@[_]` corresponds to the return type-and-effect of the arrow type or to the funtion type-and-effect.

### Var vs Resources

Gsens provides two ways of declaring a variable:

Sensitive variables or Resources can be declared using the `resource` keyword:

```js
resource var x = 2;
```

This means that `x` is 1-sensitive on itself, even though it is being initialized with a constant.

Non-directly sensitive variables can be declared using the `var` keyword:

```js
var y = 2;
```

In this case, `y` is not-sensitive because it is being instantiated with a constant value.
This does not mean that a variable declared with `var` is not subject to variations, but it inherits the behaviour from the value it was instantiated with.
For example, one can use a resource variable to instantiate a non-resource variable:

```js
var z = x + x + 3;
```

So in this case `z` is 2-sensitive on `x`.

### Numbers

GSens provides a general type `Number` which is inhabited by both floating point numbers, e.g. `3.141592` and integers, e.g. `11235813`.
Numbers can be summed or multiplied

## Project status

GSens is mainly a protoype for research and is not intended to be efficient or to be used in production.
