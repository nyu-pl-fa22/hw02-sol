## Solution to Problem 1

Part 1:

The variables are bound as follows:
* line 6: `x` -> line 7, `y` -> line 4
* line 11: `x` -> line 9, `y` -> line 4
* line 15: `x` -> line 2, `y` -> line 13

The compiler would produce a static semantic error because the
occurrence of `x` on line 4 refers to the declaration on line 9 in
the same block. Thus `x` on line 4 is used before its declaration,
which is not allowed. A similar violation is on line 6.

Part 2:

The variables are bound as follows:
* line 6: `x` -> line 2, `y` -> line 4
* line 11: `x` -> line 9, `y` -> line 4
* line 15: `x` -> line 2, `y` -> line 13

The program will be accepted by the compiler.
The program first executes the print statements on line 6 when
`inner` is called (line 10) within `middle` (line 14). This prints `2`
and `2`. Then the print statements on line 11 are executed after
`inner` returns, printing `1` and `2`. Finally, the print statements
on line 15 are executed printing `2` and `3`.


## Solution to Problem 2

* Static scoping: In this case, the program prints the sequence `2`,
  `2`, `3`, `3`. This is because with static scoping, the occurrences
  of `x` in `set_x` and `print_x` always refer to the global
  declaration of `x` in the first line. When the first `print_x` call
  within `first` executes, the global variable `x` has just been
  reassigned to value `2`. The second call to `print_x` in `main`
  after `first` returns, then again prints the current value of the
  global `x` which is still `2`. In `second`, the call to `set_x` sets
  the value of global `x` to `3`. So the following print statements
  then print that new value.
  
* Dynamic scoping: In this case, the program prints the sequence `2`,
  `2`, `3`, `2`. Now, the occurrence of `x` in `set_x` and `print_x`
  will always refer to the most recent declaration of `x` that is
  still alive during program execution. For the first two calls to
  `set_x` and `print_x`, this is the global variable `x` declared on
  the first line. These calls therefore behave just like in the case
  for static scoping. However, when the calls in `second` are
  processed, the new declaration of `x` in `second` is in the dynamic
  scope. Hence, the call to `set_x` within `second` sets the local
  variable `x` in that function to `3` but leaves the value of the
  global `x` unchanged (value `2`). The subsequent call to `print_x`
  in `second` prints the variable of the local `x` (value
  `3`). After `second` returns to `main`, the declaration of `x` in the
  dynamic scope is again the global `x`, which still holds value
  `2`. The final call to `print_x` thus prints `2`.


## Solution to Problem 3

It follows from `C`'s static scoping rules that there are at most 5
variables simultaneously in scope at any time during execution of the
code. Hence, 20 bytes are sufficient to store these variables.

## Solution to Problem 4

1. The problem with the code is that the call `foo(n2)` deallocates
   the `Node` value on the heap pointed to by `n2->next`. This struct
   value is also pointed to by `n1`. Hence, after the call to `foo`,
   `n1` is a dangling pointer. The assignment `n1->data = 42`
   dereferences this dangling pointer, which is unsafe and means that
   the program has undefined behavior according to the semantics of
   C++. In particular, the program may crash at this point because
   it tries to access invalid memory.
   
   Now why does the program likely proceed without crashing and print
   the value 42? The reason for this is that the call `create(3,
   NULL)` allocates a new `Node` value on the heap. The underlying
   implementation of the memory allocation mechanism will likely reuse
   the memory that has just been freed when deallocating `n1->data`
   for the new allocation request. Thus, after `n3` has been
   initialized, both `n3` and `n1` will point to the same memory
   address. That is, after the call to `create`, `n1` will again point
   to valid memory and the assignment `n1->data = 42` will proceed
   without problems. Since `n1` and `n3` are aliased, the update of
   the `data` field of the `Node` value through `n1` will also be
   observable through `n3` which is why `n3->data` will be `42` when
   the print statement executes.
   
   Note that this behavior is by no means guaranteed. It depends on
   the specific implementation details of the memory allocator as well
   as other extraneous circumstances that are outside of the control
   of the programmer. For instance, if `main` is executed concurrently
   with other threads, then an allocation request of another thread
   may be interleaved between the calls to `foo` and `create` causing
   `create` to use fresh memory rather then reuse the memory of
   `n2->next` that was just freed.

2. Since in this case the call to `create` happens before the deletion
   of `n2->next`, `n2->next` is still allocated when the new `Node`
   value is allocated inside of `create`. Thus, the memory used for
   this allocation is guaranteed to be different from the one pointed
   to by `n2->next`. Hence, after the calls to `create` and `foo`,
   `n1` and `n3` will definitely not be aliased. It follows that if
   the assignment `n1->data` proceeds without crashing, then
   `n3->data` will likely still hold the initial value `3` assigned in
   `create`. Thus, the program will likely print `3` in this case. I
   say "likely" here because `n1` is still dangling after `foo`
   returns. Hence, the behavior of the assignment to `n1->data` is
   still undefined and could in principle change the value of
   `n3->data`.
