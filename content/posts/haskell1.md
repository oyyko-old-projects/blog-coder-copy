---
title: "UCSD CSE 230 Midterm Review Note"
date: 2023-11-01T14:56:15-07:00
tags: [haskell, plt, ucsd]
toc: true
math: false
---

## 01-lambda
Programs are expressions e (also called λ-terms) of one of three kinds:

* Variable
  * x, y, z
* Abstraction (aka nameless function definition)
  * (\x -> e)
  * x is the formal parameter, e is the body
  * “for any x compute e”
* Application (aka function call)
  * (e1 e2)
  * e1 is the function, e2 is the argument
  * in your favorite language: e1(e2)

---

Execute = rewrite step-by-step

  * Following simple rules
  * until no more rules apply

---

An variable x is free in e if there exists a free occurrence of x in e.

If e has no free variables it is said to be **closed**.

Closed expressions are also called **combinators**

What is the shortest closed expression? \x->x

---

Rewrite Rules of Lambda Calculus

β-step (aka function call)
α-step (aka renaming formals)

---

Semantics: Redex
A redex is a term of the form

(\x -> e1) e2

A function (\x -> e1)
  * x is the parameter
  * e1 is the returned expression
Applied to an argument e2
  * e2 is the argument

---

Semantics: β-Reduction

A redex b-steps to another term …

  (\x -> e1) e2   =b>   e1[x := e2]

where e1[x := e2] means

**“e1 with all free occurrences of x replaced with e2” and as long as no free variables of e2 get captured**

Computation by search-and-replace:

  * If you see an abstraction applied to an argument, take the body of the abstraction and replace all free occurrences of the formal by that argument

  * We say that (\x -> e1) e2 β-steps to e1[x := e2]

---

Semantics: α-Renaming

\x -> e   =a>   \y -> e[x := y]
    where not (y in FV(e))


  * We rename a formal parameter x to y
  * By replace all occurrences of x in the body with y
  * We say that \x -> e α-steps to \y -> e[x := y]

---

Recall redex is a λ-term of the form

(\x -> e1) e2

A λ-term is in **normal form** if it contains no redexes.

---

Semantics: Evaluation
A λ-term e evaluates to e' if

1. There is a sequence of steps
e =?> e_1 =?> ... =?> e_N =?> e'
where each =?> is either =a> or =b> and N >= 0

2. e' is in **normal form**

**So the result of a evaluation must be a normal form!!!**

---

**ELSA**

Named λ-terms:

let ID = \x -> x  -- abbreviation for \x -> x



To substitute name with its definition, use a =d> step:

ID apple
  =d> (\x -> x) apple    -- expand definition
  =b> apple              -- beta-reduce



Evaluation:

* e1 =*> e2: e1 reduces to e2 in 0 or more steps
  * where each step is =a>, =b>, or =d>
* e1 =~> e2: e1 evaluates to e2 and e2 is in normal form
