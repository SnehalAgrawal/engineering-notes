# Python core

## Overview
Most everyday Python bugs come from two things: picking the wrong data structure, and forgetting that variables are names pointing at objects. If two names point at the same list, changing the list through one name shows up through the other.

## Key concepts
*   **list**: A dynamic array. Index access, `append`, and `pop()` from the end are O(1). Insert, delete, or `pop(0)` elsewhere are O(n). `x in list` is O(n). Use `collections.deque` for fast adds and removes at both ends.
*   **tuple**: A fixed sequence that cannot change. It can be a dict key only if everything inside it is hashable.
*   **set**: Unique items, no order. `x in s` is O(1) on average.
*   **dict**: A hash map. Lookup is O(1) on average. Keeps insertion order since Python 3.7.
*   **Mutable types**: Can change in place: `list`, `dict`, `set`, most of your own classes.
*   **Immutable types**: Cannot change: `int`, `float`, `str`, `bytes`, `tuple`, `frozenset`. A tuple holding a list cannot change, but the list inside it can.
*   **Passing arguments**: Python passes references to objects. Reassigning a parameter inside a function does nothing outside. Changing a mutable argument (like `items.append(x)`) is visible outside.
*   **Pythonic habits**: Comprehensions for simple transforms, `enumerate()` instead of a manual counter, `zip()` to loop over two lists together, `a, b = b, a` to swap, `with` for files, PEP 8 style. `import this` prints the Zen of Python, a short list of style principles.
    ```python
    for i, name in enumerate(["alice", "bob"], start=1):
        print(i, name.title())   # 1 Alice, then 2 Bob
    ```

## Real-world usage
*   **Fast lookups**: Turn a list into a `set` or `dict` before checking membership many times.
*   **Comprehensions**: Usually a bit faster than a loop with `.append()`, and clearer for simple cases.
*   **Several return values**: Return a tuple, or a `NamedTuple`/dataclass when fields need names.
*   **Many database rows**: Loop with a generator instead of building a full list (see `python-advanced.md`).
*   **Web APIs**: Flask is a small WSGI framework with many extensions. FastAPI is an ASGI framework built on type hints. It validates input with Pydantic, has dependency injection, generates API docs (Swagger UI, ReDoc), and supports `async` endpoints.
*   **Scripts and CLIs**: `argparse` or `click`/`typer` for command-line options, `pathlib` and `os` for files and paths, `subprocess.run` to call other programs.

## Tradeoffs
*   **Dynamic typing**: Fast to write, but wrong types fail only at runtime. Type hints plus mypy or pyright catch many of these early.
*   **List vs tuple**: Lists keep spare room so appends are cheap. Tuples are fixed, a bit smaller, and can be dict keys.
*   **Dict vs dataclass**: Dicts are quick to build but have no fixed fields. Dataclasses give named, typed fields. They save memory only with `@dataclass(slots=True)` (3.10+); otherwise each instance still has a `__dict__`.

## When not to use
*   **Mutable default arguments**: Defaults are created once, when `def` runs. `def f(items=[])` shares one list across all calls. Use `items=None` and create the list inside.
*   **Plain lists for heavy math**: Use NumPy. It stores numbers in compact arrays and loops in C.

## Interview focus
*   **Mutable default trap**:
    ```python
    def add_to(item, storage=[]):
        storage.append(item)
        return storage

    add_to(1)  # [1]
    add_to(2)  # [1, 2], same list both times
    ```
*   **List vs set lookup**: O(n) vs O(1) on average. A list is better when you need order, duplicates, or indexing.
*   **Shallow vs deep copy**: `copy.copy(x)` (or `x.copy()`, `x[:]`) makes a new outer list that shares the inner objects. `copy.deepcopy(x)` copies everything inside too.
*   **`is` vs `==`**: `is` checks "same object". `==` checks "equal value". Use `is` only for `None` and similar singletons.

## Common mistakes
*   **`+=` on strings in a loop**: Can build a new string each time. Collect parts in a list and use `''.join(parts)`.
*   **Changing a collection while looping over it**: A list skips items. A dict raises `RuntimeError` if its size changes. Loop over a copy instead.
*   **`[[0] * 3] * 3`**: Creates three references to the same inner list. Use `[[0] * 3 for _ in range(3)]`.
*   **Too many globals**: Hidden shared state makes code hard to test. Pass arguments instead.
