---
date: 2025-02-08
---
# **🔹 Useful References on Design Patterns and Advanced Python Concepts**

## **🔹 Singleton Pattern**  
📖 **Read More:** [Design Pattern in Python - Singleton](https://dev.to/jemaloq/design-pattern-in-python-singleton-5apj)  

---

## **🔹 Factory Pattern**  
📖 **Read More:** [Simple Factory in Python](https://dev.to/miguendes/design-patterns-that-make-sense-in-python-simple-factory-2f8c)  

---

## **🔹 Understanding Facade and Adapter Patterns**  
📖 **Read More:** [Proxy, Facade, and Adapter Pattern in Python](https://medium.com/analytics-vidhya/proxy-facade-and-adapter-pattern-using-python-69e5a0e72b42)  

---

## **🔹 Command-Query Segregation (CQS) Pattern**  
📖 **Read More:** [Command Query Segregation Implemented with Python](https://medium.com/swlh/command-query-segregation-implemented-with-python-4c7a8f733c3)  

---

## **🔹 Mixin Class in Python**  
📖 **Read More:** [Python Mixin Classes](https://www.pythontutorial.net/python-oop/python-mixin/)  

---

## **🔹 Code Challenge & Python Best Practices**  
📖 **Read More:** [Python Code Challenge #30](https://pybit.es/articles/codechallenge30/)  

---



# Understanding When to Use Various Design Patterns

📌 **Reference:** [Composition Over Inheritance](https://python-patterns.guide/gang-of-four/composition-over-inheritance/)

---

## Problem:

We need a better approach to designing a **logging system** that supports:
1. **Multiple filters** (e.g., filtering by priority, matching a keyword)
2. **Multiple output handlers** (e.g., writing logs to a file, sending logs over a network)

---

## Solution 1: **Adapter Pattern**  
> **Use when:** The original class **doesn’t need modification**, but you need to adapt its interface.

- The **existing logger class** expects a **file-like object** as input.
- The **Adapter Pattern** allows any logging mechanism to **wrap itself** to look like the expected file object.
- This enables logging to different outputs **without modifying** the original logger.

---

## Solution 2: **Bridge Pattern**  
> **Use when:** You need to **decouple abstraction from implementation**, allowing flexibility in how messages are logged.

- The logger class and **FilterLogger** class are modified.
- A **symmetric `write()` method** is added to all **implementation classes** (e.g., `FileHandler`, `SocketHandler`).
- **Filtering logic** is moved to the **abstraction** class (`Logger`).
- **Output handling** (e.g., writing to a file or socket) is moved to **implementation classes**.
- Each concrete implementation class now has an **`emit()` method**.

---

## Combining Solution 1 & 2: **Decorator Pattern**  
> **Use when:** You need to **apply multiple independent filters** dynamically.

- Supports **stacking multiple filters** (e.g., filter by priority, then by keyword).
- **Limitation:** Cannot stack multiple output routines.
- **Reason:**  
  - Filters provide a `log()` method but call the handler’s `emit()` method.
  - Wrapping one filter inside another results in an `AttributeError` when the outer filter tries to call `emit()`.
  - **While multiple filters can be stacked, multiple outputs cannot.**

---

## Custom Solution: **Supporting Multiple Filters & Outputs**  
> **Use when:** You need **both multiple filters and multiple output routines**.

### **Key Design Decisions:**
1. **Separate classes for**:
   - `Filter` (exposes a method `match()`)
   - `Handler` (exposes a method `emit()`)
2. **Logger class composition**:
   - The `Logger` class **holds instances** of `Filter` and `Handler`.
   - The `Logger` class exposes a `log()` method.

### **Implementation Example:**
```python
class Logger:
    def __init__(self, filters, handlers):
        self.filters = filters
        self.handlers = handlers

    def log(self, message):
        if all(f.match(message) for f in self.filters):
            for h in self.handlers:
                h.emit(message)
```
### **Pattern Selection Guide**

| **Pattern**       | **When to Use?** |
|------------------|----------------|
| **Adapter**     | When the original class **cannot be modified**, but needs a **different interface**. |
| **Bridge**      | When **filtering logic** should be **separate from output handling**. |
| **Decorator**   | When you need **multiple filters**, but **output methods cannot be stacked**. |
| **Custom Solution** | When you need **both multiple filters and multiple outputs**. |
---
# SOLID Principles: **Interface Segregation Principle (ISP)**

## **Use Case: Payment System**
Consider a **payment system** that provides two core functionalities:  
- **Debit**  
- **Credit**  

### **Current Design:**
We have two payment services, **GPay** and **PhonePe**, both implementing a common interface:

```plaintext
Interface Payment:
    - debit()
    - credit()

GPay      : Implements Payment Interface  → Must implement debit() and credit()
PhonePe   : Implements Payment Interface  → Must implement debit() and credit()
```
## **New Requirement:**
- **PhonePe** wants to support **Security Validation**.

## **Problem:**
- If we **add `SecurityValidation`** to the `Payment` interface, then **GPay** would also be **forced** to implement it, even if it doesn't require it.
- This **violates the Interface Segregation Principle (ISP)** by forcing **unnecessary methods** on classes that don’t need them.

## **Solution: Apply Interface Segregation Principle**
✅ **Create a separate interface** for `SecurityValidation`.  
✅ **Only the required classes** should implement it.

```plaintext
Interface Payment:
    - debit()
    - credit()

Interface SecurityValidation:
    - validateSecurity()

GPay      : Implements Payment Interface       → debit(), credit()
PhonePe   : Implements Payment & SecurityValidation Interfaces → debit(), credit(), validateSecurity()
```
## **By Applying the Interface Segregation Principle (ISP):**
✅ **Classes only implement the methods they need**, avoiding unnecessary dependencies.  
✅ **GPay is not forced** to implement `SecurityValidation`, keeping its implementation clean.  
✅ **The system remains modular, scalable, and easier to maintain**, allowing flexibility for future updates. 🚀  

# **SOLID Principles: Single Responsibility Principle (SRP)**

## **Definition:**
A **class should have only one responsibility**, meaning it should only have **one reason to change**.

## **Example: Journal Class (Violating SRP)**  
The `Journal` class below **manages journal entries** but also **handles file I/O operations**, violating **SRP**.

```python
class Journal:
    def __init__(self):
        self.entries = []
        self.count = 0

    def add_entry(self, text):
        self.entries.append(f"{self.count}: {text}")
        self.count += 1

    def remove_entry(self, pos):
        del self.entries[pos]

    def __str__(self):
        return "\n".join(self.entries)

    # ❌ Violates SRP: Handles persistence (saving/loading)
    def save(self, filename):
        file = open(filename, "w")
        file.write(str(self))
        file.close()

    def load(self, filename):
        pass

    def load_from_web(self, uri):
        pass
```
# **SOLID Principles: Single Responsibility Principle (SRP)**

## **Solution: Introduce a New Class for Persistence**
To adhere to **SRP**, we separate the **persistence (save/load) logic** from the `Journal` class by introducing a new **PersistenceManager** class.

```python
class PersistenceManager:
    def save_to_file(self, journal, filename):
        with open(filename, "w") as file:
            file.write(str(journal))
```
## Updated Implementation Following SRP
```python
class Journal:
    def __init__(self):
        self.entries = []
        self.count = 0

    def add_entry(self, text):
        self.entries.append(f"{self.count}: {text}")
        self.count += 1

    def remove_entry(self, pos):
        del self.entries[pos]

    def __str__(self):
        return "\n".join(self.entries)


# Creating a Journal instance
j = Journal()
j.add_entry("I cried today.")
j.add_entry("I ate a bug.")
print(f"Journal entries:\n{j}\n")

# Using PersistenceManager for saving
p = PersistenceManager()
file = r'c:\temp\journal.txt'
p.save_to_file(j, file)

# Verify file content
with open(file) as fh:
    print(fh.read())
```
## **Key Benefits of This Approach**
✅ **Journal class only manages journal entries** (its core responsibility).  
✅ **PersistenceManager class handles file operations**, ensuring modularity.  
✅ **Easy to extend**: We can modify storage logic (e.g., database, cloud) **without changing the Journal class**.  
✅ **Improves maintainability** and **follows SRP**, making the system **scalable and easier to manage**. 🚀  
# **SOLID Principles: Dependency Inversion Principle (DIP)**

## **Definition**
- **High-level modules should not depend on low-level modules. Both should depend on abstractions.**
- **Interfaces (or abstract classes) should be used to achieve abstraction.**

📌 **Reference:** [GeeksforGeeks - Dependency Inversion Principle](https://www.geeksforgeeks.org/dependecy-inversion-principle-solid/)

---

## **Example: Violating DIP**
In the example below, the `Manager` class directly depends on concrete classes (`Developer`, `Designer`, `Testers`).  
This **violates DIP** because changes in low-level classes will affect the `Manager` class.

```python
class Manager(object):
    def __init__(self):
        self.developers = []
        self.designers = []
        self.testers = []
  
    def addDeveloper(self, dev):
        self.developers.append(dev)
          
    def addDesigners(self, design):
        self.designers.append(design)
          
    def addTesters(self, testers):
        self.testers.append(testers)
  
class Developer(object):
    def __init__(self):
        print("Developer added")
      
class Designer(object):
    def __init__(self):
        print("Designer added")
      
class Testers(object):
    def __init__(self):
        print("Tester added")

# Creating objects
if __name__ == "__main__":
    manager = Manager()
    manager.addDeveloper(Developer())
    manager.addDesigners(Designer())
```
# **SOLID Principles: Improving Dependency Inversion Principle (DIP)**

## **Problem with the Previous Design**
Even though the previous design followed DIP by introducing the `IWorker` interface, **adding a new employee type (e.g., QA) still required modifying the `Manager` class**.  
This violates the **Open-Closed Principle (OCP)** since the `Manager` class is not truly open for extension.

---

## **Improved Solution: Introducing an Abstract `Employee` Class**
Instead of using `IWorker`, we **create an abstract class `Employee`** that all employees inherit from.  
Now, **Manager depends only on `Employee`**, making it **fully decoupled** from specific employee types.

```python
from abc import ABC, abstractmethod

# Abstract Employee class
class Employee(ABC):  
    @abstractmethod
    def work(self):
        pass

# Manager class depends only on the abstraction (Employee)
class Manager:
    def __init__(self):
        self.employees = []
    
    def addEmployee(self, employee: Employee):
        self.employees.append(employee)

    def manage(self):
        for employee in self.employees:
            employee.work()

# Concrete Employee Implementations
class Developer(Employee):
    def __init__(self):
        print("Developer added")
    
    def work(self):
        print("Turning coffee into code")

class Designer(Employee):
    def __init__(self):
        print("Designer added")
    
    def work(self):
        print("Turning lines into wireframes")

class Tester(Employee):
    def __init__(self):
        print("Tester added")
    
    def work(self):
        print("Testing everything out there")

# Example Usage
if __name__ == "__main__":
    manager = Manager()
    manager.addEmployee(Developer())
    manager.addEmployee(Designer())
    manager.addEmployee(Tester())
    
    manager.manage()
```
---

# **Decorator Pattern in Python**
### **Why Use Decorators?**
- **Avoids class explosion** by dynamically adding behavior instead of creating multiple subclasses.
- **Enhances flexibility** since functionality can be extended without modifying the original class.
- **Useful for logging, authentication, caching, and other cross-cutting concerns.**

---

## **Dynamic Decorator Example: Logging File Operations**
The following example demonstrates **a dynamic decorator** that wraps a file object and logs operations.

```python
class FileWithLogging:
    def __init__(self, file):
        self.file = file  # Store the wrapped file object

    def writelines(self, strings):
        self.file.writelines(strings)
        print(f'Wrote {len(strings)} lines')

    def __iter__(self):
        return iter(self.file)

    def __next__(self):
        return next(self.file)

    def __getattr__(self, item):
        """Delegates attribute access to the wrapped file object"""
        return getattr(self.__dict__['file'], item)

    def __setattr__(self, key, value):
        """Ensures attributes are set on the correct object"""
        if key == 'file':
            self.__dict__[key] = value
        else:
            setattr(self.__dict__['file'], key, value)

    def __delattr__(self, item):
        delattr(self.__dict__['file'], item)

# Usage Example
if __name__ == '__main__':
    file = FileWithLogging(open('hello.txt', 'w'))
    file.writelines(['hello\n', 'world\n'])  # Logs the number of lines written
    file.write('Testing\n')
    file.close()
```
# **Facade Design Pattern - Useful Resources**

## **1️⃣ Python Wife: Facade Design Pattern with Python**
📌 **Overview**:  
This article explains the **Facade Design Pattern** with an easy-to-understand example.  
It covers:
- What is the **Facade Pattern**?
- When to use it
- A **real-world example** using a **Pizza Ordering System**  
📖 **Read here:** [Facade Design Pattern with Python](https://pythonwife.com/facade-design-pattern-with-python/)

---

## **2️⃣ Medium: Facade Pattern in Python**
📌 **Overview**:  
This article provides an in-depth explanation of the **Facade Pattern** in Python.  
It includes:
- The **importance** of the pattern in software design
- How it simplifies complex subsystems
- A **step-by-step implementation** example  
📖 **Read here:** [Facade Pattern in Python](https://medium.com/design-patterns-in-python/facade-design-pattern-a29c94776870)
---
# **Factory Design Pattern**

## **🔹 What is a Factory Pattern?**
A **Factory** is typically any method that creates a new object. This approach follows the **Factory Method** pattern, which helps in **separating object creation** from the main logic.

## **🔹 When to Use?**
✅ When object creation should be separated from the main logic.  
✅ When a new object type might be introduced in the future, and we want to **avoid modifying existing code**.  

## **🔹 Use Case: Music Player & Video Player**
### **Scenario**
- You have **two classes**: `MusicPlayer` and `VideoPlayer`.
- Both classes need to **connect to devices** like **Alexa** and **Google Assistant**.
- The logic inside these classes contains **IF-ELSE conditions** to handle different devices.

### **Problem**
❌ If a **new device** is added, both `MusicPlayer` and `VideoPlayer` need modifications.  
❌ This violates the **Open/Closed Principle** and the **Dependency Inversion Principle**.  

### **Solution**
✅ Create a **new class called `DeviceFactory`**.  
✅ Move the **IF-ELSE conditions** into `DeviceFactory`.  
✅ Now, adding a **new device only requires modifying `DeviceFactory`**, not every player class.  

---

## **🔹 Simple Factory Example**
A **simple factory** can be created using a `@classmethod`:

```python
from abc import ABC, abstractmethod

class WaterTank(ABC):
    @abstractmethod
    def get_capacity(self):
        pass

class WaterTankFactory(WaterTank):
    def __init__(self, capacity):
        self.__capacity = capacity

    def get_capacity(self):
        return self.__capacity

    @classmethod
    def set_capacity(cls, capacityRequired):
        return cls(capacityRequired)

# Calling the Factory Method
tankOne = WaterTankFactory.set_capacity(500)
```
## **🔹 Factory Pattern using Python Dictionary**
Another way to implement the **Factory Design Pattern** in Python is by using a **dictionary-based factory**.  
This approach provides a **clean and scalable way** to create objects dynamically.

📖 **Reference:**  
[The Factory Pattern with Python](https://python.plainenglish.io/the-factory-pattern-with-python-30085a7a340e)

# **Proxy Design Pattern**  
The **Proxy Pattern** acts as a **wrapper** around an existing class. The external world interacts with the client **via the Proxy class**, which can introduce additional functionality.

📖 **Good Use Case for the Proxy Pattern:**  
🔗 [Implementing Proxy Pattern in Python](https://rednafi.github.io/reflections/implementing-proxy-pattern-in-python.html)

---

## **🔹 Protection Proxy**  
Used to add **filtering conditions** before allowing access to the underlying object.  
For example, consider a **Driver class** where only individuals **aged 16 or above** are allowed to drive.  
Instead of modifying the `Car` class, we introduce a **Proxy class** to enforce this rule.

### **Example:**
```python
class Car:
    def __init__(self, driver):
        self.driver = driver

    def drive(self):
        print(f'Car being driven by {self.driver.name}')

class CarProxy:
    def __init__(self, driver):
        self.driver = driver
        self.car = Car(driver)

    def drive(self):
        if self.driver.age >= 16:
            self.car.drive()
        else:
            print('Driver too young')

class Driver:
    def __init__(self, name, age):
        self.name = name
        self.age = age

if __name__ == '__main__':
    car = CarProxy(Driver('John', 12))  # Driver is underage
    car.drive()  # Output: Driver too young
```
## 🔹 Virtual Proxy  

The **Virtual Proxy Pattern** enables **lazy initialization**, meaning the object is **only created when needed**.  
This is particularly useful for **resource-intensive objects**, such as images in a graphics application.

### **Example:**
```python
class Bitmap:
    def __init__(self, filename):
        self.filename = filename
        print(f'Loading image from {filename}')

    def draw(self):
        print(f'Drawing image {self.filename}')

class LazyBitmap:
    def __init__(self, filename):
        self.filename = filename
        self.bitmap = None

    def draw(self):
        if not self.bitmap:
            self.bitmap = Bitmap(self.filename)  # Initialize only when needed
        self.bitmap.draw()

def draw_image(image):
    print('About to draw image')
    image.draw()
    print('Done drawing image')

if __name__ == '__main__':
    bmp = LazyBitmap('facepalm.jpg')  # Lazy initialization
    draw_image(bmp)
```
# **Decorator vs Proxy Pattern**  
## **📝 Similarities**  
✅ Both patterns **wrap an existing class** to modify behavior.  

## **🔍 Differences**  

### **1️⃣ Intention**  
- **Decorator Pattern**: Used to **add additional functionality** to an existing class without modifying it.  
- **Proxy Pattern**: Provides an **identical interface** to the existing class, often to **control access** or **optimize performance** (e.g., lazy loading).  

### **2️⃣ Usability**  
- **Decorator Pattern**:  
  - The original entity can be **directly instantiated** and used.  
  - The consumer can **directly interact** with the entity.  
  - **Example**: Adding logging, caching, or additional behaviors dynamically.  

- **Proxy Pattern**:  
  - The consumer interacts **only with the proxy**, not the actual entity.  
  - The proxy can **control access**, **delay initialization**, or **add security checks**.  
  - **Example**: Virtual Proxy for lazy loading, Protection Proxy for access control.  

### **3️⃣ Structural Difference**  
- **Decorator** **aggregates** (wraps) the object it decorates and extends functionality dynamically.  
- **Proxy** **acts as an intermediary** and does not necessarily modify behavior but **controls access**.  
---
# **Dependency Injection (DI)**  

## **🔍 What is Dependency Injection?**  
Dependency Injection is a **design pattern** that promotes **loose coupling** between components by ensuring that an object (the client) receives all the dependencies (services) it needs from an external source, rather than creating them itself.  

## **✅ Benefits of Dependency Injection**  
- **Improves modularity**: Components are independent and reusable.  
- **Enhances testability**: Dependencies can be easily mocked or replaced for unit testing.  
- **Follows the Single Responsibility Principle (SRP)**: The client does not need to manage dependency creation.  

## **📌 Example Implementation**  
For a detailed example, refer to:  
🔗 [Python Dependency Injection - TestDriven.io](https://testdriven.io/blog/python-dependency-injection/)  
# **Template Design Pattern**  

## **🔍 Overview**  
The **Template Design Pattern** is used when there is **behavioral duplication** across multiple classes. It defines the **skeleton of an algorithm** in a base (abstract) class, while allowing subclasses to provide specific implementations for certain steps.  

## **📌 Components of the Template Pattern**  
1. **Abstract Class**:  
   - Contains the **template method** (defines the workflow).  
   - Includes **primitive methods** (to be implemented by subclasses).  
2. **Template Method**:  
   - Defines the structure of an algorithm.  
   - Calls abstract methods that subclasses must override.  

## **✅ When to Use?**  
- When multiple classes share a similar process with **some variations**.  
- When **duplication of behavior** is observed in different classes.  

## **📖 Use Case Example**  
Consider **two reporting classes** that generate reports following the same process:  
- **Connect to SQL/PostgreSQL database**  
- **Query data from tables**  
- **Convert data to XML/JSON**  

Instead of duplicating the logic in both classes, we can create an **abstract class** with a `templateMethod()` and implement specific steps in subclasses.  

For a real-world example:  
🔗 [Template Method Pattern - Real-World Example (Java)](https://www.codiwan.com/template-method-design-pattern-real-world-example-java/)  
---
# **Mediator Design Pattern**  

## **🔍 When to Use?**  
The **Mediator Pattern** is useful when multiple entities **communicate with each other** to achieve a common goal. Instead of letting components communicate directly (leading to complex dependencies), a **mediator object** handles communication, making the system **more maintainable and decoupled**.  

## **📌 Key Benefits**  
✅ Reduces **tight coupling** between interacting components.  
✅ Improves **modularity** by centralizing communication logic.  
✅ Enhances **maintainability** as changes to interactions are handled in one place.  

## **✅ Use Case Example**  
### **🔹 System Failure Detector**  
A **System Failure Detector** can act as a **mediator**, communicating with internal system components to check for failures and returning a **consolidated result**.  
- Instead of each component checking others for failures, the **mediator** coordinates and simplifies the process.  
- This approach makes failure detection **scalable and modular**.  

# **Prototype Design Pattern**  

## **📌 When to Use?**  
The **Prototype Pattern** is useful when:  
✅ You need to **duplicate an object** while making modifications to the copied object.  
✅ You have an **existing design/class** and need to create a **copy** with additional features.  
✅ Object creation is **expensive**, and cloning helps improve **performance**.  

## **🔹 Implementation**  
The **Prototype Pattern** can be implemented using **deepcopy** to create an independent copy of an existing object.

### **🔹 Example Implementation**
```python
import copy

class Prototype:
    def __init__(self, value):
        self.value = value

    def clone(self):
        return copy.deepcopy(self)

# Create an original object
original = Prototype(10)

# Create a cloned object and modify it
clone = original.clone()
clone.value = 20

print(f"Original Value: {original.value}")  # Output: 10
print(f"Cloned Value: {clone.value}")  # Output: 20
```
# **Adapter Design Pattern**  

## **📌 When to Use?**  
The **Adapter Pattern** is useful when:  
✅ You have an **existing interface** but need a **different interface** to work with it.  
✅ You want to **reuse existing code** without modifying its implementation.  
✅ You need to **bridge incompatible interfaces** between different systems.  

## **🔹 Adapter vs. Bridge Design Pattern**
| **Pattern** | **Purpose** |
|------------|------------|
| **Adapter** | Converts one interface into another expected interface. |
| **Bridge** | Avoids the **cartesian product explosion** by separating abstraction and implementation into two independent hierarchies. |

## **🔹 Bridge Design Pattern**
The **Bridge Pattern** is used to **connect two hierarchies of classes**.  
Instead of **inheriting multiple classes**, the idea is to **pass the parent class of one hierarchy into another**.

### **Example:**
We have two independent class hierarchies:  
🔹 **Shapes** → `Circle`, `Square`  
🔹 **Rendering Mechanisms** → `Vector`, `Raster`  

```python
# Abstraction
class Shape:
    def __init__(self, renderer):
        self.renderer = renderer

    def draw(self):
        raise NotImplementedError

# Concrete Implementation 1
class Circle(Shape):
    def draw(self):
        print(f"Drawing Circle using {self.renderer.render()}")

# Concrete Implementation 2
class Square(Shape):
    def draw(self):
        print(f"Drawing Square using {self.renderer.render()}")

# Implementor
class Renderer:
    def render(self):
        raise NotImplementedError

# Concrete Implementors
class VectorRenderer(Renderer):
    def render(self):
        return "Vector Renderer"

class RasterRenderer(Renderer):
    def render(self):
        return "Raster Renderer"

# Usage
vector_renderer = VectorRenderer()
circle = Circle(vector_renderer)
circle.draw()  # Output: Drawing Circle using Vector Renderer

raster_renderer = RasterRenderer()
square = Square(raster_renderer)
square.draw()  # Output: Drawing Square using Raster Renderer
```
# **Builder Design Pattern**  

## **📌 What is the Builder Pattern?**  
The **Builder Pattern** is a **creational design pattern** that helps in the **step-by-step construction** of complex objects.  
It allows us to create objects **piece by piece** rather than all at once.  

## **🔹 Key Features of Builder Pattern**
- **Encapsulates the construction logic** of an object.
- **Improves code readability** by separating object creation logic from business logic.
- **Supports different object representations** (e.g., different configurations of the same object).  

## **🔹 Explicit vs. Implicit Object Construction**
| **Type** | **Description** |
|----------|---------------|
| **Explicit Construction** | Uses constructors directly to create objects. |
| **Implicit Construction** | Uses Dependency Injection (DI) or Reflection to create objects dynamically. |

## **🔹 Whole vs. Piece-by-Piece Object Creation**
| **Type** | **Description** |
|----------|---------------|
| **Whole Construction** | The object is built **in a single step** (e.g., using a constructor). |
| **Piece-by-Piece** | The object is **constructed gradually**, allowing **custom configurations**. |

---

## **🔹 Structural Patterns in Builder**
The **Builder Pattern** can also be categorized as a **Structural Pattern** when it is used to **wrap an existing API** to simplify object creation.

---

## **🔹 Example: Query Builder**
The Builder Pattern is often used in **SQL Query Builders** to construct complex queries dynamically.

### **📖 References**
- **Python Query Builder**: [GitHub Link](https://github.com/zgulde/python-query-builder/blob/trunk/query.py)
- **SQL Query Builder**: [GitHub Link](https://github.com/JarkoDubbeldam/SQL-builder/blob/master/classes.py)

🔹 These repositories demonstrate how the **Builder Pattern** can be used to construct SQL queries efficiently.
---
# **Chain of Responsibility Pattern**  

## **📌 What is the Chain of Responsibility Pattern?**  
The **Chain of Responsibility** is a **behavioral design pattern** that allows a request to be processed by multiple handlers in a sequence.  
Each handler decides whether to process the request or pass it to the next handler in the chain.

## **🔹 When to Use?**  
- When multiple steps need to be executed **in a predefined sequence**.  
- When each step has a specific responsibility, and failure at any step should **stop the process**.  

---

## **🔹 Use Cases**  
### **1️⃣ Chain Breaker (Failure Stops the Process)**
- Used when **failure at any step** should terminate the execution.
- Example:  
  - **Interview Rounds:** A candidate must pass each round before moving to the next.  
  - **Request Processing:**  
    ```plaintext
    Build(req) --> Log(req) --> Validate(req) --> Execute(req)
    ```
    If **any step fails, the process stops**.

### **2️⃣ Non-Breaking Chain (All Steps Execute)**
- Used when **all steps must execute**, even if one fails.  
- Example: **Coffee Preparation Process**
  ```plaintext
  AddCoffee(container) --> AddMilk(container) --> AddSugar(container)
  ```
## **🔹 Implementation in Python**  
The **Chain of Responsibility Pattern** can be implemented in Python by creating a sequence of handlers that process a request **one step at a time**.



## **🔗 Reference Implementation**  
🔹 **GitHub Repository:** [Python Chain of Responsibility](https://github.com/Sunuba/PythonChainOfResponsibility/tree/master/pattern_classes)  

This repository demonstrates how to implement the **Chain of Responsibility Pattern** effectively in Python.
---
# **State Design Pattern**  

## **🔹 Overview**  
The **State Design Pattern** is used when an object's behavior **varies based on its internal state**.  
Common use cases include:  
✔ Ordering System  
✔ Reservation System  
✔ Ticketing System (e.g., JIRA)  
✔ Database Connector Service  

---

## **🔹 Use Case: E-commerce Ordering System**  
An **e-commerce order** transitions through multiple states, such as:  
1️⃣ **Order Placed**  
2️⃣ **Payment Processed**  
3️⃣ **Shipped**  
4️⃣ **Delivered**  

Using the **State Pattern**, we can define each state as a separate class and allow the order to transition dynamically between them.  
---
# **Domain Modelling**  

## **🔹 What is Domain Modelling?**  
Domain Modelling is a **framework for thinking** that helps developers understand how their code scales **with respect to requirements**—not just input size.  

Many developers are familiar with **Big-O notation**, which describes how an algorithm scales with input size.  
However, **how does code scale when new requirements are added**?  

**Domain Modelling** helps answer this question by structuring software in a way that accommodates evolving requirements efficiently.

---

## **🔹 Why is it Important?**  
✔ Helps in designing flexible and scalable systems  
✔ Reduces the complexity of adding new features  
✔ Prevents code from becoming unmanageable as requirements grow  

📖 **Reference:**  
🔗 [Domain Modelling – A Problem You Didn’t Know You Have](https://medium.com/python-in-plain-english/domain-modelling-1-a-problem-you-didnt-know-you-have-56e072bd3c1)  
---
# **Singleton Design Pattern**  

## **🔹 When to Use Singleton?**  
The **Singleton Pattern** is used when **resource consumption** is high, and we need to ensure that only **one instance** of a class exists across the application.  

### **Use Cases:**  
✔ **Database Connection** – Creating and maintaining a DB connection is expensive. Singleton ensures only one connection instance is shared.  
✔ **Application-Level Configuration** – Global settings that should be accessible throughout the application.  

---

## **🔹 Multiple Ways to Implement Singleton in Python**  

### **1️⃣ Singleton Allocator**  
- Uses a **class variable** that is initialized during instantiation.  
- Implemented using the `__new__` method.  
- **Limitation:** The class is initialized every time it's instantiated, even though there's only one instance.  

### **2️⃣ Alternative Implementations**  
To **overcome the above limitation**, Singleton can be implemented in different ways:  
✅ **Singleton Decorator** – Uses a function decorator to enforce single instance creation.  
✅ **Singleton Metaclass** – Ensures only one instance exists by controlling object creation at the metaclass level.  
✅ **Singleton Monostate (Borg Pattern)** – Shares state among multiple instances instead of restricting instantiation.  



📖 **Reference:** Stay tuned for code implementations of these approaches! 🚀  
---
# **Flyweight Design Pattern**  

## **🔹 When to Use Flyweight?**  
The **Flyweight Pattern** is used to **avoid redundancy** when storing and managing objects. It **optimizes memory usage** by sharing common parts of objects instead of creating new instances.

### **Use Case:**  
📌 **Student Object Example:**  
- Suppose we have a `Student` class that contains **grade-related information**.  
- Instead of creating a **new `Grade` object** every time, we can **reuse an existing instance** if the grade already exists.  
- This significantly reduces **memory consumption** and improves **performance**.  



## **🔹 Benefits of Flyweight Pattern**  
✅ **Reduces memory usage** by reusing existing instances.  
✅ **Improves performance** by avoiding unnecessary object creation.  
✅ **Useful for large-scale applications** where repeated objects can be shared.  



📖 **Reference:** Stay tuned for implementation details! 🚀  
---