# Python Patterns for LangChain

Klement's reference guide — code + explanation, built together pattern by pattern.

---

## Roadmap — All 12 Patterns

| # | Pattern | Status |
|---|---------|--------|
| 1 | Classes + OOP | Complete |
| 2 | Type Hints (int, str, bool, list, dict, Optional, Union, Literal) | Complete |
| 3 | TypedDict — agent state | Complete |
| 4 | Pydantic BaseModel — tool input validation | Complete |
| 5 | Decorators + `@tool` | Complete |
| 6 | async/await — parallel execution | Complete |
| 7 | try/except — error handling | Complete |
| 8 | Dataclasses | Complete |
| 9 | `**kwargs` — flexible arguments | Complete |
| 10 | List comprehensions | Complete |
| 11 | Annotated types | Complete |
| 12 | Putting it all together — full LangChain agent | Complete |

---

## Pattern 1 — Classes + OOP

A class is a **blueprint**. An object is the **thing built from that blueprint**.

Every house on a street was built from the same blueprint — but each house is
its own building, with its own address, its own people inside.

In Python: one class, many objects. Each object holds its own data.

---

### Level 1 — Basic class

```python
class Dog:
    def __init__(self, name, breed):
        self.name = name
        self.breed = breed

    def greet(self):
        print(f"Hi, I am {self.name} and I am a {self.breed}!")

dog = Dog("Rex", "Labrador")
dog.greet()
```

**Output:**

```text
Hi, I am Rex and I am a Labrador!
```

**Line by line:**

- `class Dog:` — defines the blueprint named Dog
- `def __init__(self, name, breed):` — runs automatically the moment you create a Dog.
  Think of it as the "setup" step. `self` means "this specific dog I am creating right now."
- `self.name = name` — stores the name inside this dog object
- `self.breed = breed` — stores the breed inside this dog object
- `def greet(self):` — a method (action) this dog can do
- `dog = Dog("Rex", "Labrador")` — creates one dog: name=Rex, breed=Labrador
- `dog.greet()` — calls the action on that dog

**`self`** = "me, this specific object." Every method gets `self` as the first
parameter so it can access its own data.

---

### Level 2 — Methods with return values

```python
class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def area(self):
        return self.width * self.height

    def perimeter(self):
        return 2 * (self.width + self.height)

r = Rectangle(5, 3)
print(r.area())       # 15
print(r.perimeter())  # 16
```

**Output:**

```text
15
16
```

**What changed from Level 1:** Methods now `return` a value instead of just
printing. The method calculates something and gives it back to you. You can
store it, print it, or use it in another calculation.

- `r.area()` → returns 15 (5 × 3)
- `r.perimeter()` → returns 16 (2 × (5 + 3))

---

### Level 3 — Two independent objects (same class, own state)

```python
class BankAccount:
    def __init__(self, owner):
        self.owner = owner
        self.balance = 0

    def deposit(self, amount):
        self.balance += amount

    def get_balance(self):
        return self.balance

account1 = BankAccount("Klement")
account2 = BankAccount("Nathan")

account1.deposit(500)
account2.deposit(100)

print(f"Klement: ${account1.get_balance()}")  # 500
print(f"Nathan:  ${account2.get_balance()}")  # 100
```

**Output:**

```text
Klement: $500
Nathan:  $100
```

**Key lesson:** `account1` and `account2` are two separate objects. They were
both built from the same `BankAccount` class — but they each hold their own
`balance`. Depositing into `account1` does not touch `account2`.

Same blueprint. Independent state. They do not share memory.

---

### Level 4 — Inheritance

**Inheritance** = a child class gets everything from a parent class.
The child can also **override** (redefine) any method from the parent.

```python
class Animal:
    def __init__(self, name):
        self.name = name

    def speak(self):
        print("...")

class Dog(Animal):
    def speak(self):
        print(f"{self.name} says: Woof!")

class Cat(Animal):
    def speak(self):
        print(f"{self.name} says: Meow!")

dog = Dog("Rex")
cat = Cat("Luna")

dog.speak()  # Rex says: Woof!
cat.speak()  # Luna says: Meow!
```

**Output:**

```text
Rex says: Woof!
Luna says: Meow!
```

**Line by line:**

- `class Dog(Animal):` — Dog inherits from Animal. The `(Animal)` means "get
  everything Animal has."
- Dog gets `__init__` for free from Animal — no need to rewrite it
- Dog **overrides** `speak()` with its own version
- When you call `dog.speak()`, Python looks at Dog first — finds the override —
  uses that. Never touches Animal's speak.

**Overriding** = child redefines a method the parent already has.

---

## Pattern 2 — Type Hints

A type hint is a **label** you add to a function to say what type of value goes
in and what comes out.

```python
# Without type hints
def greet(name):
    return "Hello, " + name

# With type hints
def greet(name: str) -> str:
    return "Hello, " + name
```

The code runs exactly the same. The labels don't do anything at runtime.
They are information — for you, for your IDE, and for LangChain.

**Three parts:**

| Syntax | Meaning |
|--------|---------|
| `a: int` | parameter `a` should be an integer |
| `-> int` | this function returns an integer |
| `-> None` | this function returns nothing |

---

### Basic types — int, str, bool

```python
def add(a: int, b: int) -> int:
    return a + b

def greet(name: str) -> str:
    return f"Hello, {name}!"

def is_adult(age: int) -> bool:
    return age >= 18

print(add(3, 4))        # 7
print(greet("Klement")) # Hello, Klement!
print(is_adult(25))     # True
print(is_adult(15))     # False
```

**`bool`** is the type for `True` or `False`. When you write `-> bool`, the
function will always give back yes or no.

**Are type hints enforced?** No. Python does not crash if you pass the wrong
type. They are labels only. If you pass a string to an `int` parameter, Python
just runs.

---

### list and dict

```python
def total(scores: list[int]) -> int:
    return sum(scores)

def get_grade(scores: dict[str, int]) -> str:
    average = sum(scores.values()) / len(scores)
    if average >= 90:
        return "A"
    return "B"

print(total([90, 85, 92]))                              # 267
print(get_grade({"Math": 95, "English": 88}))           # A
```

- `list[int]` — a list where every item is a whole number
- `dict[str, int]` — a dictionary where keys are strings and values are integers

---

### Optional — value or nothing

**Problem:** Sometimes a parameter is required. Sometimes it is not.
`Optional[str]` means: "this slot can hold text, or it can be empty (None)."

Think of it like a sticky note on a package. Sometimes there is a note.
Sometimes the slot is blank. Either way the package is valid.

```python
from typing import Optional

def book_cab(pickup: str, destination: str, note: Optional[str] = None) -> str:
    print(f"note is: {note}")
    if note:
        return f"Cab booked: {pickup} → {destination}. Note: {note}"
    return f"Cab booked: {pickup} → {destination}"

print(book_cab("Nacharam", "Airport"))
print(book_cab("Nacharam", "Airport", "Call mom when arriving"))
```

**Output:**

```text
note is: None
Cab booked: Nacharam → Airport
note is: Call mom when arriving
Cab booked: Nacharam → Airport. Note: Call mom when arriving
```

- First call: no note passed → `note` is `None` → if skips → simple output
- Second call: note passed → if runs → note appears

`None` in Python = nothing. Empty. Blank.
`= None` after the type hint sets the default — if you do not pass it, it
defaults to nothing.

---

### Union — one type OR another

```python
from typing import Union

def display(value: Union[int, str]) -> str:
    print(f"value is: {value}")
    return f"Received: {value}"

print(display(42))
print(display("hello"))
```

**Output:**

```text
value is: 42
Received: 42
value is: hello
Received: hello
```

`Union[int, str]` = "this can be a whole number OR text — either is fine."

**Optional vs Union:**

| | What it allows |
|---|---|
| `Optional[str]` | string **or None** |
| `Union[int, str]` | int **or** string |

`Optional[str]` is actually a shortcut for `Union[str, None]`. Same thing.

---

### Literal — must be one of these exact values

```python
from typing import Literal

def set_mode(mode: Literal["read", "write", "admin"]) -> str:
    print(f"mode is: {mode}")
    return f"Mode set to: {mode}"

print(set_mode("read"))
print(set_mode("admin"))
# set_mode("delete") → rejected by Pydantic
```

**Output:**

