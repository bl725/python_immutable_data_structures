# Immutable Data Structures in Python

In Python, when you want an object that behaves like a tuple (immutable, ordered, indexed access, iterable) but also allows accessing its elements by name, you're looking for a **named tuple** or something functionally equivalent.

Here are the primary ways to define such a "structure sequence" in Python, ranging from standard library options to more advanced alternatives:

### 1. `collections.namedtuple` (Python 2.6+)

This is the classic and most direct way to create a named tuple. It's a factory function that returns a new tuple subclass.

**Characteristics:**
*   **Immutable:** Once created, its values cannot be changed.
*   **Named Access:** Access fields using dot notation (e.g., `point.x`).
*   **Indexed Access:** Access fields using integer indices (e.g., `point[0]`).
*   **Iterable:** Can be unpacked or iterated over like a regular tuple.
*   **Lightweight:** Low memory footprint, good performance.
*   **No default values:** All fields must be provided during instantiation.

**Example:**

```python
from collections import namedtuple

# 1. Define the named tuple
#    - 'Point': The name of the new class
#    - ['x', 'y']: A list of field names
#    - Alternatively, 'x y' (space-separated string) works too
Point = namedtuple('Point', ['x', 'y'])

# 2. Create instances
p1 = Point(10, 20)
p2 = Point(y=30, x=40) # Can use keyword arguments

# 3. Access elements
print(f"p1: {p1}")        # Output: p1: Point(x=10, y=20)
print(f"p1.x: {p1.x}")    # Output: p1.x: 10
print(f"p1[0]: {p1[0]}")  # Output: p1[0]: 10

# 4. Immutability
try:
    p1.x = 15
except AttributeError as e:
    print(f"Error: {e}") # Output: Error: can't set attribute

# 5. Iteration and Unpacking
x, y = p1
print(f"Unpacked: x={x}, y={y}") # Output: Unpacked: x=10, y=20

# 6. Useful methods
print(f"Fields: {p1._fields}") # Output: Fields: ('x', 'y')
print(f"As dictionary: {p1._asdict()}") # Output: As dictionary: OrderedDict([('x', 10), ('y', 20)])
```

**When to use:**
*   You need a simple, immutable data structure with named fields.
*   You don't need custom methods, default values, or complex behavior.
*   Backward compatibility with older Python versions is a concern.

---

### 2. `typing.NamedTuple` (Python 3.6+)

This is the type-hinted equivalent of `collections.namedtuple`, defined using class syntax. It's essentially a syntactic sugar around `collections.namedtuple` that integrates well with type checkers.

**Characteristics:**
*   All benefits of `collections.namedtuple` (immutable, named/indexed access, lightweight).
*   **Type Hinting:** Supports explicit type annotations for fields, improving code clarity and enabling static analysis.
*   **Default Values:** Can specify default values for fields (but order matters: fields with defaults must come after fields without).

**Example:**

```python
from typing import NamedTuple

# 1. Define the NamedTuple using class syntax
class Point(NamedTuple):
    x: int
    y: int
    z: int = 0  # Field with a default value

# 2. Create instances
p1 = Point(10, 20)
p2 = Point(x=40, y=50, z=60)
p3 = Point(100, 200) # z defaults to 0

# 3. Access elements (same as collections.namedtuple)
print(f"p1: {p1}")        # Output: p1: Point(x=10, y=20, z=0)
print(f"p1.x: {p1.x}")    # Output: p1.x: 10
print(f"p1[0]: {p1[0]}")  # Output: p1[0]: 10
print(f"p3.z: {p3.z}")    # Output: p3.z: 0

# Type checking will work:
# p4 = Point("hello", 20) # A type checker would flag this!
```

**When to use:**
*   You need the benefits of `collections.namedtuple`.
*   You want to leverage Python's type hinting for better code quality and maintainability.
*   You need default values for some fields.
*   You are on Python 3.6 or newer.

---

### 3. `dataclasses` with `frozen=True` (Python 3.7+)

