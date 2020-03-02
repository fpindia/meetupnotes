Cons Should Not Cons Its Arguments
==================================

## Paper Link

[CONS Should not CONS its Arguments, or, a Lazy Alloc is a Smart Alloc]( 
    https://www.cs.tufts.edu/comp/150FP/archive/henry-baker/cons-lazy-alloc.pdf).

## Introduction

- The primary forms of storage available to a program are **Heap**, **Stack**,
  and a small number of **Machine Registersa**. During the course of a program's
  execution, **values** are allocated on one of these.

- **Stack Allocation** of objects is elegant, efficient and can handle the great
  majority of object allocations. Often the entire stack can fit within the
  processor cache, and allocating memory on a stack is just incrementing a pointer.

- High Level languages like Algol, Pascal, provide automatic allocation and deallocation
  of memory on the stack. Allocation on the heap is left to the programmer to manage.

- Stack Allocation, however is limited because of strict LIFO allocation /
  deallocation ordering. When a new function scope is entered, a **Stack frame**
  is reserved for stack allocation within that function. When the function is
  exited, the stack frame is rewinded and all the associated memory is deallocated.
  This means that some things are not possible with only stack allocation.
  For example, returning a non-primitive or function value (more on this later)).

- Languages such as Lisp, SmallTalk, ML etc. seek to escape these LIFO
  restrictions to gain more expressive power while retaining simplicity and
  elgance. But increased flexibility comes at the cost of increased use of **Heap
  Allocation**, which is more expensive than Stack Allocation.

## Stack vs. Heap Allocation is an Implementation Issue, not a Language Issue

- Stack Allocation was a conscious decision on part of language designers due
  to its implementation efficiency, even though it compromises on language
  elegance and limits expressiveness.

- As long as the basic values being manipulated are numbers and individual
  characters, stack allocation has proved to be remarkably expressive, but
  it breaks down for non-primitive values such as arrays, and dynamic strings.

- Non-stack allocation gained importance for complex programs, e.g. in
  object-oriented style, most objects are heap allocated rather than stack
  allocated. But heap allocation is inefficient. 

## The primary contribution of this paper - Lazy Allocation

- The approach, of using Non-stack allocation is throwing away the baby with the bathwater.
  For a vast majority of usecases, Stack Allocation LIFO ordering is still
  a perfectly acceptable scheme. For those allocations, we would still prefer
  to use Stack Allocation, while switching to Heap Allocation for exceptional cases.

- Therefore, the core requirement is: provide stack allocation by default for all objects
  and reserve the more expensive heap allocation for only those objects that require it.

- The problem is determining which objects obey LIFO symantics and can be stack
  allocated, because it depends on a particular pattern of function calls, which
  in turn depends on input data.

- While most of research has been done on compile time techniques which are
  complex and expensive, this paper presents **Lazy Allocation**, which acts more
  like cache or virtual memory, where heap usage is determined during run time.

- **Function Values**: While lazy allocation already provides for stack-allocation
  of arguments and temporaries, a programmer can extract even better performance
  by utilizing "Continuation-Passing Style" to stack-allocate function results.


## The Lazy Allocation Model {TODO: NEEDS CLEANUP AND EXPANSION}

- In the lazy Allocation model we have:

	- A stack formatted into stack frames: represents execution state of an invocation of a procedure
	- A global heap : contains objects which cannot be allocated/deallocated in LIFO manner
	- small number of machine registers

- In languages like C and PL/I, the programmer can create references to local
  variables and pass them to another procedure by reference. **passing by
  reference** is more efficient than **passing by value** but it can lead to
  dangling pointers.

- A **dangling reference** is caused when a reference into the stack is stored
  into a variable having larger scope, such as a global variable, and is then
  deferenced outside of stack object's lifetime. This can lead to bugs.

- The lazy allocation model detects these objects whose reference are about to
  escape from the lifetime of their enclosing stack frame, and relocates these
  objects into the permanent heap. This is called **eviction**.

- **Ordering Relation**: object addresses adhere to the following axioms:
  - The location address of an object in the global heap compares lower than
  the location address of an object in the stack
  - The location address of an object in a frame near the stack top compared
  greater than the location address of an object in a frame near the base of
  the stack.

- **Temporary Restraining Order**: No temporary object can be pointed to by an
  object whose location address compares lower i.e., no object in the heap can
  point into the stack, and no object in the stack can point "up" the stack, only
  "down" the stack.

- The TRO is enforced by checking every primitive operation which could
  possibly violate it. The only operation which can violate TRO is the storage of
  a pointer into a stack frame (or the heap) which compares lower than the object
  pointed at. In other words, when performing an assignment, we need only check
  that the ordering is preserved:

  `component (p) := q; /* p,q pointers; assert (P>=q) */`

- e.g. by assigning a pointer to a global variable, we're very likely to
  violate TRO because lazy allocation will allocate everything to stack.

- When a TRO violation is about to take place, the system executes a
  **transporter trap**, which evicts the target object of the pointer from the
  stack into the heap.

- Theorem: If a frame from a well-ordered stack is popped, and the registers
  contain no references to the popped frame, then no dangling references are
  created by the pop.

