---
title: Scheme Review 1
date: 2023-07-04
tags: [scheme, lisp]
---
https://courses.cs.washington.edu/courses/cse341/04wi/lectures/14-scheme-quote.html

## Quote
Scheme has a convenient syntax for representing data literals: prefix any expression with ' (single quote) and the expression, rather than being evaluated, will be returned as data:

```scheme
'3        ; => 3                 (a number)
'"hi"     ; => "hi"              (a string)
'a        ; => a                 (a symbol)
'(+ 3 4)  ; => (list '+ '3 '4)   (a list)
'(a b c)  ; => (list 'a 'b 'c)   (a list)

'(define x 25)                   (a list)
          ; => (list 'define 'x '25)
          ; => (list 'define 'x 25)

'(lambda (x) (+ x 3))            (a list)
          ; => (list 'lambda (list 'x) (list '+ 'x '3))
          ; => (list 'lambda (list 'x) (list '+ 'x 3))
```
As these examples illustrate, "quoted" data remains unevaluated, and provides a convenient way of representing Scheme programs. This is one of the big payoffs of Lisp's simple syntax: since programs themselves are lists, it is extremely simple to represent Lisp programs as data. Compare the simplicity of quoted lists with the ML datatype that we used to represent ML expressions.

This makes it simple to write programs that manipulate other programs --- it is easy to construct and transform programs on the fly.

Note that names in Lisp programs are translated into symbols in quoted Lisp expressions. This is so that quoted names can be distinguished from quoted strings; consider the difference between the following two expressions:
```scheme
'(define x 10)    ; => (list 'define 'x 10)
'("define" x 10)  ; => (list "define" 'x 10)
```

This distinction is probably the only good reason that strings and symbols are distinct data types in Lisp.

## Quasiquote
Sometimes one doesn't want to escape evaluation of an entire list. In this case, one can use the ` (backquote) operator, also called quasiquote with the , (comma) operator, also called unquote. Quasiquote behaves like quote, except that unquoted expressions are evaluated:

```scheme
`(1 2 ,(+ 3 4))   ; => '(1 2 7)
(quasiquote (1 2 (unquote (+ 3 4))))  ; => '(1 2 7)
```