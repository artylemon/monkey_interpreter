# What is this project 
This is an interpreter written in [Go](https://go.dev/) for a toy programming language called '[monkey](https://monkeylang.org/)'. This project follows the [writing an interpreter in Go](https://interpreterbook.com/) book.

## Why am I doing this
I started this project to deepen my understanding of scripting languages, gain more practical experience with Go, and test my new [neovim environment setup](https://github.com/artylemon/kickstart.nvim).
Now, I want to share my summary of how this interpreter works to show how it is possible to communicate with computers using a human-like language.

## Overview of the monkey language
Here is the demonstration of the monkey's features.

Monkey's built-in data-types: 
```js
// It has the integers and arithmetic
let i = (3 + 2) * 8 / 4

// strings
let s = "Monkey is awesome"

// booleans
let isMonkeyAwesome = true

// arrays and hashmaps:
let arrayOfHashMaps = [{"type": "int", "value": 15}, {"type": "string", value: "name"}]
```

It also supports function literals, allowing to create user-defined functions: 
```js
let fibonacci = fn(x) {
  if (x == 0) {
    0                // Monkey supports implicit returning of values
  } else {
    if (x == 1) {
      return 1;      // ... and explicit return statements
    } else {
      fibonacci(x - 1) + fibonacci(x - 2); // Recursion! Yay!
    }
  }
};
```

User defined functions can even be passed as arguments to higher-order functions: 
```js
// Define the higher-order function `map`, that calls the given function `f`
// on each element in `arr` and returns an array of the produced values.
let map = fn(arr, f) {
  let iter = fn(arr, accumulated) {
    if (len(arr) == 0) {
      accumulated
    } else {
      iter(rest(arr), push(accumulated, f(first(arr))));
    }
  };

  iter(arr, []);
};

// define the function we want to apply to every map
let getType = fn(obj) {obj["type"]}

// Now let's take the `arrayOfHashMaps` array and the `getType` function from above and use them with `map`.
map(arrayOfHashMaps, getType); // => ["int", "string"]
```

Monkey also has some built-in functions, namely:
- `len(arg)` which returns the length of a string, or an array
- `first(arr)`, which returns the first element of an array `arr` (semantically the same as `arr[0]`)
- `last(arr)`, which returns the last element of an array `arr`
- `rest(arr)`, which returns a copy of `arr` without the first element
- `push(arr, obj)`, which pushes `obj` into the array `arr`
- And of course, the most important: `puts(args...)`, which prints all the arguments to the console.

# The stages of interpretation
For a piece of text like "`let a = 1 + 2`" to be translated into actionable instructions for a computer, it will go through three stages of transformation: lexing, parsing and evaluation.
## Lexing
The first stage is about turning a bunch of characters, that a computer sees as a string into 'meaningful chunks', which are technically called **tokens**.
A token would be a substring of our program that has a fixed, well-defined meaning. You, probably, already see what are the tokens from the example string are: `["let", "a", "+", "1", "+", "2"]`.  An example of a substring that is not a token would be something like `"et a"`, it is a a substring of our program, but it is not clear what it actually means.

A lexer does not actually produce this list of tokens, rather it provides a method called `NextToken()`, which outputs one token at a time. 
## Parsing
The goal of parsing is to combine the tokens together, so that it is easier to interpret how they relate to each other. This relationship of tokens to each other is captured in an **abstract syntax tree** (ast).

Our example program would produce the following ast:

![AST_example](https://github.com/user-attachments/assets/3da409fd-6a8b-42b6-97b6-b2d12f8b8422)

The tree shows what we intend to do with this code: we want to create a variable named "a" and assign to it whatever is the result of an expression `1 + 2`. The expression itself is subdivided into the left operand `1`, the operator `+` and the right operand `2`. 
Some other nodes that can be included in the ast are:
- `PrefixExpression` - an expression which only has the right side, like `-1`. It means a negative value, but has to be parsed as a prefix expression.
- `Boolean` - a boolean value.
- `BlockStatement` - a number of statements inside the curly braces `{}`, used as a single block. Something like a function's body would be a block statement.
- `IfExpression` - an expression that represents a general `if-else` statement, capturing the condition expression to be evaluated, the block to be used if the condition evaluates to `true` (the consequence),  and the optional else-block (the alternative).


## Evaluation
After the code is parsed and the ast is constructed, the evaluator traverses the ast, executing all the necessary statements to produce the result of the program. 
Our simple example would resolve to an integer variable named `a` with the value `3` being added to the program's environment.

The evaluator does that, by using pre-defined functions for different types of nodes it can encounter on the ast. In our example, it would start at the let statement, evaluate the `Value` branch, and assign the resulting value to a variable with the identifier `a`. To evaluate the right branch, the `Eval` method calls itself recursively, to evaluate the left branch, which in this case is just the integer literal `1`, then it calls `Eval` to evaluate the right branch, which is also an integer literal `2`, and then it calls a separate procedure `evaluateInfixExpression(Operator)`, which, for our example just adds up the two values, resulting in integer `3` being assigned to a variable `a`.
