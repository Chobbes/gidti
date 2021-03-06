# 5. Introduction to Idris

In this chapter, we will introduce Idris' syntax, by defining functions and types.

Depending at what level of abstraction we are working on, types and proofs can give us a kind of security based on some truths we take for granted (axioms). In fact, this is how we program on a daily basis, as software engineers. We have a list of axioms, for example a `foreach` loop in a programming language, and starting from it we build abstractions. However, this is not always easy to achieve. For example, consider a scenario where we have a button that is supposed to download a PDF document. In order to prove that it works as expected, we must first pick the abstraction level we will be working on, and then proceed by defining things (what is a PDF, what is a download). So, we first have to define our **specifications**, and then we can proceed with proving correctness.

Idris, even though a research language, can still have its own uses. It is a Turing complete language, which means that it has the same expressive power as other programming languages, for example Java, or C++.

There are several ways to install Idris. For Haskellers, one way is to install Idris as an additional package with `cabal install idris`. The second way is direct Idris installation via https://www.idris-lang.org/download.

## 5.1. Basic syntax and definitions

There are two working modes in Idris: REPL (read-evaluate-print-loop) i.e. interactive mode, and compilation of code. We will mostly work in the interactive mode in this chapter.

You can copy any of the example codes to some file, say, `test.idr` and launch Idris in REPL mode by writing `idris test.idr`. This will allow you to interact with Idris given the definitions in the file. If you change the contents of the file, that is, update the definitions, you can use the command `:r` while in REPL mode to reload the definitions.

### 5.1.1. Basic definitions

Functions are defined as follows:

```
add_1 : Nat -> Nat
add_1 x = x + 1
```

With the code above we're defining a function {$$}f(x) = x + 1{/$$} where {$$}x{/$$} is natural number, and {$$}f(x){/$$} (the result) is also a natural number[^ch5n1]. The first line `add_1 : Nat -> Nat` specifies the type of our function, that is, it is a function that takes a natural number and returns a natural number. The second line, `add_1 x = x + 1` is the definition of the function, which states that if `add_1` is called with a number `x`, the result would be `x + 1`. As can be seen by the example, every function has to be provided a type definition. We can interact as follows once in REPL mode:

```
Idris> add_1 5
6 : Nat
```

Idris responds to us that as a result we get 6, which is of type `Nat`. Constants are defined similarly:

```
number_1 : Nat
number_1 = 1
```

As in Haskell, we can use pattern matching. What happens during this phase is Idris does a check (match) against the definitions of the function and uses the definition of the function that matches the value.

```
is_it_zero : Nat -> Bool
is_it_zero 0 = True
is_it_zero x = False
```

We've just defined a function that accepts a natural number, and returns a boolean value (`True` or `False`). On the first line, we specify its type. On the second line, we pattern match against the input `0` and return `True` in that case. On the third line, we pattern match against the input `x` (which is all remaining inputs except 0). In this case, we return `False`. So, what the code above does is, depending on the input, it will branch the computation to the corresponding definition.

X> ### Exercise 1
X>
X> Write a function `my_identity` which accepts a natural number and returns the same number.

X> ### Exercise 2
X>
X> Write a function `five_if_zero` which accepts a natural number, and returns 5 when called with an argument of 0, and otherwise returns the same number. For example, `five_if_zero 0` should return 5. `five_if_zero 123` should return 123.

### 5.1.2. Defining our own types

In Idris, types are first-class citizens. This means that types can be computed and passed to other functions. We define new types by using the keyword `data`. All types should begin with an uppercase letter, since lowercase letter variables are reserved for polymorphic types in functions. There are a couple of ways to define types. One example is by using the so called `Haskell98` syntax:

```
data A a b = A_inst a b
```

This will create a new type that accepts two arguments, `a` and `b`. Valid types are `A Nat Bool`, `A Nat Nat`, etc. The `A_inst` part is actually the type constructor that instantiates an object of such type:

```
Idris> A_inst True True
A_inst True True : A Bool Bool
```

An equivalent definition by using the `GADT` (Generalized Algebraic Data Types) syntax:

```
data A : Type -> Type -> Type where
	A_inst : a -> b -> A a b
```

Which is equivalent to the following definition, where we define an empty data structure along with an axiom:

```
data A : Type -> Type -> Type
postulate A_inst : a -> b -> A a b
```