## {TODO: MORE INFORMATION IS NEEDED HERE}

## Stack Allocation where the size of the value is not known at compile time

- A big problem with stack allocation is that we can only allocate values on the
  stack where the amount of memory needed is known at compile time. Let's say we
  have to allocate space for a linked list where the number of nodes is available
  only at runtime. We will need to write something like this -

  ```c
  // Number of nodes is only available at runtime
  int number_of_nodes = some_call_to get_a_number();

  // Each node of the linked list is called a `cons_cell`
  // We can allocate a single cons_cell on the stack 
  cons_cell cell1;
  // But there's no way to allocate a variable number of cells directly.
  ```

  We can try wrapping the cell creation in a function and calling it multiple times.

  ```c
  // Make cells at runtime
  for(i=0;i<number_of_nodes; i++) {
    x = mkCell();
    // Do something with this cell
  }
  ```

  However, that would cause unnecessary call-by-value copying of memory.

  ```c
  // Memory allocation
  cons_cell mkCell() {
    // Create a cell
    cons_cell cell1;
    // However, upon return, the stack is rewinded and memory is freed automatically!
    // So the compiler has to reallocate memory in the parent stack frame.
    // This is extremely inefficient!
    return cons_cell;
  }
  ```

  We can't use call-by-reference, because as explained, the memory allocated disappears on return.

## Lazy Allocation to the rescue!

- With lazy allocation, the programmer can use Stack allocation, and the runtime will automatically evict
  the values to the Heap as needed. This allows stack allocation even when the memory size is
  not known at compile time.

## The Paper Title: Cons should not Cons its arguments

- The terminology is obscure now so it’s hard to say what it exactly meant, but
  this is what we think - let's take the context of function arguments
  which are usually passed as records instead of a lispy linked list, which is
  where “cons” (and car and cdr etc) comes from.
  
- So the first “cons” in the title refers to cons of lisp lists. The paper says
  that lists (with cons) were not used for function arguments in lisp because
  lisp lists don’t have a fixed size and are allocated on the heap. This heap
  allocation is what is the second “cons” refers to. It basically is saying that
  the list cons function should not allocate on the heap.
  
- The paper’s “lazy allocation” method allows variable size allocation on the
  stack, so the list cons can now allocate args on the stack. This makes it
  possible to pass function args as lists, since function args have to be on the
  stack.
  
- So the title puns on the two meanings of cons, the first is the consing of
  lists, it’s not really comparable to data constructors in Haskell unless you
  specifically mean haskell lists or some other recursive data structure like
  trees. Because if the data structure is not recursive then it will have a fixed
  size which is anyways okay to allocate on the stack.


# Part II: Cheney on the MTA

- Lazy Allocation is closely related to the problem of implementing tail-recursion. Since tail-recursion
  in a language which doesn't support it, usually leads to a stack overflow. Since tail-calls do NOT
  rewind the stack!

- So we are stuck in a situation where we want to avoid stack rewinds to prevent memory from being deallocated
  prematurely, but also want to rewind the stack to avoid the stack from running out of space.

- Part II of the same paper, dubbed "Cheney on the MTA", talks about solving the tail-recursion problem,
  AND the stack rewinding problem with a scheme known as Cheney on the MTA.

## Recursive nested function calls to the rescue!

- One way is to avoid stack rewinds is to NEVER return, and never rewind the stack.

  ```c
  // Do something with a list of 10 nodes
  mkLinkedListAndDoSomething(nullPointer, 10);

  // Takes the number of nodes to allocate, and the parent node
  // No valid return value!
  void mkLinkedListAndDoSomething(cons_cell parent, int n) {
    // If we have created enough nodes
    if(n<=0) {
      do_something_with_the_list_created(parent);
    }
    else {
      // Create a cell
      cons_cell child;
      // Attach it to the chain
      child->next = parent;
      // Recurse to create the remaining ones
      mkLinkedList(child, n-1);
    }
  }
  ```

  But what if we want to do something other than `do_something_with_the_list_created()` after the memory is allocated?

  "What we want to do after an operation" is also known as a **Continuation**. We can pass in the desired continuation
  to the function itself.

  ```c
  void mkLinkedListAndDoThis(void (*cont)(cons_cell), cons_cell parent, int n) {
    // If we have created enough nodes
    if(n<=0) {
      continuation(parent);
    }
    ...
  }
  ```

  Converting your code to pass continuations around, rather than using the C call and return
  convention is called CPS or **Continuation-Passing-Style**.

- The CPS style also gets rid of the problem with tail-recursion. Since we now don't need to
  implement a **trampoline** to rewind the stack.

- Cheney on the MTA solves both problems by "allocating all frames in the
  (garbage-collected) heap. The Scheme program is translated into
  continuation-passing style, so the target C functions never return. The C stack
  pointer then becomes the allocation pointer for a Cheney-style copying garbage
  collection scheme. Our Scheme can use C function calls, C arguments, C
  variable-arity functions, and separate compilation without requiring complex
  block-compilation of entire programs."

# END

