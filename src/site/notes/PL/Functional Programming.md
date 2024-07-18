---
{"dg-publish":true,"permalink":"/pl/functional-programming/","noteIcon":"","created":"2024-06-25T12:55:05.712+08:00","updated":"2024-07-18T00:13:49.894+08:00"}
---


#PL #Haskell #First_Class
### What is functional programming?
**Functional programming** is a computer programming paradigm that relies on functions modeled on mathematical functions. The essence of functional programming is that programs are a combination of expressions. *Expressions* include concrete values, variables, and also functions.

*Functions* have a more specific definition: they are expressions that are applied to an
argument or input, and once applied, can be reduced or evaluated. Functions are [[PL/First-class\|First-class]].

### What is a function?
A **function** is a relation between a set of possible inputs and a set of possible outputs. The function itself defines and represents the relationship.


An **abstraction** is a function. It is a lambda term that has a head (a lambda) and a body and is applied to an argument. An argument is an input value.
Abstractions consist of two parts: the head and the body. The head of the function is a $\lambda$ (lambda) followed by a variable name. The body of the function is another expression.
Go to [[PL/Lambda Calculus\|Lambda Calculus]] for a further read.