With the `postulate` keyword we can define axioms, which are functions that satisfy the types without giving an actual argument for construction. With the command `:t` we can check the type of an expression, so as a result, we get:

```
Idris> :t A
A : Type -> Type -> Type
Idris> :t A_inst
A_inst : a -> b -> A a b
```

That is, we show the type definition for both the newly-defined type and its type constructor. Note how we created a product type here. Idris has a built-in notion of pairs, which is a data type that can be defined in terms of products. So for example, `(1, 2)` is one pair. We can also define tuples with `(1, "Hi", True)`, which is equivalent to `(1, ("Hi", True))`, i.e. a pair where the first element is a number, and the second element is a pair.

Analogously, if we want to create a sum type, we could do the following:

```
data B a b = B_inst_left a | B_inst_right b
```

As a result of that, we get:

```
Idris> :t B
B : Type -> Type -> Type
Idris> :t B_inst_left
B_inst_left : a -> B a b
Idris> :t B_inst_right
B_inst_right : b -> B a b
```

For extracting values from data types such as `B a b`, we can use pattern matching. As an example, to extract `a` from `B a b`, we can use the following function:

```
f : B a b -> a
f (B_inst_left a) = a
```

Note how we use the data type at the function type level, and how we use the type constructor to pattern match against.

Natural numbers are defined as `data Nat = Z | S Nat`, where we either have a zero or a successor of a natural number. Natural numbers are built-in as a type in Idris. We can use the operator `==` to compare two numbers. Note that `==` is still a function, but it's an infix one. This means that unlike other functions that we define which are prefix, i.e. start at the beginning of the expression, `==` needs to appear between the parameters. For example:

```
Idris> 1 == 1
True : Bool
```

Idris has several built-in primitive types, including: `Bool`, `String` (list of characters), `Integer`, `Char`, `List`, etc. The only type constructors for the type `Bool` are `True` and `False`. The difference between a `Nat` and an `Integer` is that `Integer` can also contain negative values. Here are some examples of interacting with these types:

```
Idris> "Test"
"Test" : String
Idris> '!'
'!' : Char
Idris> 1
1 : Integer
Idris> '1'
'1' : Char
Idris> [1, 2, 3]
[1, 2, 3] : List Integer
```

In order to make Idris infer the necessary type of the function that needs to be built, we can take advantage of holes. A hole is any variable that starts with a question mark. For example, if we have the following definition:

```
test : Bool -> Bool
test p = ?hole1
```

We can now ask Idris to tell us the type of `hole1`, that is, with `:t hole1` we can see that Idris inferred that the specific result is expected to be of type `Bool`. This is useful because it allows us to write programs incrementally (piece by piece) instead of constructing the program all at once.

X> ### Exercise 3
X>
X> Define your own custom type. One example is `data Person name age = PersonInst name age`. Then check the type of the type constructor with `:t`.

X> ### Exercise 4
X>
X> Invent a function that works with your custom type (for example, it extracts some value) by pattern matching against its type constructor(s).

X> ### Exercise 5
X>
X> Repeat exercise 4 and use holes to check the types of the expressions used in your function definition.

### 5.1.3. Lambda anonymous functions

With the syntax `let X in Y` we're defining a set of variables `X` which are only visible in the body of `Y`. As an example, here is one way to use this syntax:

```
Idris> let f = 1 in f
1 : Integer
```

Alternatively, the REPL has a command `:let` that allows us to set a variable without evaluating it:

```
Idris> :let f = 1
Idris> f
1 : Integer
```

Lambda (anonymous) functions are defined with the syntax `\a, b, ..., n => ...`. For example:

```
Idris> let addThree = (\x, y, z => x + y + z) in addThree 1 2 3
6 : Integer
```

With the example above, we've defined a function `addThree` that accepts three parameters, and as a result it sums them. However, if we do not pass all parameters to a function, it will result in:

```
Idris> let addThree = (\x, y, z => x + y + z) in addThree 1 2
\z => prim__addBigInt 3 z : Integer -> Integer
```

We can see (from the type) that as a result we get another function.

I> ### Definition 1
I>
I> Currying is a concept that allows us to evaluate a function with multiple parameters as a sequence of functions, each having a single parameter.

