# Heap Management — A One-Page Explainer

> _Concepts explained using everyday life in Ghana._

---

## Memory in Two Places: Stack vs Heap

When a C program runs, it has two main places to store data in RAM.

**The Stack** works like a tro-tro: passengers (function calls) board in order and leave in reverse order — last on, first off. The OS manages this automatically. Every time you call a function, a *stack frame* is pushed on top with that function's local variables. When the function returns, its frame is popped off and the memory is freed instantly. You never have to think about it.

**The Heap** works like renting a market stall at Makola Market. Space is available, but you must walk up to the market manager (the OS), request a stall of a specific size, use it for as long as you need, and then — critically — hand it back yourself when you are done. Nobody cleans up your stall for you.

---

## What `malloc()` Does

```c
int *prices = malloc(10 * sizeof(int));
```

`malloc(n)` asks the memory manager: _"Give me n bytes on the heap."_ It searches a free-list of available blocks, marks a suitable chunk as used, and returns a **pointer** — the address of your stall in memory. If no space is available, it returns `NULL`.

You later release that memory with:

```c
free(prices);
```

Forgetting to call `free()` is a **memory leak** — your stall stays occupied forever, crowding out other processes, just like a trader who never vacates their Makola stall even after closing.

---

## How the Heap Gets More Space: `sbrk()` and `mmap()`

The heap is not infinite. When `malloc()` runs out of room, it asks the OS for more memory using one of two system calls:

| System call | What it does | Analogy |
|---|---|---|
| `sbrk(n)` | Moves the *program break* pointer upward by n bytes, extending the heap | The market authority extending the market's boundary fence to add more stalls |
| `mmap()` | Maps a brand-new, independent region of memory anywhere in virtual address space | The market authority opening an entirely new market branch in a different neighbourhood |

Modern allocators (like `glibc`'s `malloc`) prefer `mmap()` for large allocations (≥ 128 KB by default) because those regions can be returned to the OS immediately via `munmap()`. Smaller allocations use `sbrk()`-style heap growth, managed internally with a free-list.

---

## The Lifecycle of a Heap Allocation

```
Program starts
    │
    ▼
malloc(size) called
    │
    ├─► Space available in free-list? ──► Return pointer
    │
    └─► No space ──► Call sbrk() or mmap() ──► Return pointer
    │
    ▼
Program uses the memory via the pointer
    │
    ▼
free(pointer) called
    │
    ▼
Block returned to free-list (heap memory stays in process until exit)
```

---

## Key Rules to Remember

1. **Every `malloc` needs a `free`.** Unfreed memory is a leak.
2. **Never use memory after `free()`**. This is *use-after-free*, a dangerous bug.
3. **Never `free()` the same pointer twice**. Double-free corrupts the heap.
4. **Check for `NULL`**. If `malloc()` returns `NULL`, allocation failed — using that pointer will crash the program.

---

## Summary Table

| Concept | Location | Managed by | Speed | Size |
|---|---|---|---|---|
| Stack | RAM | CPU / OS automatically | Very fast | Fixed, small (~8 MB) |
| Heap | RAM | Programmer explicitly | Slower | Large, limited by RAM |
| `malloc()` | — | C standard library | — | Allocates heap block |
| `sbrk()` | — | OS kernel | — | Extends heap segment |
| `mmap()` | — | OS kernel | — | Maps new memory region |

---

_Written as a beginner's guide to heap management in C._
