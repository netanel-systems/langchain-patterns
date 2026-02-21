# Python Patterns for LangChain

Klement's reference guide — built pattern by pattern, example by example.

---

## Pattern 1 — Classes + OOP

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

### Level 4 — Inheritance

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

---

## Pattern 2 — Type Hints

### Basic types

```python
def add(a: int, b: int) -> int:
    return a + b

def greet(name: str) -> str:
    return f"Hello, {name}!"

def is_adult(age: int) -> bool:
    return age >= 18
```

### list and dict

```python
def total(scores: list[int]) -> int:
    return sum(scores)

def get_grade(scores: dict[str, int]) -> str:
    average = sum(scores.values()) / len(scores)
    if average >= 90:
        return "A"
    elif average >= 75:
        return "B"
    return "C"
```

### Optional — value or nothing

```python
from typing import Optional

def book_cab(pickup: str, destination: str, note: Optional[str] = None) -> str:
    if note:
        return f"Cab booked: {pickup} → {destination}. Note: {note}"
    return f"Cab booked: {pickup} → {destination}"

print(book_cab("Nacharam", "Airport"))               # no note
print(book_cab("Nacharam", "Airport", "Call mom"))   # with note
```

### Union — one type OR another

```python
from typing import Union

def display(value: Union[int, str]) -> str:
    return f"Received: {value}"

print(display(42))       # int
print(display("hello"))  # str
```

### Literal — must be one of these exact values

```python
from typing import Literal

def set_mode(mode: Literal["read", "write", "admin"]) -> str:
    return f"Mode set to: {mode}"

print(set_mode("read"))   # valid
print(set_mode("admin"))  # valid
# set_mode("delete")      # rejected by Pydantic
```

---

## Pattern 3 — TypedDict

Used for **agent state** in LangGraph — the memory that flows between nodes.
You write it. You control it. No enforcement needed.

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

### Cab booking state

```python
from typing import TypedDict

class CabBooking(TypedDict):
    pickup: str
    destination: str
    seats: int
    confirmed: bool

def summarize(booking: CabBooking) -> str:
    status = "Confirmed" if booking["confirmed"] else "Pending"
    return f"{booking['pickup']} → {booking['destination']} | {booking['seats']} seats | {status}"
```

### LangGraph node — state in, state out

```python
from typing import TypedDict

class AgentState(TypedDict):
    message: str
    done: bool
    reply: str

def process(state: AgentState) -> AgentState:
    state["reply"] = f"Got it: {state['message']}"
    state["done"] = True
    return state
```

---

## Pattern 4 — Pydantic BaseModel

Used for **tool inputs** in LangChain — the LLM fills these.
Pydantic validates every field. Wrong type = immediate error.

Access with dot `.` — not `["key"]` like a dict.

### Basic BaseModel

```python
from pydantic import BaseModel

class CabBooking(BaseModel):
    pickup: str
    destination: str
    seats: int

booking = CabBooking(pickup="Nacharam", destination="Airport", seats=2)
print(booking.pickup)
print(booking.seats)
```

### Optional field

```python
from pydantic import BaseModel
from typing import Optional

class Contact(BaseModel):
    name: str
    phone: str
    note: Optional[str] = None

c1 = Contact(name="Klement", phone="+91-9999")            # note = None
c2 = Contact(name="Mom", phone="+91-8888", note="Telugu")  # note filled
```

### Literal inside BaseModel

```python
from pydantic import BaseModel
from typing import Literal

class ReminderInput(BaseModel):
    person: str
    message: str
    type: Literal["call", "cab", "food"]

# type="email" → Pydantic crashes: Input should be 'call', 'cab' or 'food'
```

### Nested models

```python
from pydantic import BaseModel

class Address(BaseModel):
    city: str
    country: str

class Person(BaseModel):
    name: str
    age: int
    address: Address

klement = Person(name="Klement", age=25, address=Address(city="New York", country="USA"))
print(klement.address.city)     # New York
print(klement.address.country)  # USA
```

### model_validator — custom rule across fields

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
```

### Field with number constraints

```python
from pydantic import BaseModel, Field

class OrderInput(BaseModel):
    item: str
    quantity: int = Field(ge=1, le=10)
    discount: float = Field(ge=0.0, le=100.0)
```

### All combined — real Aria tool input

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

---

## TypedDict vs Pydantic — Quick Reference

| | TypedDict | Pydantic BaseModel |
|---|---|---|
| Syntax | `class X(TypedDict)` | `class X(BaseModel)` |
| Enforces types? | No | Yes |
| Access | `state["key"]` | `obj.key` |
| Used for | LangGraph state (you write it) | LangChain tool inputs (LLM writes it) |

**Rule:** LLM writes it → Pydantic. You write it → TypedDict.

---

*Patterns 5–12 coming soon.*