Function application in Idris is left-associative (just like in lambda calculus), which means that if we try to evaluate `addThree 1 2 3`, then it will be evaluated as `(((addThree 1) 2) 3)`. A combination of left-associative functions and currying (i.e. partial evaluation of function) is what allows us to write `addThree 1 2 3`, which is much more readable.

Abstractions are right-associative (just like in lambda calculus), which means that `addThree 1 : a -> a -> a` is equivalent to `addThree 1 : (a -> (a -> a))`. If we had written a type of `(a -> a) -> a` instead, then this function would accept as its first parameter a function that takes an `a` and returns an `a`, and then the original function also returns an `a`. This is how we can define higher-order functions which we will discuss later.

Note that `a` is a polymorphic type, which means that it can accept all types which are either defined by the programmer or primitive ones.

The `if...then...else` syntax is defined as follows, which is quite intuitive:

```
Idris> if 1 == 1 then 'a' else 'b'
'a' : Char
Idris> if 2 == 1 then 'a' else 'b'
'b' : Char
```

X> ### Exercise 6
X>
X> Write a lambda function that returns `True` if the parameter passed to it is 42, and `False` otherwise.

### 5.1.4. Recursive functions

By definition 18 of chapter 3, recursive functions are those functions that refer to themselves. One example is the function `even : Nat -> Bool`, a function that checks if a natural number is even or not. It can be defined as follows:

```
even : Nat -> Bool
even Z = True
even (S k) = not (even k)
```

The definition states that 0 is an even number, and that `n + 1` is even or not depending on the parity (evenness) of `n`. As a result, we get:

```
Idris> even 3
False : Bool
Idris> even 4
True : Bool
Idris> even 5
False : Bool
```

The way `even 5` unfolds in Idris is as follows:

```
even 5 =
not (even 4) =
not (not (even 3)) =
not (not (not (even 2))) =
not (not (not (not (even 1)))) =
not (not (not (not (not (even 0))))) =
not (not (not (not (not True)))) =
not (not (not (not False))) =
not (not (not True)) =
not (not False) =
not True =
False
```

We see how this exhibits a recursive behaviour since the recursive cases were reduced to the base case in attempt to get a result. With this example, we can see the power of recursion and how it allows us to process values in a repeating manner.

A recursive function can generate an **iterative** or a **recursive** process. An iterative process is a process where the "state" is captured completely by its parameters. A recursive one, in contrast, is one where the "state" is not captured by the parameters, and so it relies on "deferred" evaluations. In the example above, `even` generates a recursive process since it needs to go down to the base case, and then build its way back up to do the calculations that were "deferred". Alternatively, we can rewrite `even` so that it captures the "state" by introducing another variable, as such:

```
even : Nat -> Bool -> Bool
even Z t     = t
even (S k) t = even k (not t)
```

Now, at any point in computation, we can view the state of the function result by referring to the second parameter. As a consequence, this function now generates an iterative process, since the results are captured in the parameters. Note how we brought down the base case to refer to the parameter, instead of a constant `True`. Here is how Idris evaluates `even 5 True`:

```
even 5 True =
even 4 False =
even 3 True =
even 2 False =
even 1 True =
even 0 False =
False
```

Recursive functions combined with pattern matching are one of the most powerful tools in Idris, since by using them we can do computation. They are also useful for proving mathematical theorems with induction, as we will see in the examples later.