```text
mode is: read
Mode set to: read
mode is: admin
Mode set to: admin
```

`Literal["read", "write", "admin"]` = "must be exactly one of these three
strings — nothing else allowed."

If you pass `"delete"`, LangChain/Pydantic will reject it immediately. The
agent gets the error, corrects itself, and tries again with a valid value.
This is how you restrict an agent to only valid choices.

---

### Why LangChain cares about type hints

When you decorate a function with `@tool` (Pattern 5), LangChain reads the
type hints and builds the schema automatically:

```
name → what to pass in
return type → what to expect back
```

The agent (Claude/GPT) reads that schema, fills in the values, Pydantic
validates them, and your function runs. Without type hints, LangChain cannot
build the schema. The tool breaks.

**All type hints — quick reference:**

| Type | Meaning |
|------|---------|
| `int` | whole number — `42` |
| `str` | text — `"hello"` |
| `bool` | true or false |
| `float` | decimal — `3.14` |
| `list[int]` | list of integers |
| `dict[str, int]` | dict with string keys, int values |
| `Optional[str]` | string or None |
| `Union[int, str]` | int or string |
| `Literal["a", "b"]` | must be exactly "a" or "b" |

---

## Pattern 3 — TypedDict

A regular dict has no structure. Any key, any type, no rules:

```python
person = {"name": "Klement", "age": 25}  # Python has no idea what's inside
```

**TypedDict** gives the dict a declared shape. You say exactly what keys exist
and what type each key holds. Python (and LangChain) now knows the structure.

```python
from typing import TypedDict

class Person(TypedDict):
    name: str
    age: int
```

**Used for:** Agent state in LangGraph — the memory that flows between nodes.
You write it. You control it. No enforcement needed.

---

### Basic structure

```python
from typing import TypedDict

class Person(TypedDict):
    name: str
    age: int

def greet(person: Person) -> str:
    return f"Hello {person['name']}, you are {person['age']} years old."

klement = {"name": "Klement", "age": 25}
print(greet(klement))
```

**Output:**

```text
Hello Klement, you are 25 years old.
```

Access TypedDict values with `["key"]` — same as a regular dict.

---

### Real scenario — cab booking state

```python
from typing import TypedDict

class CabBooking(TypedDict):
    pickup: str
    destination: str
    seats: int
    confirmed: bool

def summarize(booking: CabBooking) -> str:
    status = "Confirmed" if booking["confirmed"] else "Pending"
    return (
        f"Pickup: {booking['pickup']}\n"
        f"Destination: {booking['destination']}\n"
        f"Seats: {booking['seats']}\n"
        f"Status: {status}"
    )

order = {"pickup": "Nacharam", "destination": "Airport", "seats": 2, "confirmed": True}
print(summarize(order))
```

**Output:**

```text
Pickup: Nacharam
Destination: Airport
Seats: 2
Status: Confirmed
```

The function knows exactly what keys to expect. No guessing. No wrong key names.

---

### LangGraph node — state in, state out

This is exactly how every LangGraph node works.

```python
from typing import TypedDict

class AgentState(TypedDict):
    message: str
    done: bool
    reply: str

def process(state: AgentState) -> AgentState:
    print(f"Received: {state['message']}")
    state["reply"] = f"Got it: {state['message']}"
    state["done"] = True
    return state

state = {"message": "Book a cab for Klement", "done": False, "reply": ""}

print("--- Before ---")
print(state)

result = process(state)

print("--- After ---")
print(result)
```

**Output:**

```text
--- Before ---
{'message': 'Book a cab for Klement', 'done': False, 'reply': ''}
Received: Book a cab for Klement
--- After ---
{'message': 'Book a cab for Klement', 'done': True, 'reply': 'Got it: Book a cab for Klement'}
```

The function received the state, updated two fields, returned it. Before:
`done=False, reply=""`. After: `done=True, reply="Got it: ..."`.

**In every LangGraph project, every node has this exact shape:**

```python
def my_node(state: AgentState) -> AgentState:
    # read from state
    # update state
    # return state
```

TypedDict is the state that flows through the entire graph.

---

## Pattern 4 — Pydantic BaseModel

TypedDict is just labels — no enforcement. If you pass wrong data, nothing
happens. Pydantic actually **checks** the data and **crashes immediately** if
something is wrong.

**Used for:** Tool inputs in LangChain — the LLM fills these in. The LLM might
pass the wrong type. Pydantic catches it, returns the error, the LLM corrects
itself and tries again.

**Access:** dot `.` — not `["key"]` like a dict.

---

### Basic BaseModel

```python
from pydantic import BaseModel

class CabBooking(BaseModel):
    pickup: str
    destination: str
    seats: int

booking = CabBooking(pickup="Nacharam", destination="Airport", seats=2)
print(booking.pickup)       # Nacharam
print(booking.destination)  # Airport
print(booking.seats)        # 2
```

Pydantic checks every field when you create the object. Pass the wrong type:

```python
booking = CabBooking(pickup="Nacharam", destination="Airport", seats="two")
# ValidationError: seats — Input should be a valid integer
```

`seats="two"` → Pydantic crashed immediately. Clear error, exact field.

---

### Optional field

```python
from pydantic import BaseModel
from typing import Optional

class Contact(BaseModel):
    name: str
    phone: str
    note: Optional[str] = None

c1 = Contact(name="Klement", phone="+91-9999")              # note = None
c2 = Contact(name="Mom", phone="+91-8888", note="Telugu")   # note filled

print(c1.name, c1.phone, c1.note)   # Klement +91-9999 None
print(c2.name, c2.phone, c2.note)   # Mom +91-8888 Telugu
```

- `name` and `phone` are required — must always be passed
- `note` is optional — can skip it, defaults to `None`

This is Pattern 2 (Optional) + Pattern 4 (Pydantic) working together. The type
hints you write → Pydantic reads and enforces them.

---

### Literal inside BaseModel

```python
from pydantic import BaseModel
from typing import Literal

class ReminderInput(BaseModel):
    person: str
    message: str
    type: Literal["call", "cab", "food"]

r = ReminderInput(person="Mom", message="Doctor at 3pm", type="call")
print(r.person, r.message, r.type)  # Mom Doctor at 3pm call

# r2 = ReminderInput(person="Mom", message="Doctor at 3pm", type="email")
# → ValidationError: type — Input should be 'call', 'cab' or 'food'
```

`Literal` in Pattern 2 was just a label. Inside Pydantic, it becomes
enforced. Pass `"email"` → crash. The agent sees the error, corrects itself.

---

### Nested models — one model inside another

```python
from pydantic import BaseModel

class Address(BaseModel):
    city: str
    country: str

class Person(BaseModel):
    name: str
    age: int
    address: Address

klement = Person(
    name="Klement",
    age=25,
    address=Address(city="New York", country="USA")
)

print(klement.name)            # Klement
print(klement.address.city)    # New York
print(klement.address.country) # USA
```

One model holds another. Access goes one level deeper with a dot.
Pydantic validates both levels.

---

### model_validator — custom rule across fields

Sometimes you need a rule that Python cannot express with just a type.
You write it yourself:

```python
from pydantic import BaseModel, model_validator

class BookingInput(BaseModel):
    seats: int
    max_seats: int

    @model_validator(mode="after")
    def check_seats(self):
        if self.seats > self.max_seats:
            raise ValueError(f"seats {self.seats} cannot exceed max_seats {self.max_seats}")
        return self

b1 = BookingInput(seats=2, max_seats=4)
print(f"Booked {b1.seats} of {b1.max_seats} seats — OK")

# b2 = BookingInput(seats=6, max_seats=4)
# → ValueError: seats 6 cannot exceed max_seats 4
```

- `@model_validator(mode="after")` — runs after all fields are loaded
- `self.seats`, `self.max_seats` — access the fields using `self`
- `raise ValueError(...)` — your custom error message
- `return self` — if everything is fine, return the object

`Literal` catches wrong values. `model_validator` catches wrong logic between
two fields.

---

### Field — number boundaries

```python
from pydantic import BaseModel, Field

class OrderInput(BaseModel):
    item: str
    quantity: int = Field(ge=1, le=10)
    discount: float = Field(ge=0.0, le=100.0)

o1 = OrderInput(item="Rice", quantity=3, discount=10.0)
print(f"{o1.item} x{o1.quantity} — {o1.discount}% off")  # Rice x3 — 10.0% off

# OrderInput(item="Rice", quantity=0, discount=10.0)
# → ValidationError: quantity — Input should be greater than or equal to 1
```

