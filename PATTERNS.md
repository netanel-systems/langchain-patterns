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
| 7 | try/except — error handling | Remaining |
| 8 | Dataclasses | Remaining |
| 9 | `**kwargs` — flexible arguments | Remaining |
| 10 | List comprehensions | Remaining |
| 11 | Annotated types | Remaining |
| 12 | Putting it all together — full LangChain agent | Remaining |

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
```
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
```
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
```
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
```
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
```
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
```
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
```
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
```
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
```
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
```
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
```
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
```
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
```
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
```
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
```
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
```
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
```
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
```
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
```
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
```
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
```
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
```
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
```
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
```
Weather in Hyderabad: Sunny, 28°C
Reminder for Mom: Doctor at 3pm
```

The only change from sync: `.invoke()` → `.ainvoke()`. Same tool, same input
dict. Now it runs inside `asyncio.gather`.

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