X> ### Exercise 7
X>
X> Factorial is defined as:
X> {$$}fact(n) = \left\{ \begin{array}{ll} 1\text{, if } n = 0 \\	n * fact(n - 1)\text{, otherwise} \end{array} \right.{/$$}
X>
X> Unfold the evaluation of {$$}fact(5){/$$} on paper, and then implement it in Idris and confirm that Idris also computes the same value.
X>
X> Hint: The type is `fact : Nat -> Nat` and you should pattern match against `Z` (`Nat` type constructor for 0) and `(S n)` (successor).

X> ### Exercise 8
X>
X> Re-write the factorial function to generate an iterative process.
X>
X> Hint: The type is `fact_iter : Nat -> Nat -> Nat` and you should pattern match against `(S Z) acc` (number 1, and accumulator), and `(S n) acc` (successor, and accumulator).

### 5.1.5. Recursive data types

I> ### Definition 2
I>
I> A recursive data type is a data type where in some of its constructors there is a reference to the same data type.

We will start by defining a recursive data type, which is a data type that in the constructor refers to itself. In fact, earlier we already gave a recursive definition of `Nat`. As a motivating example, we will try to define the representation of lists. For this data type we'll use a combination of sum and product types. A list is defined as either `End` (end of list) or `Element`, which is a value appended to another `MyList`:

```
data MyList a = Element a (MyList a) | End
```

This means that the type `MyList` has two constructors, `End` and `Element`. If it's `End`, then it's the end of the list (and does not accept any more values). However, if it's `Element`, then we can append another value (e.g. `Element 3`), but afterwards we have to specify another value of type `MyList a` (which can be `End` or another `Element`). This definition allows us to define a list. As an example, this is how we would represent {$$}(1, 2, 3){/$$} using our `Element End` representation:

```
Idris> :t Element 1 (Element 2 End)
Element 1 (Element 2 End) : MyList Integer
Idris> :t Element 'a' (Element 'b' End)
Element 'a' (Element 'b' End) : MyList Char
```

Note how Idris automatically infers the polymorphic type to `MyList Integer` and `MyList Char`. Here is one way of implementing the concatenation function, which given two lists, should produce a list with the elements appended:

```
add' : MyList a -> MyList a -> MyList a
add' End ys            = ys
add' (Element x xs) ys = Element x (add' xs ys)
```

The first line of the code says that `add'` is a function that accepts two polymorphic lists (`MyList Nat`, `MyList Char`, etc), and produces the same list as a result. The second line of the code pattern matches against the first list and when it's empty, we just return the second list. The third line of the code also pattern matches against the first list, but this time it covers the `Element` case. So, whenever there is an `Element` in the first list, as a result we return this element `Element x`, appended recursively to `add' xs ys`, where `xs` is the remainder of the first list and `ys` is the second list. Example usage:

```
Idris> add' (Element 1 (Element 2 (Element 3 End))) (Element 4 End)
Element 1 (Element 2 (Element 3 (Element 4 End))) : MyList Integer
```

X> ### Exercise 9
X>
X> Unfold `add' (Element 1 (Element 2 (Element 3 End))) (Element 4 End)` on paper to get a better understanding of how this definition appends two lists.

X> ### Exercise 10
X>
X> Come up with a definition for `length'`, which should return the number of elements, given a list.
X>
X> Hint: The type is `length' : MyList a -> Nat`

X> ### Exercise 11
X>
X> Come up with a definition for `even-only'`, which should return a new list with even natural numbers only.
X>
X> Hint: The type is `even-only' : MyList Nat -> MyList Nat`. You can re-use the definition of `even` we've discussed earlier.

X> ### Exercise 12
X>
X> Come up with a definition for `sum'`, which should return a number that will be the sum of all elements in a list of natural numbers.
X>
X> Hint: The type is `sum' : MyList Nat -> Nat`.

### 5.1.6. Interfaces and implementations

Interfaces are defined using the `interface` keyword, and they allow us to add constraints to functions that implement them[^ch5n2]. As an example, we'll take a look at the `Eq` interface:

```
interface Eq a where
    (==) : a -> a -> Bool
    (/=) : a -> a -> Bool
        -- Minimal complete definition:
        --      (==) or (/=)
    x /= y     =  not (x == y)
    x == y     =  not (x /= y)
```

Note how we can specify comments in the code by using two dashes. Comments are ignored by the Idris compiler, and are only useful to the reader of the code.

This definition says that if a function implements this interface then it has to support (or implement) the functions `==` and `/=` for the type that it's used. Additionally the interface also contains a definition for the methods themselves, but this is optional. Since the definition of `==` depends on `/=` (and vice-versa), it will be sufficient to only override one of them (if we need to), and the other one will be automatically generated. As an example, let's assume that we have a data type:

```
data Foo : Type where
    Fooinst : Nat -> String -> Foo
```

To implement `Eq` for `Foo`, we can use the following syntax:

```
implementation Eq Foo where
    (Fooinst x1 str1) == (Fooinst x2 str2) = (x1 == x2) && (str1 == str2)
```

We use `==` for `Nat` and `String`, since this is already defined in Idris itself. With this, we can easily use `==` and `/=` on the types:

```
Idris> Fooinst 3 "orange" == Fooinst 6 "apple"
False : Bool
Idris> Fooinst 3 "orange" /= Fooinst 6 "apple"
True : Bool
```

X> ### Exercise 13
X>
X> Implement your own data type `Person` that accepts `name` and `age` and implement an interface for comparing `Person`s.
X>
X> Hint: One valid data type is `data Person name age = Personinst name age`.

### 5.1.7. Total and partial functions

I> ### Definition 3
I>
I> A total function is a function that:
I> 
I> 1. Terminates for all possible inputs
I> 1. Returns a value of a potentially infinite result

Analogously, a partial function is one that does not hold for at least one of the points above. If a function is total, its type can be understood as a precise description of what that function does. Idris differentiates total from partial functions. As an example, if we assume that we have a function that returns a `String`, then:

1. If it's total, it will return a `String` in finite time
1. If it's partial, then unless it crashes or enters in an infinite loop, it will return a `String`

In Idris, to define total functions we just put the keyword `total` in front of the function definition. So, for example, for the following program, we define two functions `test` and `test2`, a partial and a total one respectively:

```
test : Nat -> String
test Z = "Hi"

total
test2 : Nat -> String
test2 Z = "Hi"
test2 _ = "Hello"
```

If we try to interact with these functions, we get the following results:

```
Idris> test 0
"Hi" : String
Idris> test 1
test 1 : String
Idris> test2 0
"Hi" : String
Idris> test2 1
"Hello" : String
```

We can note that the evaluation of `test 1` does not produce a computed value as a result. Note that at compile-time, Idris will **evaluate the types only for total functions**.

X> ### Exercise 14
X>
X> Try to define a function to be `total`, and at the same time make sure you are not covering all input cases. Note what Idris returns in this case.

### 5.1.8. Strict evaluation

I> ### Definition 4
I>
I> Lazy evaluation means that parameters are evaluated only when necessary. Conversely, strict evaluation means that all parameters are evaluated on a function call. As an example:

Idris evaluates parameters in a strict fashion[^ch5n3]. For example, let's take a look at the following function:

```
ifThenElse : Bool -> a -> a -> a
ifThenElse True  t e = t
ifThenElse False t e = e
```

This function uses either the `t` or the `e` parameter, but not both. But the way it is written, it will evaluate both arguments before returning a result. It is a good optimization to only evaluate the parameter that is used. However, to achieve this, in Idris there is a built-in support for lazy evaluation. The implementation is:

```
data Lazy : Type -> Type where
    Delay : (val : a) -> Lazy a
    Force : Lazy a -> a
```

A value of type `Lazy a` will not be evaluated until `Force` is called upon it. Now we can write our `ifThenElse` function in the following way:

```
ifThenElse : Bool -> Lazy a -> Lazy a -> a
ifThenElse True  t e = t
ifThenElse False t e = e
```

### 5.1.9. Documentation and searching

By using the `:doc` command, we can get detailed information about a data type:

```
Idris> :doc Nat
Data type Prelude.Nat.Nat : Type
    Natural numbers: unbounded, unsigned integers which can be pattern matched.
    
Constructors:
    Z : Nat
        Zero
        
    S : Nat -> Nat
        Successor
```

We can use the command `:search` to get a list of functions that have the corresponding type. For example:

```
Idris> :search (Nat -> Nat)
...

= Prelude.Nat.S : Nat -> Nat
Successor

= Prelude.Nat.fact : Nat -> Nat
Factorial function
...
```

X> ### Exercise 15
X>
X> Check the documentation for `Bool` and `List`.

X> ### Exercise 16
X>
X> Find out a few binary operators for `Nat` by searching `Nat -> Nat -> Nat`, and then try to use some of them.

### 5.1.10. Dependent types

We will implement the `List n` data type that we discussed in section 4.3, which should limit the length of a list at the type level. To not conflict with the built-in Idris' `List`, we'll name it `MyList`. We can implement it as follows:

```
data MyList : (n : Nat) -> Type where
    Empty : MyList 0
    Cons  : (x : Nat) -> (xs : MyList len) -> MyList (S len)
```

What we've done above is we created a new type called `MyList` which accepts a natural number and returns a `Type`, that is joined with two type constructors:

1. `Empty` - which is just the empty list
1. `Cons : (x : Nat) -> (xs : MyList len) -> MyList (S len)` - which, given a natural number and a list of length `len`, it will return a list of length `S len`, that is, `len + 1`

If we now use the following code snippet, it will pass the compile-time checks:

```
x : MyList 2
x = Cons 1 (Cons 2 Empty)
```

However, if we try to use this code snippet instead:

```
x : MyList 3
x = Cons 1 (Cons 2 Empty)
```

We will get the following error:

```
Type mismatch between
	MyList 0 (Type of Empty)
and
	MyList 1 (Expected type)
```

Which is a way of Idris telling us that our types do not match and that it cannot verify the "proof" provided.

In this example we've implemented a dependent type that puts the length of the list at the type level. In other programming languages that do not support dependent types, this is usually checked at the code level (run-time), and compile-time checks are not able to verify this.

X> ### Exercise 17
X>
X> Come up with a function `isSingleton` that accepts a `Bool` and returns a `Type`. This function should return a type of `Nat` in the `True` case, and `List Nat` otherwise. Further, implement a function `mkSingle` that accepts a `Bool`, and returns `isSingleton True` or `isSingleton False`, and as a computed value will either return `0` or `Empty`.
X>
X> Hint: The data definitions are `isSingleton : Bool -> Type` and `mkSingle : (x : Bool) -> isSingleton x` respectively.

### 5.1.11. Implicit parameters (or arguments)

Implicit parameters allow us to bring values from the type level to the program level. At the program level, using curly braces we allow them to be used in the body of the function. Let's take a look at the following example, which uses our dependent type `MyList` that we defined earlier:

```
lengthMyList : MyList n -> Nat
lengthMyList { n = Z } list = 0
lengthMyList { n = k } list = k
```

In this case, we've defined a function `lengthMyList` that takes a `MyList` and returns a natural number. The value `n` in the body of the function will be the same with the value of `n` at the type level. They are called implicit parameters because the caller of this function needn't pass these parameters. In the function body, we define implicit parameters with curly braces, and we also need to specify the list parameter which is of type `MyList n` to pattern match against. But, note how we don't refer to the list parameter in the computation part of this function, so instead, we can use an underscore (which represents an unused parameter), to get to:

```
lengthMyList : MyList n -> Nat
lengthMyList { n = Z } _ = 0
lengthMyList { n = k } _ = k 
```

We can also have implicit parameters at the type level. As a matter of fact, an equivalent type definition of that function is:

```
lengthMyList : {n : Nat} -> MyList n -> Nat
```

If we ask Idris to give us the type of this function, we will get the following for either of the type definitions above:

```
Idris> :t lengthMyList
lengthMyList : MyList n -> Nat
```

However, we can use the command `:set showimplicits` which will show the implicits on the type level. If we do that, we will get the following for either of the type definitions above:

```
Idris> :set showimplicits
Idris> :t lengthMyList
lengthMyList : {n : Nat} -> MyList n -> Nat
```

To pass values for implicit arguments, we can use the following syntax:

```
Idris> lengthMyList {n = 1} (Cons 1 Empty)
1 : Nat
```

X> ### Exercise 18
X>
X> Try to evaluate `lengthMyList {n = 2} (Cons 1 Empty)` and observe the results.

### 5.1.12. Higher order functions

I> ### Definition 5
I>
I> A higher order function is a function that takes one or more functions as parameters, or returns a function as a result.

There are three built-in higher order functions that are generally useful: `map`, `filter`, `fold` (left and right). Here's the description of each:

1. `map` is a function that takes as input a function with a single parameter, and a list, and returns a list where all members of the list have this function applied
1. `filter` is a function that takes as input a function (predicate) with a single parameter (that returns a `Bool`), and a list, and only returns those members in the list whose predicate evaluates to `True`
1. `fold` is a function that takes as input a combining function that accepts two parameters (current value and accumulator), an initial value, and a list, and returns a value combined with this function. There are two types of folds, a left and a right one, which combines from the left and from the right respectively

As an example usage:

```
Idris> map (\x => x + 1) [1, 2, 3]
[2, 3, 4] : List Integer
Idris> filter (\x => x > 1) [1, 2, 3]
[2, 3] : List Integer
Idris> foldl (\x, y => x + y) 0 [1, 2, 3]
6 : Integer
Idris> foldr (\x, y => x + y) 0 [1, 2, 3]
6 : Integer
```

We can actually implement the map function ourselves:

```
mymap : (a -> a) -> List a -> List a
mymap _ [] = []
mymap f (x::xs) = (f x) :: (mymap f xs)
```

Note that `::` is used by the built-in `List` type, and is equivalent to `Cons` we've used earlier. In addition, the built-in `List` type is polymorphic. 

X> ### Exercise 19
X>
X> Do a few different calculations with `mymap` in order to get a deeper understanding of how it works.

X> ### Exercise 20
X>
X> Implement a function `myfilter` that acts just like the `filter` function.
X>
X> Hint: Use `:t filter` to get its type.

X> ### Exercise 21
X>
X> Given `foldl (\x, y => [y] ++ x) [] [1, 2, 3]` and `foldr (\x, y => y ++ [x]) [] [1, 2, 3]`:
X>
X> 1. Evaluate both of them in Idris to see the values produced
X> 2. Try to understand the differences between the 2 expressions
X> 3. Remove the square brackets `[` and `]` in the lambda body to see what errors Idris produces
X> 4. Evaluate them on paper to figure out why they produce the given results

### 5.1.13. Reasoning by cases

For the sake of example, let's assume that we have the following data structure:

```
data Probably x = Kinda x | Nope
```

The `Probably` data structure allows us to encode an additional value for the given polymorphic type. So, if we use `Probably Bool` and we know that `Bool` has only two constructors, then with `Probably Bool` we get one additional constructor, `Nope`. Further, let's see how we can use this data type as an example:

```
right_answer : Int -> Probably Bool
right_answer 1  = Kinda False
right_answer 42 = Kinda True
right_answer _  = Nope
```

Now let's assume that we just want to return a `Bool` in terms of `right_answer`, that is `True` for the 42 case and `False` otherwise. To extract values from this function, we can take several approaches: we can use pattern matching, we can use the `if...then...else` syntax, or another alternative is to use the `case` keyword, which has a form of:

```
case (conditional_to_check) of
    pattern_match_1 => ...
    pattern_match_2 => ...
    ...
    _               => ...
```

It works similarly to pattern matching, but can be convenient when we have a function that has more than a few lines of code, and we need to pattern match somewhere in between. If we instead did a separate pattern match, we would have to duplicate the existing lines of code, where with `case` we can pattern match right away. The underscore `_` matches all unspecified cases.

```
is_right : Int -> Bool
is_right x = case (right_answer x) of
                 Kinda y => case y of
                     True => True
                     _    => False
                 _ => False
```

Interacting:

```
Idris> is_right 1
False : Bool
Idris> is_right 42
True : Bool
```

X> ### Exercise 22
X>
X> In fact, Idris has a built-in data similar to `Probably`, which is called `Maybe`. Check its documentation with `:doc` and rework `right_answer` and `is_right` to work with `Maybe` instead.

## 5.2. Curry-Howard isomorphism

The Curry-Howard isomorphism (also known as Curry-Howard correspondence) is the direct relation between computer programs and mathematical proofs. It is named after the mathematician Haskell Curry and logician William Howard. In other words, a mathematical proof is represented by a computer program, and the formula we're proving is the type of that program. As an example, we can take a look at the function swap, defined as follows:

```
swap : (a, b) -> (b, a)
swap (a, b) = (b, a)
```

The isomorphism says that this function has an equivalent form of a mathematical proof. Although it may not be immediately obvious, let's consider the following proof: Given {$$}P \land Q{/$$}, prove that {$$}Q \land P{/$$}. In order to prove it, we have to use 2 inference rules: and-introduction and and-elimination, which are defined as follows:

1. And-introduction means that if we are given {$$}P{/$$}, {$$}Q{/$$}, then we can construct a proof for {$$}P \land Q{/$$}
1. Left and-elimination means that if we are given {$$}P \land Q{/$$}, we can conclude {$$}P{/$$}
1. Right and-elimination means that if we are given {$$}P \land Q{/$$}, we can conclude {$$}Q{/$$}

If we try to implement this proof in Idris, we can represent it as a product type, as follows:

```
data And a b = And_intro a b

and_comm : And a b -> And b a
and_comm (And_intro a b) = And_intro b a
```

As we've discussed, we can use product types to encode pairs. Now we can note the following similarities with our earlier definition of `swap`:

1. `And_intro x y` is equivalent to constructing a product type `(x, y)`
1. Left-elimination, which is a pattern match of `And_intro a _` is equivalent to the first element of the product type
1. Right-elimination, which is a pattern match of `And_intro _ b` is equivalent to the second element of the product type

X> ### Exercise 23
X>
X> Given `data Or a b = Or_introl a | Or_intror b`, show that {$$}a \to (a \lor b){/$$} and {$$}b \to (a \lor b){/$$}.
X>
X> Hint: Check the documentation of `the` with `:doc the`, and use it with the type constructors. Programs are proofs, and types are the theorems proven. You may want to come back to this exercise after having finished chapter 6.

## 5.3. IO

IO stands for Input/Output. Examples of a few IO operations are: write to a disk file, talk to a network computer, launch rockets. 

I> ### Definition 6
I>
I> Functions can be roughly categorized in two parts: **pure** and **impure**.
I>
I> 1. Pure functions are functions that every time they are called, will produce the same result
I> 1. In contrast, impure functions are functions that might return different result on a function call

An example of a pure function is {$$}f(x) = x + 1{/$$}. An example of an impure function is {$$}f(x) = \text{launch x rockets}{/$$}. Since this function causes side-effects, sometimes the launch of the rockets may not be successful (e.g. the case where we have no more rockets to launch).

Computer programs are not usable if there is no interaction with the user. One problem arises with languages such as Idris (where expressions are mathematical and have no side effects) is that IO contains side effects. For this reason, such interactions will be encapsulated in a data structure that looks something like:

```
data IO a -- IO operation that returns a value of type a
```

The concrete definition for `IO` is built within Idris itself, that is why we will leave it at the data abstraction as defined above. But what `IO` does is it describes all operations that need to be executed. The resulting operations are executed externally by the Idris Run-Time System (or `IRTS`). The most basic IO program is the following one:

```
main : IO ()
main = putStrLn "Hello world"
```

The type of `putStrLn` says that this function receives a `String` and returns an `IO` operation.

```
Idris> :t putStrLn
putStrLn : String -> IO ()
```

We can read from the input similarly:

```
getLine : IO String
```

In order to combine several IO functions, we can use the `do` notation as follows:

```
main : IO ()
main = do
    putStr "What's your name? "
    name <- getLine
    putStr "Nice to meet you, "
    putStrLn name
```

In the REPL, we can say `:x main` to execute the IO function. Alternatively, if we save that code to `test.idr`, we can use the command `idris test.idr -o test` in order to output an executable file that we can use on our system. Interacting with it:

```
boro@bor0:~$ idris test.idr -o test
boro@bor0:~$ ./test
What's your name? Boro
Nice to meet you, Boro
boro@bor0:~$
```

Let's slightly rewrite our code by abstracting out the concatenation into a separate function:

```
concat_string : String -> String -> String
concat_string a b = a ++ b

main : IO ()
main = do
    putStr "What's your name? "
    name <- getLine
    let concatenated = concat_string "Nice to meet you, " name
    putStrLn concatenated
```

Note how we use the `let x = y` syntax with pure functions, where in contrast we use the `x <- y` with impure functions.

The `++` operator is a built-in one used to concatenate strings and lists. A `String` can be viewed as a list of `Char`. In fact, Idris has functions called `pack` and `unpack` that allow for conversion between these two data types:

```
Idris> unpack "Hello"
['H', 'e', 'l', 'l', 'o'] : List Char
Idris> pack ['H', 'e', 'l', 'l', 'o']
"Hello" : String
```

X> ### Exercise 24
X>
X> Create a program that accepts a `String`, and then prints the length of the input string.

X> ### Exercise 25
X>
X> Find the function that converts a `Nat` to a `String` (search for `Nat -> String`) and then finish the following program:
X>
X> ```
X> main : IO ()
X> main = do
X>     putStrLn ("The number is: " ++ (<??> 12345))
X> ```

[^ch5n1]: It is worth noting that in Haskell we have types and kinds. Kinds are similar to types, that is, they are defined as one level above types in simply typed lambda calculus. For example, types such as `Nat` have a kind `Nat :: *` and it's stated that `Nat` is of kind `*`. Types such as `Nat -> Nat` have a kind of `* -> *`. Since in Idris types are first-class citizens, there is no distinction between types and kinds.

[^ch5n2]: This is equivalent to Haskell's `class` keyword. Interfaces in Idris are very similar to OOP's interfaces.

[^ch5n3]: In contrast, the default behaviour in Haskell is lazy evaluation.