| | Meaning |
|---|---|
| `ge=1` | greater than or equal to 1 |
| `le=10` | less than or equal to 10 |

Pass `quantity=0` → below minimum → instant error.
Pass `quantity=11` → above maximum → instant error.

---

### All combined — real Aria tool input

This is what a real LangChain tool input looks like. Every pattern together:

```python
from pydantic import BaseModel, Field
from typing import Optional, Literal

class BookCabInput(BaseModel):
    pickup: str
    destination: str
    seats: int = Field(ge=1, le=6)
    language: Literal["english", "telugu"] = "english"
    note: Optional[str] = None
```

| Field | Pattern |
|-------|---------|
| `pickup: str` | Pattern 2 — type hint |
| `seats: int = Field(ge=1, le=6)` | Pattern 4 — Field constraints |
| `language: Literal["english", "telugu"]` | Pattern 2 — Literal |
| `note: Optional[str] = None` | Pattern 2 — Optional |

- Minimal call: `BookCabInput(pickup="Nacharam", destination="Airport", seats=2)`
  → language defaults to "english", note defaults to None
- Full call: all fields filled explicitly

---

## TypedDict vs Pydantic — Quick Reference

| | TypedDict | Pydantic BaseModel |
|---|---|---|
| Syntax | `class X(TypedDict)` | `class X(BaseModel)` |
| Enforces types? | No | Yes — crashes if wrong |
| Access | `state["key"]` | `obj.key` |
| Used for | LangGraph state (you write it) | LangChain tool inputs (LLM fills it) |

**The rule:**
- LLM writes it → use Pydantic (it needs validation)
- You write it → use TypedDict (you control it, no enforcement needed)

In every LangGraph project you will see both in the same file:

```python
class AgentState(TypedDict):    # state between nodes — you control it
    messages: list[str]
    next_step: str
    done: bool

class BookCabInput(BaseModel):  # tool input — LLM fills this — needs checking
    pickup: str
    destination: str
    seats: int = Field(ge=1, le=6)
```

Both needed. Different jobs.

---

*Patterns 5–12 coming soon.*

---

## Pattern 5 — Decorators + `@tool`

A decorator is a line that starts with `@` placed directly above a function.
It wraps that function and adds behavior — without changing the function itself.

### What a decorator does

```python
def shout(func):
    def wrapper(*args, **kwargs):
        print("--- calling function ---")
        result = func(*args, **kwargs)
        print("--- done ---")
        return result
    return wrapper

@shout
def greet(name: str) -> str:
    return f"Hello, {name}!"

message = greet("Klement")
print(message)
```

**Output:**

```text
--- calling function ---
--- done ---
Hello, Klement!
```

`@shout` wraps `greet`. Every time `greet` runs, `shout` adds the before/after
lines automatically. You did not change `greet` at all.

**The pattern:**
```
@decorator
def my_function():
    ...
```
= "before running my_function, pass it through decorator first."

---

### `@tool` — the LangChain decorator

`@tool` wraps your function and registers it as a LangChain tool. LangChain
reads three things automatically:

1. **Function name** → tool name
2. **Type hints** → schema (what the agent must fill in)
3. **Docstring** → description (what the agent reads to decide when to use it)

```python
from langchain.tools import tool

@tool
def book_cab(pickup: str, destination: str, seats: int = 1) -> str:
    """Book a cab from pickup to destination.

    Args:
        pickup: The pickup location
        destination: The drop-off location
        seats: Number of seats needed
    """
    return f"Cab booked: {pickup} → {destination} for {seats} seat(s)"

# Call it
result = book_cab.invoke({"pickup": "Nacharam", "destination": "Airport", "seats": 2})
print(result)

# See what LangChain built automatically
print(book_cab.name)
print(book_cab.description)
print(book_cab.args)
```

**Output:**

```text
Cab booked: Nacharam → Airport for 2 seat(s)

name:        book_cab
description: Book a cab from pickup to destination.
args:        pickup → string
             destination → string
             seats → integer (default 1)
```

**Line by line:**

- `@tool` — LangChain reads the function name, type hints, and docstring
- The **docstring** becomes the tool description — what the agent reads to decide
  when to call this tool
- The **type hints** become the schema — what fields the agent must fill in
- LangChain built the entire schema automatically from your code

---

### How to call a tool

```python
# Always use .invoke() with a dict — not book_cab(...)
book_cab.invoke({"pickup": "Nacharam", "destination": "Airport", "seats": 2})
```

---

### The full flow — type hints + docstring + `@tool`

```
You write:
  @tool
  def book_cab(pickup: str, destination: str, seats: int) -> str:
      """Book a cab..."""

LangChain reads:
  name        = "book_cab"
  description = "Book a cab..."
  schema      = pickup (string), destination (string), seats (integer)

Agent reads the schema and fills in:
  pickup      = "Nacharam"
  destination = "Airport"
  seats       = 2

Pydantic validates the values.
Your function runs with correct inputs.
```

This is why Pattern 2 (type hints) and Pattern 4 (Pydantic) came first.
`@tool` is the step that connects them to the agent.


---

### `@tool` Example 2 — with Pydantic input schema (production style)

Instead of just type hints on the function, you define a full Pydantic model
as the input schema. LangChain uses the model for full validation.

```python
from langchain.tools import tool
from pydantic import BaseModel, Field
from typing import Literal, Optional

class SendReminderInput(BaseModel):
    person: str
    message: str
    type: Literal["call", "cab", "food"]
    urgent: bool = False
    note: Optional[str] = None

@tool("send_reminder", args_schema=SendReminderInput)
def send_reminder(person: str, message: str, type: str, urgent: bool, note: Optional[str]) -> str:
    """Send a reminder to a person via call, cab booking, or food order."""
    prefix = "[URGENT] " if urgent else ""
    result = f"{prefix}Reminder sent to {person}: {message} via {type}"
    if note:
        result += f" ({note})"
    return result

result = send_reminder.invoke({
    "person": "Mom",
    "message": "Doctor appointment at 3pm",
    "type": "call",
    "urgent": True,
    "note": "Speak Telugu"
})
print(result)
```

**Output:**

```text
[URGENT] Reminder sent to Mom: Doctor appointment at 3pm via call (Speak Telugu)
```

Two new things in `@tool("send_reminder", args_schema=SendReminderInput)`:
- `"send_reminder"` — custom name for the tool
- `args_schema=SendReminderInput` — Pydantic model plugged in as schema

**What the schema produces:**
- `type` → `enum: ['call', 'cab', 'food']` — Literal became a strict enum
- `urgent` → `default: False` — bool with default
- `note` → `anyOf: [string, null]` — Optional[str]

The agent sees only those three choices for `type`. It cannot pass `"email"`.

**Example 1 vs Example 2:**

| | Example 1 | Example 2 |
|---|---|---|
| Schema from | type hints on function | Pydantic model |
| Validation | basic | full (Field, Literal, Optional) |
| Custom tool name | no | yes |
| Used when | simple tools | production tools |

---

### `@tool` Example 3 — multiple tools as a list

```python
from langchain.tools import tool

@tool
def get_weather(city: str) -> str:
    """Get the current weather for a city."""
    return f"Weather in {city}: Sunny, 28°C"

@tool
def book_cab(pickup: str, destination: str) -> str:
    """Book a cab from pickup to destination."""
    return f"Cab booked: {pickup} → {destination}"

@tool
def send_message(to: str, body: str) -> str:
    """Send a text message to a contact."""
    return f"Message sent to {to}: {body}"

tools = [get_weather, book_cab, send_message]

for t in tools:
    print(f"{t.name}: {t.description.strip()}")

print(get_weather.invoke({"city": "Hyderabad"}))
print(book_cab.invoke({"pickup": "Nacharam", "destination": "Airport"}))
print(send_message.invoke({"to": "Mom", "body": "Doctor at 3pm"}))
```

**Output:**

```text
get_weather: Get the current weather for a city.
book_cab: Book a cab from pickup to destination.
send_message: Send a text message to a contact.

Weather in Hyderabad: Sunny, 28°C
Cab booked: Nacharam → Airport
Message sent to Mom: Doctor at 3pm
```

