---
{"dg-publish":true,"dg-permalink":"lambda","permalink":"/lambda/","noteIcon":"","created":"2024-06-16T02:13:47.870+08:00","updated":"2024-06-25T15:23:13.550+08:00"}
---

#PL #Logic
- $\alpha$-equivalent/conversion
	- rename var  $\lambda x.x \Longleftrightarrow \lambda d.d$
- $\lambda$-equivalent
	- $x\mapsto M \Longleftrightarrow \lambda x.M$
- $\beta$-reduction
	- substitute
	- $(\lambda x.x + 1)\ 3 \rightarrow_{\beta} 3 + 1$
	- $(\lambda x.M)N\rightarrow_{\beta} M[N\ \text{substituted}\ for\ x]$ with abbreviated to $M[N/x]$
	- Beta normal form is when you cannot beta reduce (apply lambdas to arguments) the terms any further.

### Syntax
- Variables: x, y...
- $\lambda$-Abstractions: $\lambda x.M\ :\ A\rightarrow B$
	- where $x:A,\ M:B$
- Applications: $M\ M$
### Computation
- $(\lambda x.M)N\rightarrow_{\beta} M[N/x]$

That's it. We can do anything a computer can do. Without loops, booleans, data structures...
- **Church-Turing Thesis**
Which we must call a new concept, Higher-order
- Functions can be passed as inputs to functions
- Functions can return functions as output
- like $(\lambda x.\lambda y.x + y)1\ 2\rightarrow_{\beta} (\lambda y.1 + y)2\rightarrow_{\beta} 1 + 2$

and a new concept flows out, Currying
-  $(\lambda x.\lambda y.L)M\ N\rightarrow_{\beta} (\lambda y.L[M/x])N\rightarrow_{\beta} L[M/x][N/y]$

How can we represent booleans or conditionals?
- True =  $\lambda x.\lambda y.x$
- False =  $\lambda x.\lambda y.y$
- if then =.  $\lambda b.\lambda x.\lambda y.b\ x\ y$

Types need to be limited.

#[Curry–Howard correspondence](https://en.wikipedia.org/wiki/Curry%E2%80%93Howard_correspondence)
