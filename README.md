This is an implementation of the ideas in the essay by floooh called: [Handles are the better pointers](https://floooh.github.io/2018/06/17/handles-vs-pointers.html).

The idea is to use an alternative to pointers and heaps for managing memory, which I'm calling *type segregated pools*. These are fixed-size arrays that hold elements of a fixed type. Instead of passing pointers around, you pass handles around that represent indices into these arrays. The style of programming with *type segregated pools* is similar to typical `malloc()` and `free()` based programming in C, but allocates and frees from pools of the corresponding type. The essay proposes a heuristic for catching *use-after-free errors* and even *memory leaks*.
  - For use-after-free detection, it proposes to attach a uniqueness ID to each handle, and to each element of the pool arrays. If these don't match when a user dereferences a handle then this proves that a use-after-free error has taken place.
  - If more memory is allocated then there is room in the array, then this could indicate a memory leak has taken place.
It also claims better cache locality than C++'s smart pointers.

The implementation I've designed is *exception-safe*; this is important in any language with exception-handling, like C++. It uses `with` statements to accomplish this; in C++ you would use destructors.

This is an example of how to use this:
.python
    myPool = Pool[int](100, int)
    myHandle = myPool.alloc()
    print(myPool[myHandle])
    myPool.free(myHandle)
    
    floatPool = Pool[float](100)
    strPool = Pool[str](100)
    with ScopedHandle(floatPool, lambda: 1.0) as float_handle,\
         ScopedHandle(strPool, lambda: "blabla") as str_handle:
             print(floatPool[float_handle])
             print(strPool[str_handle])