`tools = [get_weather, book_cab, send_message]` — this exact list is passed to
the agent:

```python
agent = create_agent(model, tools=tools)
```

The agent reads all three descriptions and decides which tool to call:
- "What is the weather in Hyderabad" → agent picks `get_weather`
- "Book me a cab" → agent picks `book_cab`
- "Text mom" → agent picks `send_message`

**The docstring is everything.** Clear docstring = agent picks the right tool.
Vague docstring = agent guesses wrong.

---

### `@tool` Example 4 — tool using real data inside

```python
from langchain.tools import tool

contacts = {
    "Mom": "+91-8888",
    "Klement": "+91-9999",
    "Aria": "+91-7777"
}

@tool
def lookup_contact(name: str) -> str:
    """Look up a contact's phone number by name."""
    phone = contacts.get(name)
    if phone:
        return f"{name}: {phone}"
    return f"Contact '{name}' not found."

print(lookup_contact.invoke({"name": "Mom"}))       # Mom: +91-8888
print(lookup_contact.invoke({"name": "Aria"}))      # Aria: +91-7777
print(lookup_contact.invoke({"name": "Unknown"}))   # Contact 'Unknown' not found.
```

**Output:**

```text
Mom: +91-8888
Aria: +91-7777
Contact 'Unknown' not found.
```

The tool looks up from a real dict inside the function. Found → returns the
number. Not found → returns a clear message. This is exactly how Aria's contact
lookup tool works.

---

### Pattern 5 — All examples summary

| Example | What it shows |
|---------|--------------|
| 1 | Basic `@tool` — type hints become schema automatically |
| 2 | `args_schema=PydanticModel` — full Pydantic validation |
| 3 | Multiple tools as a list — how you register with an agent |
| 4 | Tool using real data inside the function |


---

### `@tool` Example 5 — error handling inside a tool

Tools should never crash the agent. Always return a string — even for errors.
The agent reads the error string and knows to try different inputs.

```python
from langchain.tools import tool
from pydantic import BaseModel

class DivideInput(BaseModel):
    numerator: float
    denominator: float

@tool("divide", args_schema=DivideInput)
def divide(numerator: float, denominator: float) -> str:
    """Divide two numbers. Returns an error if denominator is zero."""
    if denominator == 0:
        return "Error: cannot divide by zero."
    result = numerator / denominator
    return f"{numerator} ÷ {denominator} = {result}"

print(divide.invoke({"numerator": 10, "denominator": 2}))   # 10.0 ÷ 2.0 = 5.0
print(divide.invoke({"numerator": 5, "denominator": 0}))    # Error: cannot divide by zero.
```

---

### `@tool` Example 6 — real Aria tool: remind Mom

```python
from langchain.tools import tool
from pydantic import BaseModel
from typing import Literal

class RemindMomInput(BaseModel):
    message: str
    language: Literal["english", "telugu"]
    urgent: bool = False

@tool("remind_mom", args_schema=RemindMomInput)
def remind_mom(message: str, language: str, urgent: bool) -> str:
    """Send a reminder to Mom in English or Telugu. Use urgent=True for time-sensitive messages."""
    prefix = "[URGENT] " if urgent else ""
    lang_note = "Telugu call" if language == "telugu" else "English message"
    return f"{prefix}{lang_note} to Mom: {message}"

print(remind_mom.invoke({"message": "Doctor at 3pm", "language": "telugu", "urgent": True}))
print(remind_mom.invoke({"message": "Lunch is ready", "language": "english"}))
print(remind_mom.invoke({"message": "Take medicine", "language": "telugu"}))
```

**Output:**

```text
[URGENT] Telugu call to Mom: Doctor at 3pm
English message to Mom: Lunch is ready
Telugu call to Mom: Take medicine
```

Third call — no `urgent` passed → defaults to `False` → no `[URGENT]` prefix.

---

### `@tool` Example 7 — tool that builds a formatted summary

```python
from langchain.tools import tool
from pydantic import BaseModel, Field
from typing import Literal

class DailySummaryInput(BaseModel):
    person: Literal["Klement", "Mom", "Aria"]
    tasks_done: int = Field(ge=0)
    pending: int = Field(ge=0)

@tool("daily_summary", args_schema=DailySummaryInput)
def daily_summary(person: str, tasks_done: int, pending: int) -> str:
    """Generate a daily summary report for a person."""
    total = tasks_done + pending
    pct = round((tasks_done / total) * 100) if total > 0 else 0
    return (
        f"Daily summary for {person}:\n"
        f"  Done:    {tasks_done}/{total} tasks ({pct}%)\n"
        f"  Pending: {pending} tasks"
    )

print(daily_summary.invoke({"person": "Klement", "tasks_done": 7, "pending": 3}))
print(daily_summary.invoke({"person": "Aria", "tasks_done": 12, "pending": 1}))
```

**Output:**

```text
Daily summary for Klement:
  Done:    7/10 tasks (70%)
  Pending: 3 tasks

Daily summary for Aria:
  Done:    12/13 tasks (92%)
  Pending: 1 tasks
```

- `Literal["Klement", "Mom", "Aria"]` — only these three names allowed
- `Field(ge=0)` — task count cannot be negative
- Multi-line return — tools can return formatted text, not just one line


---

## Pattern 6 — async/await

### The problem

Normal functions run one at a time. If a function waits (API call, database,
file read), Python sits idle doing nothing.

```
call API → wait 2s → get result → call API again → wait 2s → get result
Total: 4 seconds wasted
```

`async/await` lets Python do other work while waiting:

```
call API → while waiting, call another API → both results arrive together
Total: 2 seconds
```

### Two keywords

| Keyword | Meaning |
|---------|---------|
| `async def` | "this function can pause and wait" |
| `await` | "pause here until this is ready, let others run" |

### Example 1 — basic async function

```python
import asyncio

async def greet(name: str) -> str:
    await asyncio.sleep(1)   # simulates waiting (like an API call)
    return f"Hello, {name}!"

async def main():
    result = await greet("Klement")
    print(result)

asyncio.run(main())
```

**Output:**

```text
Hello, Klement!
```

**Line by line:**

- `async def greet(...)` — same as a normal function, just `async` in front.
  Means "this function can pause."
- `await asyncio.sleep(1)` — pause here for 1 second (simulating an API call).
  Let other things run while waiting.
- `async def main():` — to use `await`, you must be inside another `async def`
- `result = await greet("Klement")` — to call an async function, you must `await` it
- `asyncio.run(main())` — the entry point. Starts the async engine and runs `main()`

---

### Example 2 — sequential vs parallel

**Sequential (slow):**

```python
import asyncio

async def fetch_weather(city: str) -> str:
    await asyncio.sleep(2)
    return f"Weather in {city}: Sunny"

async def fetch_news() -> str:
    await asyncio.sleep(2)
    return "Top news: Space house launched"

async def main():
    weather = await fetch_weather("Hyderabad")   # wait 2s
    news = await fetch_news()                     # wait 2s more
    print(weather)
    print(news)

asyncio.run(main())
# Total: 4 seconds
```

**Parallel (fast) — `asyncio.gather`:**

```python
async def main():
    weather, news = await asyncio.gather(
        fetch_weather("Hyderabad"),
        fetch_news()
    )
    print(weather)
    print(news)

asyncio.run(main())
# Total: 2 seconds
```

**Output (both):**

```text
Weather in Hyderabad: Sunny
Top news: Space house launched
```

`asyncio.gather(task1, task2)` = start both at once, wait for both, return both
results together. The left side unpacks results in the same order as the tasks.

---

### Example 3 — gather with 3 tasks (Aria morning briefing)

```python
import asyncio

async def get_weather(city: str) -> str:
    await asyncio.sleep(1)
    return f"Weather in {city}: Sunny, 28°C"

async def get_news() -> str:
    await asyncio.sleep(1)
    return "Top news: Space house launched"

async def get_reminders() -> str:
    await asyncio.sleep(1)
    return "Reminder: Mom doctor at 3pm"

async def main():
    weather, news, reminders = await asyncio.gather(
        get_weather("Hyderabad"),
        get_news(),
        get_reminders()
    )
    print("--- Morning Briefing ---")
    print(weather)
    print(news)
    print(reminders)

asyncio.run(main())
```

