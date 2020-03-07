# Algebraic Effects

## Navigating the learning material

Most content on the matter tend to involve a lot of formalism - the
most popular one being [What's algebraic about Algebraic
Effects](https://arxiv.org/pdf/1807.05923.pdf). This document compiles
a list with some pointers to help with the navigation.

### Polysemy is fun (Haskell)

[Article link](https://haskell-explained.gitlab.io/blog/posts/2019/07/28/polysemy-is-cool-part-1/)

**Pre-requisites:** Haskell, Monads, Monad Transformers

This article takes a hands on approach to illustrating Algebraic
Effects as means to provide purity without introducing Monads all
over. Employing monads to model effectful computations often require
monad transformers so that different monads can be composed
together. Algebraic Effects, unlike monads, are inherently
compositional and provide an interesting alternative to managing side effects. 

### Algebraic Effects for the rest of us (Javascript)

[Article
Link](https://overreacted.io/algebraic-effects-for-the-rest-of-us/)

**Pre-requisites:** Javascript

Javascript does not have Algebraic Effects. In this article, Dan
Abramov provides intuition for what Algebraic Effects are - using
resumable exceptions as the mental model to understand how Algebraic
Effects work - and the implications to possibilities of abstractions
that open up. This article does not talk about types or purity of
effectful computations. Includes links to more resources to help learn
Alg. Effects.

### Concurrent Programming with Effect Handlers (OCaml)

[Article Link](https://github.com/ocamllabs/ocaml-effects-tutorial)

**Pre-requisites:** OCaml

The multicore team at OCamllabs uses algebraic effects (untyped) as
basic primitive to manage thread scheduling in the compiler. In this
tutorial, KC Sivaramakrishnan (and others) provide step by step
explanation of using Alg. Effects as resumable exceptions that reduce
monad poisoning in concurrent OCaml code and also how popular language
constructs are async/await, generators/iterators, message passing,
exception handling etc can all be expressed with Algebraic Effects and
handlers.

### Lecture Series at Oregon Programming Languages Summer School by Andrej Bauer

[Lecture
1](https://www.youtube.com/watch?v=atYp386EGo8)

[Lecture 2](https://www.youtube.com/watch?v=rIDAxWzccCU)

[Lecture 3](https://www.youtube.com/watch?v=_06324bmFGo)

**Pre-requisites:** Abstract Algebra (and some Category theory)

In this series, Andrej Bauer explains algebraic nature of Algebraic
Effects. The first lecture assumes the audience understands basic
Abstract Algebra and is comfortable with some mathematical rigour
involved in understanding them. Explaining the basics of Universal
Algebra (signatures, equations, models, co-models etc), it explains
what makes Alg. Effects algebraic in nature.

### An Introduction to Algebraic Effects and Handlers by Martija Pretnar

[Tutorial Link](https://www.eff-lang.org/handlers-tutorial.pdf)

Eff is an experimental programming language that acts as a playground
of research ideas around Algebraic Effects. This tutorial introduces
what it means for a programming language (not necessarily Eff) to
model it's effectful operations as effectful computations and
handlers. This tutorial is aim more at those who dont shy away from
operational/denotational semantics of a hypothetical programming languages.