`dataclasses` are a powerful way to create classes primarily for storing data. By setting `frozen=True`, you can make them immutable, similar to named tuples. However, they are *not* inherently sequence-like (you can't access elements by index `obj[0]`) unless you explicitly implement `__getitem__`, `__len__`, etc.

**Characteristics:**
*   **Immutable (if `frozen=True`):** Prevents modification after instantiation.
*   **Named Access:** Access fields using dot notation.
*   **Type Hinting:** Excellent integration with type hints.
*   **Default Values:** Easy to define default values.
*   **Custom Methods:** Can define your own methods.
*   **Richer Features:** Automatically generates `__init__`, `__repr__`, `__eq__`, etc.
*   **Not a sequence by default:** You cannot index `d[0]`. If you need this, you'd have to add `__getitem__`, `__len__`, `__iter__` manually.

**Example:**

```python
from dataclasses import dataclass

# 1. Define the dataclass with frozen=True
@dataclass(frozen=True) # frozen=True makes instances immutable
class Point:
    x: int
    y: int
    z: int = 0 # Field with a default value

    # You can add custom methods
    def magnitude(self) -> float:
        return (self.x**2 + self.y**2 + self.z**2)**0.5

# 2. Create instances
p1 = Point(10, 20)
p2 = Point(x=40, y=50, z=60)
p3 = Point(100, 200)

# 3. Access elements (named access only)
print(f"p1: {p1}")        # Output: p1: Point(x=10, y=20, z=0)
print(f"p1.x: {p1.x}")    # Output: p1.x: 10
print(f"p3.z: {p3.z}")    # Output: p3.z: 0

# 4. Immutability
try:
    p1.x = 15
except AttributeError as e:
    print(f"Error: {e}") # Output: Error: cannot assign to field 'x'

# 5. Use custom method
print(f"Magnitude of p2: {p2.magnitude():.2f}") # Output: Magnitude of p2: 86.60

# 6. Not sequence-like by default
try:
    print(p1[0])
except TypeError as e:
    print(f"Error: {e}") # Output: Error: 'Point' object is not subscriptable
```

**Making a `dataclass` behave like a sequence (if truly needed):**

If you *really* need both the power of `dataclasses` and sequence-like behavior (indexed access), you can manually add `__getitem__`, `__len__`, and `__iter__`. However, this is more boilerplate and generally implies `collections.namedtuple` or `typing.NamedTuple` might be a better fit if sequence behavior is primary.

```python
from dataclasses import dataclass, asdict, astuple

@dataclass(frozen=True)
class PointSequence:
    x: int
    y: int
    z: int = 0

    def __getitem__(self, item):
        return astuple(self)[item] # Convert to tuple and get item

    def __len__(self):
        return len(astuple(self))

    def __iter__(self):
        return iter(astuple(self))

p = PointSequence(1, 2, 3)
print(p[0]) # Output: 1
print(list(p)) # Output: [1, 2, 3]
```

**When to use:**
*   You need immutable data structures with named fields.
*   You want type hinting and auto-generated `__repr__`, `__eq__`, etc.
*   You need to add custom methods or properties to your data structure.
*   You are on Python 3.7 or newer.
*   You don't strictly require indexed access (or are willing to implement it manually).

---

### 4. Plain Class with `__slots__` (Manual Approach)

You can create a regular class and use `__slots__` to limit memory consumption and prevent dynamic attribute creation, making it more tuple-like in its fixed structure. To make it immutable, you would generally avoid setters or use `@property` with only a getter.

**Characteristics:**
*   **Memory Efficiency:** `__slots__` reduces the memory footprint by not creating a `__dict__` for each instance.
*   **Named Access:** Standard dot notation.
*   **No Automatic Immutability:** You have to enforce immutability yourself (e.g., by making attributes read-only after `__init__`).
*   **No Sequence-like Behavior:** You must implement `__getitem__`, `__len__`, `__iter__` manually if desired.
*   **More Boilerplate:** You write `__init__`, `__repr__`, `__eq__` yourself (or use a decorator like `functools.total_ordering`).

**Example:**

```python
class PointSlot:
    __slots__ = ('_x', '_y') # Use leading underscore for internal state

    def __init__(self, x: int, y: int):
        self._x = x
        self._y = y

    @property # Make attributes read-only externally
    def x(self) -> int:
        return self._x

    @property
    def y(self) -> int:
        return self._y

    def __repr__(self) -> str:
        return f"PointSlot(x={self.x}, y={self.y})"

    # If you want it to be sequence-like:
    def __getitem__(self, index):
        if index == 0: return self.x
        if index == 1: return self.y
        raise IndexError("PointSlot index out of range")

    def __len__(self):
        return 2

p = PointSlot(10, 20)
print(f"p: {p}")      # Output: p: PointSlot(x=10, y=20)
print(f"p.x: {p.x}")  # Output: p.x: 10
print(f"p[0]: {p[0]}") # Output: p[0]: 10

try:
    p.x = 15 # This won't work because `x` is a property without a setter
except AttributeError as e:
    print(f"Error: {e}") # Output: Error: can't set attribute 'x'

# Can't directly assign to _x either for true immutability after init,
# but it's possible if not careful.
```

**When to use:**
*   You need absolute maximum control over memory footprint and attribute access.
*   You are comfortable writing more boilerplate.
*   You want fine-grained control over mutability/immutability beyond what `dataclasses(frozen=True)` offers (though this is rare).
*   Performance in very tight loops is a critical concern, and you've profiled and found `namedtuple`/`dataclass` to be a bottleneck (also rare).

---

### Summary / When to Choose Which

*   **`typing.NamedTuple` (Recommended for most cases):**
    *   Best balance of features: immutable, named/indexed access, iterable, type-hinted, supports default values.
    *   Modern and clean syntax.
    *   Good for simple data structures where sequence behavior is a core requirement.

*   **`collections.namedtuple`:**
    *   Good for older Python versions or if you explicitly want to avoid `typing` imports.
    *   Functionally very similar to `typing.NamedTuple` but without type hints or default values in definition.

*   **`dataclasses(frozen=True)`:**
    *   Excellent choice when you need immutable objects with named fields and potentially custom methods.
    *   Strong type-hinting support and automatic method generation (`__repr__`, `__eq__`).
    *   **Caveat:** Not inherently sequence-like (no indexed access like `obj[0]`) unless you manually add `__getitem__`, `__len__`, `__iter__`. If sequence behavior is paramount, `NamedTuple` is better.

*   **Plain Class with `__slots__`:**
    *   Use when you need extreme memory optimization or very specific custom behavior that `dataclasses` can't easily handle.
    *   Requires more manual implementation for basic features. Generally overkill for simple named tuple requirements.

For the common request of "an object that behaves like a named tuple," **`typing.NamedTuple`** is often the sweet spot in modern Python. If you also need methods or more dataclass-like features and don't strictly require indexed access, `dataclasses(frozen=True)` is an excellent alternative.