**Output:**

```text
--- Morning Briefing ---
Weather in Hyderabad: Sunny, 28°C
Top news: Space house launched
Reminder: Mom doctor at 3pm
```

Time: 1 second — not 3. Add more tasks → still takes the time of the slowest
one, not the sum of all.

---

### Example 4 — what if one task fails?

By default, if one task crashes, everything stops. `return_exceptions=True`
collects the error as a result instead — the other tasks still finish.

```python
import asyncio

async def get_weather(city: str) -> str:
    await asyncio.sleep(1)
    return f"Weather in {city}: Sunny"

async def get_news() -> str:
    await asyncio.sleep(1)
    raise ValueError("News API is down!")

async def get_reminders() -> str:
    await asyncio.sleep(1)
    return "Reminder: Mom doctor at 3pm"

async def main():
    results = await asyncio.gather(
        get_weather("Hyderabad"),
        get_news(),
        get_reminders(),
        return_exceptions=True
    )
    for result in results:
        if isinstance(result, Exception):
            print(f"ERROR: {result}")
        else:
            print(result)

asyncio.run(main())
```

**Output:**

```text
Weather in Hyderabad: Sunny
ERROR: News API is down!
Reminder: Mom doctor at 3pm
```

`isinstance(result, Exception)` — checks if this result is an error or a real
value. Without `return_exceptions=True`, one failure kills everything.

---

### Example 5 — timeout: cancel if too slow

```python
import asyncio

async def fetch_data(source: str, delay: float) -> str:
    await asyncio.sleep(delay)
    return f"Data from {source}"

async def main():
    try:
        result = await asyncio.wait_for(
            fetch_data("slow API", delay=5),
            timeout=3
        )
        print(result)
    except asyncio.TimeoutError:
        print("ERROR: took too long — cancelled")

asyncio.run(main())
```

**Output:**

```text
ERROR: took too long — cancelled
```

`asyncio.wait_for(task, timeout=N)` = run this task, but cancel it if it takes
longer than N seconds. Change `delay=5` to `delay=1` and it finishes in time.

Aria pattern:
```python
try:
    weather = await asyncio.wait_for(get_weather(), timeout=3)
except asyncio.TimeoutError:
    weather = "Weather unavailable right now"
```

---

### Example 6 — async inside a class

```python
import asyncio

class Aria:
    def __init__(self, name: str):
        self.name = name

    async def get_weather(self, city: str) -> str:
        await asyncio.sleep(1)
        return f"Weather in {city}: Sunny, 28°C"

    async def get_reminder(self) -> str:
        await asyncio.sleep(1)
        return "Reminder: Mom doctor at 3pm"

    async def morning_briefing(self, city: str) -> None:
        weather, reminder = await asyncio.gather(
            self.get_weather(city),
            self.get_reminder()
        )
        print(f"Good morning! I am {self.name}.")
        print(weather)
        print(reminder)


async def main():
    aria = Aria("Aria")
    await aria.morning_briefing("Hyderabad")

asyncio.run(main())
```

**Output:**

```text
Good morning! I am Aria.
Weather in Hyderabad: Sunny, 28°C
Reminder: Mom doctor at 3pm
```

Same rules inside a class: `async def`, `await`, gather works with `self.method()`.
Calling an async method from outside still needs `await`.

---

### Example 7 — async in LangChain (`ainvoke`)

Every LangChain tool has two versions:

| Sync | Async |
|------|-------|
| `.invoke()` | `.ainvoke()` |
| blocks until done | can run in parallel |

```python
import asyncio
from langchain.tools import tool

@tool
def get_weather(city: str) -> str:
    """Get the weather for a city."""
    return f"Weather in {city}: Sunny, 28°C"

@tool
def get_reminder(person: str) -> str:
    """Get reminders for a person."""
    return f"Reminder for {person}: Doctor at 3pm"

async def main():
    weather, reminder = await asyncio.gather(
        get_weather.ainvoke({"city": "Hyderabad"}),
        get_reminder.ainvoke({"person": "Mom"})
    )
    print(weather)
    print(reminder)

asyncio.run(main())
```

**Output:**

```text
Weather in Hyderabad: Sunny, 28°C
Reminder for Mom: Doctor at 3pm
```

The only change from sync: `.invoke()` → `.ainvoke()`. Same tool, same input
dict. Now it runs inside `asyncio.gather`.

**Important:** calling `.ainvoke()` on a synchronous `@tool` does not make it a
true async coroutine. LangChain runs it in a thread pool via
`asyncio.run_in_executor`. The parallelism is thread-based, not coroutine-based.
`.ainvoke()` still gives you an async interface and avoids blocking the event
loop — it just uses threads under the hood for sync functions.

---

### Pattern 6 — All examples summary

| Example | What it shows |
|---------|--------------|
| 1 | Basic async function + await |
| 2 | Sequential vs parallel — gather saves time |
| 3 | Gather with 3 tasks — morning briefing |
| 4 | Error handling — return_exceptions=True |
| 5 | Timeout — wait_for |
| 6 | Async inside a class |
| 7 | LangChain ainvoke — the real use case |

---

## Pattern 7 — try/except

### The problem

Without error handling, one bad thing crashes the entire program:

```python
result = 10 / 0
print("done")   # never runs
# ZeroDivisionError: division by zero — program stopped
```

`try/except` catches the crash and lets you decide what to do instead.

### The shape

```
try:
    [code that might fail]
except SomeError:
    [what to do if it fails]
else:
    [runs only if try succeeded — no error]
finally:
    [always runs — error or not]
```

---

### Example 1 — basic try/except

```python
def divide(a: float, b: float) -> str:
    try:
        result = a / b
        return f"{a} ÷ {b} = {result}"
    except ZeroDivisionError:
        return "Error: cannot divide by zero"

print(divide(10, 2))   # 10.0 ÷ 2.0 = 5.0
print(divide(5, 0))    # Error: cannot divide by zero
```

**Output:**

```text
10.0 ÷ 2.0 = 5.0
Error: cannot divide by zero
```

`try` runs the code. If `ZeroDivisionError` happens, Python jumps straight to
`except`. The program does not crash.

---

### Example 2 — multiple except blocks

```python
def parse_and_divide(a: str, b: str) -> str:
    try:
        num_a = int(a)
        num_b = int(b)
        result = num_a / num_b
        return f"{num_a} ÷ {num_b} = {result}"
    except ValueError:
        return "Error: both inputs must be numbers"
    except ZeroDivisionError:
        return "Error: cannot divide by zero"

print(parse_and_divide("10", "2"))     # 10 ÷ 2 = 5.0
print(parse_and_divide("10", "0"))     # Error: cannot divide by zero
print(parse_and_divide("ten", "2"))    # Error: both inputs must be numbers
```

**Output:**

```text
10 ÷ 2 = 5.0
Error: cannot divide by zero
Error: both inputs must be numbers
```

Python checks each `except` in order — top to bottom. First match wins.

---

### Example 3 — else and finally

```python
def read_file(filename: str) -> None:
    try:
        f = open(filename, "r")
        content = f.read()
    except FileNotFoundError:
        print(f"Error: '{filename}' not found")
    else:
        print(f"Success! Contents: {content}")
    finally:
        print("Done — always runs")
```

| Block | When it runs |
|-------|-------------|
| `try` | always — the code to attempt |
| `except` | only if try raised that error |
| `else` | only if try succeeded — no error |
| `finally` | always — error or not |

`finally` is for cleanup: close a file, close a database connection, log that
the attempt happened — regardless of what went wrong.

---

### Example 4 — catching the error message with `as e`

```python
def fetch_data(url: str) -> str:
    try:
        if not url.startswith("https"):
            raise ValueError(f"Invalid URL: {url}")
        return f"Data from {url}"
    except ValueError as e:
        return f"Error caught: {e}"

print(fetch_data("https://api.example.com"))   # Data from https://api.example.com
print(fetch_data("http://api.example.com"))    # Error caught: Invalid URL: http://...
print(fetch_data("not-a-url"))                 # Error caught: Invalid URL: not-a-url
```

`as e` gives the error a name so you can read its message.
`raise ValueError("message")` creates and throws your own error immediately.

In LangChain tools: `except Exception as e: return f"Tool failed: {e}"`

