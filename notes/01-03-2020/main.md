Cons Should Not Cons Its Arguments
==================================

## Paper Link

[CONS Should not CONS its Arguments, or, a Lazy Alloc is a Smart Alloc]( 
    https://www.cs.tufts.edu/comp/150FP/archive/henry-baker/cons-lazy-alloc.pdf).

## Introduction
- **Stack Allocation** of objets is elegant, efficient and can handle the great
  majority of object allocations.

- High Level languages like Algol, Pascal perform all storage using stack, and
  all non-stack allocation is left for the programmer.

- Stack Allocation, however is limited because of strict LIFO allocation /
  deallocation ordering. This restricts some behaviours (e.g. returning a
  functional argument as a result (talked about later)).

- Languages such as Lisp, SmallTalk, ML etc. seek to escape these LIFO
  restrictions to gain more expressive power while retaining simplicity and
  elgance. But increased flexibility comes at the cost of increased use of **Heap
  Allocation** for objects. Heap Allocation is more expensive than Stack
  Allocation.

- But, vast majority of objects still obey the straight forward LIFO allocation,
  hence can still continue to use Stack Allocation.

- Therefore, the task is: provide stack allocation and reserve the more
  expensive heap allocation for only those objects that require it.

- The problem is determining which objects obey LIFO symantics and can be stack
  allocated, because it depends on a particular pattern of function calls, which
  in turn depends on input data.

- While most of research has been done on compile time techniques which are
  complex and expensive, here comes **Lazy Allocation**, which acts more like
  cache or virtual memory, whose usage is determined during run time.

- While lazy allocation already provides for stack-allocation of arguments and
  temporaries, a programmer can extract even better performance by utilizing
  "continuation-passing style" to stack-allocate funtion results.

## Stack Allocation is an Implementation Issue, not a Language Issue
- Stack Allocation was a conscious decision on part of language designers due
  to its implementation efficiency, even though it compromises on language
  elegance and limits expressiveness.

- As long as the basic values being manipulated are numbers and individual
  characters, stack allocation proved to remarkably expressive. But it breaks
  down for objects such as arrays, dynamic strings.

- Non-stack allocation gained importance for complex programs, e.g. in
  object-oriented style, most objects are heap allocated rather than stack
  allocated. But heap allocation is ineffiencient. 

## Model
- In lazy Allocation model we have:
	- stack formatted into stack frames: represents execution state of an invocation of a procedure
	- a global heap : contains objects which cannot be allocated/deallocated in LIFO manner
	- small number of machine registers

- In languages like C and PL/I, the programmer can create references to local
  variables and pass them to another procedure by reference. **passing by
  reference** is more efficient than **passing by value** but it can lead to
  dangling pointers.

- A **dangling reference** is caused when a reference into the stack is stored
  into a variable having larger scope, such as a global variable, and is then
  deferenced outside of stack object's lifetime. This can lead to bugs.

- The lazy allocation model detects these objets whose reference are about to
  escape from the lifetime of their enclosing stack frame, and relocates these
  objects into the permanent heap. This is called **eviction**.

- **Ordering Relation**: object addresses adhere to the following axioms:
  - The location address of an object in the global heap compares lower than
  the location address of an objet in the stack
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