---

### Example 5 — custom exceptions

```python
class InvalidCityError(Exception):
    pass

class APITimeoutError(Exception):
    pass

def get_weather(city: str) -> str:
    if not city:
        raise InvalidCityError("City name cannot be empty")
    if city == "unknown":
        raise APITimeoutError("Weather API did not respond")
    return f"Weather in {city}: Sunny, 28°C"

for city in ["Hyderabad", "", "unknown"]:
    try:
        print(get_weather(city))
    except InvalidCityError as e:
        print(f"Bad input: {e}")
    except APITimeoutError as e:
        print(f"API issue: {e}")
```

**Output:**

```text
Weather in Hyderabad: Sunny, 28°C
Bad input: City name cannot be empty
API issue: Weather API did not respond
```

`class InvalidCityError(Exception): pass` — inherits from Exception, that is
all it needs. Now you have a brand new error type with a meaningful name.

---

### Example 6 — try/except inside a LangChain tool (production pattern)

```python
from langchain.tools import tool
from pydantic import BaseModel
from typing import Literal

contacts = {"Mom": "+91-8888", "Klement": "+91-9999"}

class CallContactInput(BaseModel):
    name: str
    language: Literal["english", "telugu"]

@tool("call_contact", args_schema=CallContactInput)
def call_contact(name: str, language: str) -> str:
    """Call a contact by name in the given language."""
    try:
        if name not in contacts:
            raise ValueError(f"Contact '{name}' not found")
        phone = contacts[name]
        return f"Calling {name} ({phone}) in {language}"
    except ValueError as e:
        return f"Error: {e}"
    except Exception as e:
        return f"Unexpected error: {e}"

print(call_contact.invoke({"name": "Mom", "language": "telugu"}))
print(call_contact.invoke({"name": "Dad", "language": "english"}))
```

**Output:**

```text
Calling Mom (+91-8888) in telugu
Error: Contact 'Dad' not found
```

Always two except blocks in tools: one for expected errors you raised yourself,
one for anything unexpected. Tools must never crash the agent — always return a
string.

---

### Pattern 7 — All examples summary

| Example | What it shows |
|---------|--------------|
| 1 | Basic try/except — catch one error |
| 2 | Multiple except — different errors, different responses |
| 3 | else + finally — success path + cleanup |
| 4 | `as e` + `raise` — read the error, create your own |
| 5 | Custom exceptions — your own error types |
| 6 | try/except inside a LangChain tool — production pattern |

---

## Pattern 8 — Dataclasses

### The problem with regular classes

```python
class Point:
    def __init__(self, x: int, y: int):
        self.x = x
        self.y = y
```

You write `__init__` manually every time just to store data. Repetitive.
`@dataclass` generates all of that automatically.

---

### Example 1 — basic dataclass

```python
from dataclasses import dataclass

@dataclass
class Point:
    x: int
    y: int

p = Point(x=3, y=7)
print(p.x)    # 3
print(p.y)    # 7
print(p)      # Point(x=3, y=7)
```

**Output:**

```text
3
7
Point(x=3, y=7)
```

`@dataclass` gives you for free: `__init__` (no need to write it) and
`__repr__` (`print(p)` shows `Point(x=3, y=7)` not a memory address).

**Dataclass vs Pydantic vs TypedDict:**

| | TypedDict | Dataclass | Pydantic |
|---|---|---|---|
| Validates types? | No | No | Yes |
| Auto `__init__`? | No | Yes | Yes |
| Access | `["key"]` | `.field` | `.field` |
| Used for | LangGraph state | internal data structures | tool inputs |

---

### Example 2 — default values

```python
from dataclasses import dataclass
from typing import Optional

@dataclass
class Contact:
    name: str
    phone: str
    language: str = "english"
    note: Optional[str] = None

c1 = Contact(name="Mom", phone="+91-8888")
c2 = Contact(name="Klement", phone="+91-9999", language="telugu", note="Call after 6pm")

print(c1)
print(c2)
```

**Output:**

```text
Contact(name='Mom', phone='+91-8888', language='english', note=None)
Contact(name='Klement', phone='+91-9999', language='telugu', note='Call after 6pm')
```

Fields with defaults must come after fields without defaults.

---

### Example 3 — field() for complex defaults

You cannot use a list or dict as a default directly — Python shares it across
all instances. `field(default_factory=...)` gives each object its own fresh copy.

```python
from dataclasses import dataclass, field
from typing import List

@dataclass
class TaskList:
    owner: str
    tasks: List[str] = field(default_factory=list)

t1 = TaskList(owner="Klement")
t2 = TaskList(owner="Mom")

t1.tasks.append("Buy groceries")
t1.tasks.append("Call doctor")
t2.tasks.append("Watch movie")

print(t1)
print(t2)
```

**Output:**

```text
TaskList(owner='Klement', tasks=['Buy groceries', 'Call doctor'])
TaskList(owner='Mom', tasks=['Watch movie'])
```

| Default type | How to write it |
|---|---|
| `"english"`, `0`, `True` | write directly |
| `[]`, `{}` | use `field(default_factory=list)` or `field(default_factory=dict)` |

---

### Example 4 — methods inside a dataclass

```python
from dataclasses import dataclass, field
from typing import List

@dataclass
class ShoppingList:
    owner: str
    items: List[str] = field(default_factory=list)

    def add(self, item: str) -> None:
        self.items.append(item)

    def remove(self, item: str) -> None:
        if item in self.items:
            self.items.remove(item)

    def total(self) -> int:
        return len(self.items)

    def summary(self) -> str:
        return f"{self.owner} has {self.total()} items: {self.items}"

cart = ShoppingList(owner="Klement")
cart.add("Rice")
cart.add("Milk")
cart.add("Eggs")
cart.remove("Milk")

print(cart.summary())
print(cart)
```

**Output:**

```text
Klement has 2 items: ['Rice', 'Eggs']
ShoppingList(owner='Klement', items=['Rice', 'Eggs'])
```

`@dataclass` handles the boring part (`__init__`, `__repr__`). You write the
interesting part — the methods. Everything else is identical to a regular class.

---

### Example 5 — frozen dataclass (immutable)

`frozen=True` makes the object read-only after creation.

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Config:
    model: str
    temperature: float
    max_tokens: int

config = Config(model="gpt-4o-mini", temperature=0.0, max_tokens=1000)
print(config)

config.model = "gpt-4o"   # FrozenInstanceError: cannot assign to field 'model'
```

| | Normal `@dataclass` | `@dataclass(frozen=True)` |
|---|---|---|
| Data changes over time | Yes (shopping list, task list) | No |
| Config / settings | No | Yes |

---

### Example 6 — `__post_init__` (run code after creation)

`__post_init__` runs automatically after `__init__`. Use it to compute fields
or validate data right after the object is created.

```python
from dataclasses import dataclass

@dataclass
class CabBooking:
    pickup: str
    destination: str
    seats: int
    price_per_seat: float

    total_price: float = 0.0

    def __post_init__(self):
        if self.seats < 1:
            raise ValueError("seats must be at least 1")
        self.total_price = self.seats * self.price_per_seat

b1 = CabBooking(pickup="Nacharam", destination="Airport", seats=2, price_per_seat=150.0)
print(b1.total_price)   # 300.0
print(b1)

# b2 = CabBooking(..., seats=0, ...) → ValueError: seats must be at least 1
```

**Output:**

```text
300.0
CabBooking(pickup='Nacharam', destination='Airport', seats=2, price_per_seat=150.0, total_price=300.0)
```

| Use | Example |
|-----|---------|
| Compute a field | `self.total_price = seats × price_per_seat` |
| Validate | `if seats < 1: raise ValueError(...)` |

This is the dataclass equivalent of Pydantic's `@model_validator`.

---

### Example 7 — real Aria use case (all combined)

```python
from dataclasses import dataclass, field
from typing import List

@dataclass
class Reminder:
    person: str
    message: str
    urgent: bool = False

    def label(self) -> str:
        prefix = "[URGENT] " if self.urgent else ""
        return f"{prefix}{self.person}: {self.message}"


@dataclass
class DailyBriefing:
    weather: str
    reminders: List[Reminder] = field(default_factory=list)
    news: List[str] = field(default_factory=list)

    def add_reminder(self, person: str, message: str, urgent: bool = False) -> None:
        self.reminders.append(Reminder(person, message, urgent))

    def add_news(self, headline: str) -> None:
        self.news.append(headline)

    def summary(self) -> str:
        lines = [f"Weather: {self.weather}", ""]
        lines.append("Reminders:")
        for r in self.reminders:
            lines.append(f"  - {r.label()}")
        lines.append("")
        lines.append("News:")
        for n in self.news:
            lines.append(f"  - {n}")
        return "\n".join(lines)


briefing = DailyBriefing(weather="Hyderabad: Sunny, 28°C")
briefing.add_reminder("Mom", "Doctor at 3pm", urgent=True)
briefing.add_reminder("Klement", "Team call at 5pm")
briefing.add_news("Space house launched")
briefing.add_news("Markets up 2%")

print(briefing.summary())
```

**Output:**

```text
Weather: Hyderabad: Sunny, 28°C

Reminders:
  - [URGENT] Mom: Doctor at 3pm
  - Klement: Team call at 5pm

News:
  - Space house launched
  - Markets up 2%
```

---

### Pattern 8 — All examples summary

| Example | What it shows |
|---------|--------------|
| 1 | Basic `@dataclass` — auto `__init__` + `__repr__` |
| 2 | Default values — required first, optional after |
| 3 | `field(default_factory=list)` — each object gets its own list |
| 4 | Methods inside a dataclass |
| 5 | `frozen=True` — immutable, for config |
| 6 | `__post_init__` — compute fields + validate after creation |
| 7 | Real Aria use case — nested dataclasses, all combined |

---

## Pattern 9 — `**kwargs`

### Two building blocks

```python
# *args — collects extra positional arguments into a list
def total(*args):
    return sum(args)

print(total(1, 2, 3))   # 6

# **kwargs — collects extra keyword arguments into a dict
def show(**kwargs):
    for key, value in kwargs.items():
        print(f"{key}: {value}")

show(name="Klement", city="Hyderabad", age=25)
# name: Klement
# city: Hyderabad
# age: 25
```

`**kwargs` — the `**` unpacks key=value pairs into a dictionary named `kwargs`.

### The order rule

```python
def func(required, default="value", *args, **kwargs):
    pass
# required → optional → *args → **kwargs (always last)
```

---

### Example 1 — basic `**kwargs`

```python
def create_profile(**kwargs) -> dict:
    return kwargs

p = create_profile(name="Klement", role="founder", city="Hyderabad")
print(p)          # {'name': 'Klement', 'role': 'founder', 'city': 'Hyderabad'}
print(p["name"])  # Klement
```

Inside the function, `kwargs` is just a regular dictionary.

---

### Example 2 — mixing required + `**kwargs`

```python
def send_message(to: str, body: str, **kwargs) -> str:
    result = f"To: {to}\nBody: {body}"
    if kwargs:
        result += f"\nExtras: {kwargs}"
    return result

print(send_message("Mom", "Doctor at 3pm"))
print(send_message("Mom", "Doctor at 3pm", urgent=True, language="telugu"))
```

**Output:**

```text
To: Mom
Body: Doctor at 3pm

To: Mom
Body: Doctor at 3pm
Extras: {'urgent': True, 'language': 'telugu'}
```

---

### Example 3 — passing `**kwargs` to another function

```python
def call_api(endpoint: str, **kwargs) -> str:
    return f"POST {endpoint} with {kwargs}"

def book_cab(pickup: str, destination: str, **kwargs) -> str:
    return call_api("/cab/book", pickup=pickup, destination=destination, **kwargs)

print(book_cab("Nacharam", "Airport"))
print(book_cab("Nacharam", "Airport", seats=2, language="telugu"))
```

**Output:**

```text
POST /cab/book with {'pickup': 'Nacharam', 'destination': 'Airport'}
POST /cab/book with {'pickup': 'Nacharam', 'destination': 'Airport', 'seats': 2, 'language': 'telugu'}
```

| Position | What it does |
|----------|-------------|
| `def func(**kwargs)` | collects incoming keyword args into a dict |
| `func(**my_dict)` | unpacks a dict into keyword args |

---

### Example 4 — `**kwargs` in LangChain tools

```python
from langchain.tools import tool

@tool
def send_reminder(person: str, message: str, **kwargs) -> str:
    """Send a reminder. Extra options: urgent, language, repeat."""
    parts = [f"Reminder to {person}: {message}"]
    if kwargs.get("urgent"):
        parts.insert(0, "[URGENT]")
    if kwargs.get("language"):
        parts.append(f"(in {kwargs['language']})")
    if kwargs.get("repeat"):
        parts.append(f"(repeat every {kwargs['repeat']})")
    return " ".join(parts)

print(send_reminder.invoke({"person": "Mom", "message": "Doctor at 3pm", "urgent": True, "language": "telugu"}))
# [URGENT] Reminder to Mom: Doctor at 3pm (in telugu)
```

`kwargs.get("key")` — safe lookup. Returns `None` if missing — no crash.

---

### Pattern 9 — All examples summary

| Example | What it shows |
|---------|--------------|
| 1 | Basic `**kwargs` — collected into a dict |
| 2 | Required + `**kwargs` — argument order rule |
| 3 | Forwarding `**kwargs` to another function |
| 4 | `**kwargs` in a LangChain tool |

---

## Pattern 10 — List Comprehensions

### The shape

```python
[expression for item in iterable]              # transform
[expression for item in iterable if condition] # transform + filter
{key: value for item in iterable}              # dict comprehension
```

Read left to right: "give me `expression`, for each `item`, only if `condition`."

---

### Example 1 — basic transformation

```python
numbers = [1, 2, 3, 4, 5]

doubled    = [n * 2 for n in numbers]
squared    = [n ** 2 for n in numbers]
as_strings = [str(n) for n in numbers]

print(doubled)     # [2, 4, 6, 8, 10]
print(squared)     # [1, 4, 9, 16, 25]
print(as_strings)  # ['1', '2', '3', '4', '5']
```

---

### Example 2 — filtering with `if`

```python
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

evens         = [n for n in numbers if n % 2 == 0]
big           = [n for n in numbers if n > 5]
doubled_evens = [n * 2 for n in numbers if n % 2 == 0]

print(evens)          # [2, 4, 6, 8, 10]
print(big)            # [6, 7, 8, 9, 10]
print(doubled_evens)  # [4, 8, 12, 16, 20]
```

---

### Example 3 — list comprehensions with strings

```python
names    = ["klement", "mom", "aria", "nathan"]
cities   = ["  Hyderabad  ", "  New York  "]
messages = ["Doctor at 3pm", "", "Call back", "", "Lunch ready"]

capitalized = [name.capitalize() for name in names]
cleaned     = [city.strip() for city in cities]
valid       = [m for m in messages if m]   # if m skips empty strings

print(capitalized)  # ['Klement', 'Mom', 'Aria', 'Nathan']
print(cleaned)      # ['Hyderabad', 'New York']
print(valid)        # ['Doctor at 3pm', 'Call back', 'Lunch ready']
```

---

### Example 4 — list comprehensions with dictionaries

```python
contacts = [
    {"name": "Mom",     "phone": "+91-8888", "active": True},
    {"name": "Klement", "phone": "+91-9999", "active": True},
    {"name": "Old",     "phone": "+91-0000", "active": False},
]

names        = [c["name"] for c in contacts]
active_names = [c["name"] for c in contacts if c["active"]]
phone_book   = {c["name"]: c["phone"] for c in contacts if c["active"]}

print(names)        # ['Mom', 'Klement', 'Old']
print(active_names) # ['Mom', 'Klement']
print(phone_book)   # {'Mom': '+91-8888', 'Klement': '+91-9999'}
```

---

### Example 5 — real Aria use case (all combined)

```python
reminders = [
    {"person": "Mom",     "message": "Doctor at 3pm", "urgent": True,  "done": False},
    {"person": "Klement", "message": "Team call 5pm", "urgent": False, "done": True},
    {"person": "Mom",     "message": "Take medicine",  "urgent": True,  "done": False},
    {"person": "Klement", "message": "Review PR",      "urgent": False, "done": False},
]

pending = [r for r in reminders if not r["done"]]
urgent  = [r for r in reminders if r["urgent"] and not r["done"]]
lines   = [
    f"[URGENT] {r['person']}: {r['message']}" if r["urgent"]
    else f"{r['person']}: {r['message']}"
    for r in pending
]
people = list({r["person"] for r in pending})

print("Pending:", len(pending))   # 3
print("Urgent:", len(urgent))     # 2
for line in lines:
    print("-", line)
print("People:", people)          # ['Mom', 'Klement']
```

---

### Pattern 10 — All examples summary

| Example | What it shows |
|---------|--------------|
| 1 | Basic transformation |
| 2 | Filtering with `if` |
| 3 | Strings — capitalize, strip, filter empty |
| 4 | Dicts — extract fields, dict comprehension |
| 5 | Real Aria use case — all combined |

---

## Pattern 11 — Annotated Types

`Annotated` attaches extra rules to a type hint. Pydantic and LangGraph read
and enforce those rules automatically.

---

### Example 1 — Annotated with Pydantic Field

```python
from typing import Annotated
from pydantic import BaseModel, Field

class BookingInput(BaseModel):
    pickup: str
    seats:    Annotated[int,   Field(ge=1, le=6)]
    discount: Annotated[float, Field(ge=0.0, le=100.0)]

b = BookingInput(pickup="Nacharam", seats=2, discount=10.0)
print(b.seats)     # 2
print(b.discount)  # 10.0
# seats=0 → ValidationError: Input should be >= 1
```

---

### Example 2 — Annotated in LangGraph state (reducer)

```python
from typing import Annotated, TypedDict
from operator import add

class AgentState(TypedDict):
    messages: Annotated[list, add]   # append, never replace
    step: int
    done: bool
```

`Annotated[list, add]` — nodes append to the list, never overwrite it.
Without this, every node replaces the conversation history.
Every real LangGraph agent has `messages: Annotated[list, add]`.

---

### Example 3 — Annotated with custom validator

```python
from typing import Annotated
from pydantic import BaseModel, AfterValidator

def valid_language(v: str) -> str:
    if v not in ("english", "telugu"):
        raise ValueError(f"must be 'english' or 'telugu'")
    return v

Language = Annotated[str, AfterValidator(valid_language)]

class ReminderInput(BaseModel):
    person: str
    message: str
    language: Language = "english"

r1 = ReminderInput(person="Mom", message="Doctor at 3pm", language="telugu")
print(r1.language)  # telugu
# language="hindi" → ValueError: must be 'english' or 'telugu'
```

Define once, reuse everywhere — `language: Language` in any model.

---

### Example 4 — real Aria use case (all combined)

```python
from typing import Annotated, TypedDict
from pydantic import BaseModel, Field, AfterValidator
from operator import add

def valid_language(v: str) -> str:
    if v not in ("english", "telugu"):
        raise ValueError("must be 'english' or 'telugu'")
    return v

Language = Annotated[str, AfterValidator(valid_language)]
Seats    = Annotated[int, Field(ge=1, le=6)]
Price    = Annotated[float, Field(ge=0.0)]

class CabBookingInput(BaseModel):
    pickup: str
    destination: str
    seats: Seats
    language: Language = "english"
    price_per_seat: Price = 150.0

class AgentState(TypedDict):
    messages: Annotated[list, add]
    bookings: Annotated[list, add]
    done: bool

booking = CabBookingInput(pickup="Nacharam", destination="Airport", seats=2, language="telugu")
print(booking)
print(f"Total: ₹{booking.seats * booking.price_per_seat}")
# Total: ₹300.0
```

---

### Pattern 11 — All examples summary

| Example | What it shows |
|---------|--------------|
| 1 | `Annotated` + Pydantic `Field` — constraints inside the type |
| 2 | `Annotated[list, add]` — LangGraph state reducer |
| 3 | `Annotated` + `AfterValidator` — reusable custom type |
| 4 | Real Aria use case — all combined |

---

## Pattern 12 — Full LangChain Agent

Every pattern combined. Four parts: state, tools, nodes, graph.

---

### Example 1 — the state

```python
from typing import Annotated, TypedDict
from operator import add

class AgentState(TypedDict):
    messages: Annotated[list, add]   # conversation history — appended
    actions:  Annotated[list, add]   # tools called — appended
    input:    str                    # user's current message
    output:   str                    # agent's final reply
    done:     bool                   # is the agent finished?
```

| Field | Purpose |
|-------|---------|
| `messages` | Full conversation — grows with every turn |
| `actions` | Log of every tool called |
| `input` | What the user just said |
| `output` | What Aria replies |
| `done` | Tells the graph when to stop |

---

### Example 2 — the tools

```python
from langchain.tools import tool
from pydantic import BaseModel

class WeatherInput(BaseModel):
    city: str

@tool("get_weather", args_schema=WeatherInput)
def get_weather(city: str) -> str:
    """Get the current weather for a city."""
    try:
        return f"Weather in {city}: Sunny, 28°C"
    except Exception as e:
        return f"Error: {e}"

class ReminderInput(BaseModel):
    person: str
    message: str
    urgent: bool = False

@tool("send_reminder", args_schema=ReminderInput)
def send_reminder(person: str, message: str, urgent: bool) -> str:
    """Send a reminder to a person."""
    try:
        prefix = "[URGENT] " if urgent else ""
        return f"{prefix}Reminder sent to {person}: {message}"
    except Exception as e:
        return f"Error: {e}"

tools = [get_weather, send_reminder]
```

Every tool: Pydantic input + `@tool` + `try/except`. Same structure every time.

---

### Example 3 — the nodes

```python
from langchain_anthropic import ChatAnthropic
from langchain_core.messages import HumanMessage, SystemMessage
from langgraph.prebuilt import ToolNode

model = ChatAnthropic(model="claude-sonnet-4-6").bind_tools(tools)

def agent_node(state: AgentState) -> dict:
    messages = [
        SystemMessage(content="You are Aria, Klement's personal assistant."),
        HumanMessage(content=state["input"])
    ] + state["messages"]
    response = model.invoke(messages)
    return {"messages": [response], "output": response.content}

tool_node = ToolNode(tools)

def should_continue(state: AgentState) -> str:
    last = state["messages"][-1]
    if hasattr(last, "tool_calls") and last.tool_calls:
        return "tools"
    return "end"
```

| Node | Job |
|------|-----|
| `agent_node` | Ask the LLM: what should I do? |
| `tool_node` | Execute the tool the LLM chose |
| `should_continue` | Decide: call a tool, or stop? |

---

### Example 4 — the graph

```python
from langgraph.graph import StateGraph, END

builder = StateGraph(AgentState)

builder.add_node("agent", agent_node)
builder.add_node("tools", tool_node)
builder.set_entry_point("agent")

builder.add_conditional_edges(
    "agent",
    should_continue,
    {"tools": "tools", "end": END}
)
builder.add_edge("tools", "agent")

graph = builder.compile()
```

```
START → agent_node → should_continue ──── "tools" → tool_node ─┐
                              └─────────── "end"   → END        │
                    ↑_________________________________________|
```

---

### Example 5 — running it

```python
result = graph.invoke({
    "input": "What is the weather in Hyderabad? Also remind Mom about her doctor at 3pm.",
    "messages": [],
    "actions": [],
    "output": "",
    "done": False
})

print(result["output"])
# Aria's full reply with weather + reminder confirmation
```

---

### Pattern 12 — All examples summary

| Example | What it shows |
|---------|--------------|
| 1 | State — TypedDict + Annotated reducers |
| 2 | Tools — Pydantic + @tool + try/except |
| 3 | Nodes — agent, tool executor, router |
| 4 | Graph — StateGraph, edges, loop |
| 5 | Running it — invoke + read results |

---

## All 12 Patterns — Complete

| # | Pattern | Status |
|---|---------|--------|
| 1 | Classes + OOP | ✓ |
| 2 | Type Hints | ✓ |
| 3 | TypedDict | ✓ |
| 4 | Pydantic BaseModel | ✓ |
| 5 | Decorators + `@tool` | ✓ |
| 6 | async/await | ✓ |
| 7 | try/except | ✓ |
| 8 | Dataclasses | ✓ |
| 9 | `**kwargs` | ✓ |
| 10 | List comprehensions | ✓ |
| 11 | Annotated types | ✓ |
| 12 | Full LangChain agent | ✓ |

