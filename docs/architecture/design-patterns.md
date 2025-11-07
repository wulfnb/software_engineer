# Design Patterns Compendium

## Introduction

Design patterns are typical solutions to common problems in software design. This guide provides Python-specific implementations.
Python-Specific Benefits

    Duck Typing: Flexible interface implementations

    First-Class Functions: Enables functional approaches

    Decorators: Built-in support for decorator pattern

    Context Managers: Resource management made easy

    Generator Expressions: Efficient iteration patterns

## Creational Patterns
### 1. Singleton Pattern

Intent: Ensure a class has only one instance.
```python

class Singleton:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

# Pythonic approach using module
# singleton.py
class _Singleton:
    def __init__(self):
        self.data = "I'm a singleton"

_instance = _Singleton()

def get_instance():
    return _instance

# Usage
singleton1 = Singleton()
singleton2 = Singleton()
print(singleton1 is singleton2)  # True

# Module approach
from singleton import get_instance
instance = get_instance()
```
Use Cases:

    Database connections

    Configuration managers

    Logging systems

### 2. Factory Method Pattern

Intent: Define an interface for creating objects, but let subclasses decide which class to instantiate.
```python

from abc import ABC, abstractmethod
from enum import Enum

class PaymentType(Enum):
    CREDIT_CARD = "credit_card"
    PAYPAL = "paypal"
    BANK_TRANSFER = "bank_transfer"

class PaymentProcessor(ABC):
    @abstractmethod
    def process_payment(self, amount: float) -> bool:
        pass

class CreditCardProcessor(PaymentProcessor):
    def process_payment(self, amount: float) -> bool:
        print(f"Processing ${amount} via Credit Card")
        return True

class PayPalProcessor(PaymentProcessor):
    def process_payment(self, amount: float) -> bool:
        print(f"Processing ${amount} via PayPal")
        return True

class PaymentProcessorFactory:
    @staticmethod
    def create_processor(payment_type: PaymentType) -> PaymentProcessor:
        processors = {
            PaymentType.CREDIT_CARD: CreditCardProcessor,
            PaymentType.PAYPAL: PayPalProcessor,
        }
        
        processor_class = processors.get(payment_type)
        if not processor_class:
            raise ValueError(f"Unknown payment type: {payment_type}")
        
        return processor_class()

# Usage
processor = PaymentProcessorFactory.create_processor(PaymentType.CREDIT_CARD)
processor.process_payment(100.0)
```

### 3. Abstract Factory Pattern

Intent: Create families of related objects without specifying concrete classes.
```python

from abc import ABC, abstractmethod

# Abstract Products
class Button(ABC):
    @abstractmethod
    def render(self) -> str:
        pass

class Checkbox(ABC):
    @abstractmethod
    def render(self) -> str:
        pass

# Concrete Products
class WindowsButton(Button):
    def render(self) -> str:
        return "Windows-style Button"

class WindowsCheckbox(Checkbox):
    def render(self) -> str:
        return "Windows-style Checkbox"

class MacButton(Button):
    def render(self) -> str:
        return "Mac-style Button"

class MacCheckbox(Checkbox):
    def render(self) -> str:
        return "Mac-style Checkbox"

# Abstract Factory
class GUIFactory(ABC):
    @abstractmethod
    def create_button(self) -> Button:
        pass
    
    @abstractmethod
    def create_checkbox(self) -> Checkbox:
        pass

# Concrete Factories
class WindowsFactory(GUIFactory):
    def create_button(self) -> Button:
        return WindowsButton()
    
    def create_checkbox(self) -> Checkbox:
        return WindowsCheckbox()

class MacFactory(GUIFactory):
    def create_button(self) -> Button:
        return MacButton()
    
    def create_checkbox(self) -> Checkbox:
        return MacCheckbox()

# Client
class Application:
    def __init__(self, factory: GUIFactory):
        self.factory = factory
        self.button = None
        self.checkbox = None
    
    def create_ui(self):
        self.button = self.factory.create_button()
        self.checkbox = self.factory.create_checkbox()
    
    def render(self):
        return f"{self.button.render()} and {self.checkbox.render()}"

# Usage
windows_app = Application(WindowsFactory())
windows_app.create_ui()
print(windows_app.render())  # "Windows-style Button and Windows-style Checkbox"
```

### 4. Builder Pattern

Intent: Construct complex objects step by step.
```python

from typing import List

class Computer:
    def __init__(self):
        self.cpu = None
        self.ram = None
        self.storage = None
        self.gpu = None
    
    def __str__(self):
        return f"Computer: CPU={self.cpu}, RAM={self.ram}, Storage={self.storage}, GPU={self.gpu}"

class ComputerBuilder:
    def __init__(self):
        self.computer = Computer()
    
    def set_cpu(self, cpu: str):
        self.computer.cpu = cpu
        return self
    
    def set_ram(self, ram: str):
        self.computer.ram = ram
        return self
    
    def set_storage(self, storage: str):
        self.computer.storage = storage
        return self
    
    def set_gpu(self, gpu: str):
        self.computer.gpu = gpu
        return self
    
    def build(self):
        return self.computer

# Director
class ComputerDirector:
    @staticmethod
    def build_gaming_computer():
        return (ComputerBuilder()
                .set_cpu("Intel i9")
                .set_ram("32GB DDR5")
                .set_storage("2TB NVMe SSD")
                .set_gpu("NVIDIA RTX 4090")
                .build())
    
    @staticmethod
    def build_office_computer():
        return (ComputerBuilder()
                .set_cpu("Intel i5")
                .set_ram("16GB DDR4")
                .set_storage("512GB SSD")
                .build())

# Usage
gaming_pc = ComputerDirector.build_gaming_computer()
office_pc = ComputerDirector.build_office_computer()

print(gaming_pc)   # Computer: CPU=Intel i9, RAM=32GB DDR5, Storage=2TB NVMe SSD, GPU=NVIDIA RTX 4090
print(office_pc)   # Computer: CPU=Intel i5, RAM=16GB DDR4, Storage=512GB SSD, GPU=None
```

### 5. Prototype Pattern

Intent: Create new objects by copying existing instances.
```python

import copy
from typing import Any

class Prototype:
    def __init__(self):
        self._objects = {}
    
    def register_object(self, name: str, obj: Any):
        self._objects[name] = obj
    
    def unregister_object(self, name: str):
        del self._objects[name]
    
    def clone(self, name: str, **attrs):
        obj = copy.deepcopy(self._objects.get(name))
        obj.__dict__.update(attrs)
        return obj

class Car:
    def __init__(self, make: str, model: str, color: str):
        self.make = make
        self.model = model
        self.color = color
    
    def __str__(self):
        return f"{self.color} {self.make} {self.model}"

# Usage
prototype = Prototype()

# Create prototype cars
default_car = Car("Toyota", "Camry", "White")
sports_car = Car("Porsche", "911", "Red")

prototype.register_object("default_car", default_car)
prototype.register_object("sports_car", sports_car)

# Clone and customize
car1 = prototype.clone("default_car", color="Blue")
car2 = prototype.clone("sports_car", color="Black")

print(car1)  # Blue Toyota Camry
print(car2)  # Black Porsche 911
```

## Structural Patterns
### 1. Adapter Pattern

Intent: Make incompatible interfaces work together.
```python

from abc import ABC, abstractmethod

# Old interface
class OldPrinter:
    def print_document(self, text: str):
        print(f"Printing: {text}")

# New interface
class NewPrinter(ABC):
    @abstractmethod
    def print(self, content: str, copies: int = 1):
        pass

# Adapter
class PrinterAdapter(NewPrinter):
    def __init__(self, old_printer: OldPrinter):
        self.old_printer = old_printer
    
    def print(self, content: str, copies: int = 1):
        for i in range(copies):
            self.old_printer.print_document(f"Copy {i+1}: {content}")

# Usage
old_printer = OldPrinter()
adapter = PrinterAdapter(old_printer)
adapter.print("Hello World", copies=3)
```

### 2. Decorator Pattern

Intent: Add responsibilities to objects dynamically.
``` python

from abc import ABC, abstractmethod
from functools import wraps

# Using classes
class Coffee(ABC):
    @abstractmethod
    def cost(self) -> float:
        pass
    
    @abstractmethod
    def description(self) -> str:
        pass

class SimpleCoffee(Coffee):
    def cost(self) -> float:
        return 2.0
    
    def description(self) -> str:
        return "Simple coffee"

class CoffeeDecorator(Coffee):
    def __init__(self, coffee: Coffee):
        self._coffee = coffee
    
    def cost(self) -> float:
        return self._coffee.cost()
    
    def description(self) -> str:
        return self._coffee.description()

class MilkDecorator(CoffeeDecorator):
    def cost(self) -> float:
        return self._coffee.cost() + 0.5
    
    def description(self) -> str:
        return self._coffee.description() + ", milk"

class SugarDecorator(CoffeeDecorator):
    def cost(self) -> float:
        return self._coffee.cost() + 0.2
    
    def description(self) -> str:
        return self._coffee.description() + ", sugar"

# Using Python decorators
def logger_decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__} with args: {args}, kwargs: {kwargs}")
        result = func(*args, **kwargs)
        print(f"{func.__name__} returned: {result}")
        return result
    return wrapper

@logger_decorator
def add_numbers(a: int, b: int) -> int:
    return a + b

# Usage
coffee = SimpleCoffee()
print(f"{coffee.description()}: ${coffee.cost()}")

coffee_with_milk = MilkDecorator(coffee)
print(f"{coffee_with_milk.description()}: ${coffee_with_milk.cost()}")

coffee_with_milk_and_sugar = SugarDecorator(coffee_with_milk)
print(f"{coffee_with_milk_and_sugar.description()}: ${coffee_with_milk_and_sugar.cost()}")

# Using function decorator
result = add_numbers(5, 3)
print(f"Result: {result}")
```

### 3. Facade Pattern

Intent: Provide a simplified interface to a complex subsystem.
```python

class CPU:
    def start(self):
        print("CPU starting...")
    
    def execute(self):
        print("CPU executing commands...")

class Memory:
    def load(self):
        print("Memory loading data...")

class HardDrive:
    def read(self):
        print("Hard drive reading data...")

# Facade
class ComputerFacade:
    def __init__(self):
        self.cpu = CPU()
        self.memory = Memory()
        self.hard_drive = HardDrive()
    
    def start_computer(self):
        print("Starting computer...")
        self.cpu.start()
        self.memory.load()
        self.hard_drive.read()
        self.cpu.execute()
        print("Computer started successfully!")

# Usage
computer = ComputerFacade()
computer.start_computer()
```

### 4. Proxy Pattern

Intent: Control access to an object.
```python

from abc import ABC, abstractmethod

class Subject(ABC):
    @abstractmethod
    def request(self) -> str:
        pass

class RealSubject(Subject):
    def request(self) -> str:
        return "RealSubject: Handling request."

class Proxy(Subject):
    def __init__(self, real_subject: RealSubject):
        self._real_subject = real_subject
    
    def request(self) -> str:
        if self.check_access():
            result = self._real_subject.request()
            self.log_access()
            return result
        else:
            return "Proxy: Access denied."
    
    def check_access(self) -> bool:
        print("Proxy: Checking access permissions...")
        return True
    
    def log_access(self):
        print("Proxy: Logging the time of request.")

# Lazy Proxy
class LazyProxy(Subject):
    def __init__(self):
        self._real_subject = None
    
    def request(self) -> str:
        if self._real_subject is None:
            print("LazyProxy: Creating RealSubject...")
            self._real_subject = RealSubject()
        return self._real_subject.request()

# Usage
print("Client: Executing with real subject:")
real_subject = RealSubject()
print(real_subject.request())

print("\nClient: Executing with proxy:")
proxy = Proxy(real_subject)
print(proxy.request())

print("\nClient: Executing with lazy proxy:")
lazy_proxy = LazyProxy()
print(lazy_proxy.request())
```

### 5. Composite Pattern

Intent: Compose objects into tree structures.
```python

from abc import ABC, abstractmethod
from typing import List

class FileSystemComponent(ABC):
    @abstractmethod
    def show_details(self, indent: str = "") -> str:
        pass

class File(FileSystemComponent):
    def __init__(self, name: str):
        self.name = name
    
    def show_details(self, indent: str = "") -> str:
        return f"{indent}File: {self.name}"

class Directory(FileSystemComponent):
    def __init__(self, name: str):
        self.name = name
        self.children: List[FileSystemComponent] = []
    
    def add(self, component: FileSystemComponent):
        self.children.append(component)
    
    def remove(self, component: FileSystemComponent):
        self.children.remove(component)
    
    def show_details(self, indent: str = "") -> str:
        result = [f"{indent}Directory: {self.name}"]
        for child in self.children:
            result.append(child.show_details(indent + "  "))
        return "\n".join(result)

# Usage
root = Directory("Root")
home = Directory("Home")
documents = Directory("Documents")

file1 = File("file1.txt")
file2 = File("file2.txt")
file3 = File("resume.pdf")

root.add(home)
home.add(file1)
home.add(documents)
documents.add(file2)
documents.add(file3)

print(root.show_details())
```

## Behavioral Patterns
### 1. Observer Pattern

Intent: Define one-to-many dependencies between objects.
```python

from abc import ABC, abstractmethod
from typing import List

class Observer(ABC):
    @abstractmethod
    def update(self, message: str):
        pass

class Subject(ABC):
    @abstractmethod
    def attach(self, observer: Observer):
        pass
    
    @abstractmethod
    def detach(self, observer: Observer):
        pass
    
    @abstractmethod
    def notify(self):
        pass

class NewsAgency(Subject):
    def __init__(self):
        self._observers: List[Observer] = []
        self._news = ""
    
    def attach(self, observer: Observer):
        self._observers.append(observer)
    
    def detach(self, observer: Observer):
        self._observers.remove(observer)
    
    def notify(self):
        for observer in self._observers:
            observer.update(self._news)
    
    def set_news(self, news: str):
        self._news = news
        self.notify()

class NewsChannel(Observer):
    def __init__(self, name: str):
        self.name = name
    
    def update(self, message: str):
        print(f"{self.name} received news: {message}")

# Using Python's built-in observer pattern with properties
class Observable:
    def __init__(self):
        self._observers = []
    
    def add_observer(self, observer):
        self._observers.append(observer)
    
    def remove_observer(self, observer):
        self._observers.remove(observer)
    
    def notify_observers(self, *args, **kwargs):
        for observer in self._observers:
            observer(*args, **kwargs)

class WeatherStation:
    def __init__(self):
        self._temperature = 0
        self._observable = Observable()
    
    @property
    def temperature(self):
        return self._temperature
    
    @temperature.setter
    def temperature(self, value):
        self._temperature = value
        self._observable.notify_observers(self._temperature)
    
    def add_observer(self, observer):
        self._observable.add_observer(observer)

# Usage
agency = NewsAgency()

channel1 = NewsChannel("CNN")
channel2 = NewsChannel("BBC")

agency.attach(channel1)
agency.attach(channel2)

agency.set_news("Breaking: Python 4.0 released!")

# Using property-based observer
def weather_alert(temp):
    print(f"Weather alert: Temperature is now {temp}°C")

station = WeatherStation()
station.add_observer(weather_alert)
station.temperature = 25
station.temperature = 30
```

### 2. Strategy Pattern

Intent: Define a family of algorithms and make them interchangeable.
```python

from abc import ABC, abstractmethod
from typing import List

class SortingStrategy(ABC):
    @abstractmethod
    def sort(self, data: List) -> List:
        pass

class BubbleSortStrategy(SortingStrategy):
    def sort(self, data: List) -> List:
        print("Using bubble sort")
        # Simplified bubble sort implementation
        return sorted(data)

class QuickSortStrategy(SortingStrategy):
    def sort(self, data: List) -> List:
        print("Using quick sort")
        # Simplified quick sort implementation
        return sorted(data)

class MergeSortStrategy(SortingStrategy):
    def sort(self, data: List) -> List:
        print("Using merge sort")
        # Simplified merge sort implementation
        return sorted(data)

class Sorter:
    def __init__(self, strategy: SortingStrategy = None):
        self._strategy = strategy
    
    def set_strategy(self, strategy: SortingStrategy):
        self._strategy = strategy
    
    def sort(self, data: List) -> List:
        if not self._strategy:
            raise ValueError("Sorting strategy not set")
        return self._strategy.sort(data)

# Using functions as strategies (Pythonic approach)
def bubble_sort(data: List) -> List:
    print("Using bubble sort (functional)")
    return sorted(data)

def quick_sort(data: List) -> List:
    print("Using quick sort (functional)")
    return sorted(data)

class FunctionalSorter:
    def __init__(self, sort_strategy=None):
        self.sort_strategy = sort_strategy or sorted
    
    def sort(self, data: List) -> List:
        return self.sort_strategy(data)

# Usage
data = [5, 2, 8, 1, 9]

# Class-based approach
sorter = Sorter(BubbleSortStrategy())
result = sorter.sort(data)
print(result)

sorter.set_strategy(QuickSortStrategy())
result = sorter.sort(data)
print(result)

# Functional approach
functional_sorter = FunctionalSorter(bubble_sort)
result = functional_sorter.sort(data)
print(result)

functional_sorter.sort_strategy = quick_sort
result = functional_sorter.sort(data)
print(result)
```

### 3. Command Pattern

Intent: Encapsulate requests as objects.
```python

from abc import ABC, abstractmethod
from typing import List

class Command(ABC):
    @abstractmethod
    def execute(self):
        pass
    
    @abstractmethod
    def undo(self):
        pass

class Light:
    def turn_on(self):
        print("Light is ON")
    
    def turn_off(self):
        print("Light is OFF")

class LightOnCommand(Command):
    def __init__(self, light: Light):
        self.light = light
    
    def execute(self):
        self.light.turn_on()
    
    def undo(self):
        self.light.turn_off()

class LightOffCommand(Command):
    def __init__(self, light: Light):
        self.light = light
    
    def execute(self):
        self.light.turn_off()
    
    def undo(self):
        self.light.turn_on()

class RemoteControl:
    def __init__(self):
        self._on_commands: List[Command] = []
        self._off_commands: List[Command] = []
        self._history: List[Command] = []
    
    def set_command(self, slot: int, on_command: Command, off_command: Command):
        self._on_commands.insert(slot, on_command)
        self._off_commands.insert(slot, off_command)
    
    def press_on_button(self, slot: int):
        if slot < len(self._on_commands) and self._on_commands[slot]:
            command = self._on_commands[slot]
            command.execute()
            self._history.append(command)
    
    def press_off_button(self, slot: int):
        if slot < len(self._off_commands) and self._off_commands[slot]:
            command = self._off_commands[slot]
            command.execute()
            self._history.append(command)
    
    def press_undo_button(self):
        if self._history:
            command = self._history.pop()
            command.undo()

# Usage
light = Light()
light_on = LightOnCommand(light)
light_off = LightOffCommand(light)

remote = RemoteControl()
remote.set_command(0, light_on, light_off)

remote.press_on_button(0)   # Light is ON
remote.press_off_button(0)  # Light is OFF
remote.press_undo_button()   # Light is ON (undoes off command)
```

### 4. Chain of Responsibility Pattern

Intent: Pass requests along a chain of handlers.
```python

from abc import ABC, abstractmethod

class Handler(ABC):
    def __init__(self):
        self._next_handler = None
    
    def set_next(self, handler):
        self._next_handler = handler
        return handler
    
    @abstractmethod
    def handle(self, request):
        if self._next_handler:
            return self._next_handler.handle(request)
        return None

class ConcreteHandler1(Handler):
    def handle(self, request):
        if request == "A":
            return f"Handler1: Handling request {request}"
        else:
            return super().handle(request)

class ConcreteHandler2(Handler):
    def handle(self, request):
        if request == "B":
            return f"Handler2: Handling request {request}"
        else:
            return super().handle(request)

class ConcreteHandler3(Handler):
    def handle(self, request):
        if request == "C":
            return f"Handler3: Handling request {request}"
        else:
            return super().handle(request)

# Using functions (Pythonic approach)
def create_chain(*handlers):
    for i in range(len(handlers) - 1):
        handlers[i].set_next(handlers[i + 1])
    return handlers[0]

# Usage
handler1 = ConcreteHandler1()
handler2 = ConcreteHandler2()
handler3 = ConcreteHandler3()

chain = create_chain(handler1, handler2, handler3)

for request in ["A", "B", "C", "D"]:
    result = chain.handle(request)
    if result:
        print(result)
    else:
        print(f"No handler for request: {request}")
```

### 5. State Pattern

Intent: Allow object to change behavior when its state changes.
```python

from abc import ABC, abstractmethod

class State(ABC):
    @abstractmethod
    def handle_request(self, context):
        pass

class Context:
    def __init__(self, state: State):
        self._state = state
    
    def set_state(self, state: State):
        print(f"Context: Transition to {type(state).__name__}")
        self._state = state
    
    def request(self):
        self._state.handle_request(self)

class ConcreteStateA(State):
    def handle_request(self, context):
        print("State A handling request, transitioning to State B")
        context.set_state(ConcreteStateB())

class ConcreteStateB(State):
    def handle_request(self, context):
        print("State B handling request, transitioning to State A")
        context.set_state(ConcreteStateA())

# More practical example
class VendingMachineState(ABC):
    @abstractmethod
    def insert_money(self, machine, amount: float):
        pass
    
    @abstractmethod
    def select_product(self, machine, product: str):
        pass
    
    @abstractmethod
    def dispense_product(self, machine):
        pass

class ReadyState(VendingMachineState):
    def insert_money(self, machine, amount: float):
        machine.balance += amount
        print(f"Inserted ${amount}. Total: ${machine.balance}")
        machine.set_state(HasMoneyState())
    
    def select_product(self, machine, product: str):
        print("Please insert money first")
    
    def dispense_product(self, machine):
        print("Please insert money and select product first")

class HasMoneyState(VendingMachineState):
    def insert_money(self, machine, amount: float):
        machine.balance += amount
        print(f"Inserted ${amount}. Total: ${machine.balance}")
    
    def select_product(self, machine, product: str):
        if product in machine.products:
            price = machine.products[product]
            if machine.balance >= price:
                machine.selected_product = product
                machine.set_state(DispensingState())
                machine.dispense_product()
            else:
                print(f"Insufficient funds. Need ${price - machine.balance} more")
        else:
            print("Product not available")
    
    def dispense_product(self, machine):
        print("Please select a product first")

class DispensingState(VendingMachineState):
    def insert_money(self, machine, amount: float):
        print("Please wait, dispensing product...")
    
    def select_product(self, machine, product: str):
        print("Please wait, dispensing product...")
    
    def dispense_product(self, machine):
        price = machine.products[machine.selected_product]
        machine.balance -= price
        print(f"Dispensing {machine.selected_product}")
        print(f"Change: ${machine.balance}")
        machine.balance = 0
        machine.selected_product = None
        machine.set_state(ReadyState())

class VendingMachine:
    def __init__(self):
        self.products = {"soda": 1.5, "chips": 1.0, "candy": 0.75}
        self.balance = 0
        self.selected_product = None
        self._state = ReadyState()
    
    def set_state(self, state: VendingMachineState):
        self._state = state
    
    def insert_money(self, amount: float):
        self._state.insert_money(self, amount)
    
    def select_product(self, product: str):
        self._state.select_product(self, product)
    
    def dispense_product(self):
        self._state.dispense_product(self)

# Usage
vm = VendingMachine()
vm.select_product("soda")        # Please insert money first
vm.insert_money(2.0)             # Inserted $2.0. Total: $2.0
vm.select_product("soda")        # Dispensing soda, Change: $0.5
```

## Saga Design Pattern - Python Implementation
Introduction

The Saga pattern manages distributed transactions across multiple services using a sequence of local transactions with compensating actions.
### 1. Choreography-Based Saga
```python

from abc import ABC, abstractmethod
from typing import Dict, Any, List
import uuid
from dataclasses import dataclass
from enum import Enum

class SagaStatus(Enum):
    PENDING = "pending"
    COMPENSATING = "compensating"
    COMPLETED = "completed"
    FAILED = "failed"

@dataclass
class SagaEvent:
    event_id: str
    saga_id: str
    event_type: str
    data: Dict[str, Any]
    timestamp: str

class SagaStep(ABC):
    @abstractmethod
    def execute(self, saga_data: Dict[str, Any]) -> bool:
        pass
    
    @abstractmethod
    def compensate(self, saga_data: Dict[str, Any]) -> bool:
        pass

class OrderSaga:
    def __init__(self):
        self.saga_id = str(uuid.uuid4())
        self.status = SagaStatus.PENDING
        self.steps: List[SagaStep] = []
        self.data: Dict[str, Any] = {}
        self.current_step = 0
    
    def add_step(self, step: SagaStep):
        self.steps.append(step)
    
    def execute(self) -> bool:
        try:
            for i, step in enumerate(self.steps):
                self.current_step = i
                if not step.execute(self.data):
                    self.compensate()
                    return False
            self.status = SagaStatus.COMPLETED
            return True
        except Exception as e:
            print(f"Saga execution failed: {e}")
            self.compensate()
            return False
    
    def compensate(self):
        self.status = SagaStatus.COMPENSATING
        for i in range(self.current_step, -1, -1):
            try:
                self.steps[i].compensate(self.data)
            except Exception as e:
                print(f"Compensation failed at step {i}: {e}")
        self.status = SagaStatus.FAILED

# Concrete Steps
class InventoryStep(SagaStep):
    def execute(self, saga_data: Dict[str, Any]) -> bool:
        order_id = saga_data.get('order_id')
        items = saga_data.get('items', [])
        print(f"Reserving inventory for order {order_id}: {items}")
        # Simulate inventory reservation
        saga_data['inventory_reserved'] = True
        return True
    
    def compensate(self, saga_data: Dict[str, Any]) -> bool:
        order_id = saga_data.get('order_id')
        print(f"Releasing inventory for order {order_id}")
        # Simulate inventory release
        return True

class PaymentStep(SagaStep):
    def execute(self, saga_data: Dict[str, Any]) -> bool:
        order_id = saga_data.get('order_id')
        amount = saga_data.get('amount', 0)
        print(f"Processing payment for order {order_id}: ${amount}")
        # Simulate payment processing
        if amount > 1000:  # Simulate payment failure for large amounts
            print("Payment failed: Amount too large")
            return False
        saga_data['payment_processed'] = True
        return True
    
    def compensate(self, saga_data: Dict[str, Any]) -> bool:
        order_id = saga_data.get('order_id')
        print(f"Refunding payment for order {order_id}")
        # Simulate payment refund
        return True

class ShippingStep(SagaStep):
    def execute(self, saga_data: Dict[str, Any]) -> bool:
        order_id = saga_data.get('order_id')
        address = saga_data.get('shipping_address')
        print(f"Scheduling shipping for order {order_id} to {address}")
        # Simulate shipping scheduling
        saga_data['shipping_scheduled'] = True
        return True
    
    def compensate(self, saga_data: Dict[str, Any]) -> bool:
        order_id = saga_data.get('order_id')
        print(f"Cancelling shipping for order {order_id}")
        # Simulate shipping cancellation
        return True

# Usage
def create_order_saga(order_data: Dict[str, Any]) -> OrderSaga:
    saga = OrderSaga()
    saga.data = order_data
    
    # Add steps in sequence
    saga.add_step(InventoryStep())
    saga.add_step(PaymentStep())
    saga.add_step(ShippingStep())
    
    return saga

# Test successful order
successful_order = {
    'order_id': 'order_123',
    'items': ['item1', 'item2'],
    'amount': 100,
    'shipping_address': '123 Main St'
}

saga = create_order_saga(successful_order)
result = saga.execute()
print(f"Saga completed successfully: {result}")

# Test failed order (payment failure)
failed_order = {
    'order_id': 'order_124',
    'items': ['item1', 'item2'],
    'amount': 1500,  # This will cause payment to fail
    'shipping_address': '123 Main St'
}

saga2 = create_order_saga(failed_order)
result2 = saga2.execute()
print(f"Saga completed successfully: {result2}")
```

### 2. Orchestration-Based Saga
```python

import asyncio
from typing import Dict, Any, Callable, List
import uuid

class SagaOrchestrator:
    def __init__(self):
        self.sagas: Dict[str, Dict] = {}
    
    async def execute_saga(self, 
                          steps: List[Callable], 
                          compensations: List[Callable],
                          data: Dict[str, Any]) -> bool:
        saga_id = str(uuid.uuid4())
        self.sagas[saga_id] = {
            'status': 'running',
            'data': data,
            'current_step': 0
        }
        
        executed_steps = []
        
        try:
            for i, step in enumerate(steps):
                self.sagas[saga_id]['current_step'] = i
                print(f"Executing step {i}")
                
                # Execute step
                success = await step(data)
                executed_steps.append(i)
                
                if not success:
                    raise Exception(f"Step {i} failed")
            
            self.sagas[saga_id]['status'] = 'completed'
            return True
            
        except Exception as e:
            print(f"Saga failed at step {i}, compensating...")
            self.sagas[saga_id]['status'] = 'compensating'
            
            # Execute compensations in reverse order
            for step_index in reversed(executed_steps):
                try:
                    await compensations[step_index](data)
                except Exception as comp_error:
                    print(f"Compensation failed for step {step_index}: {comp_error}")
            
            self.sagas[saga_id]['status'] = 'failed'
            return False

# Example usage with async functions
async def reserve_inventory(data: Dict[str, Any]) -> bool:
    await asyncio.sleep(0.1)  # Simulate async operation
    print(f"Reserving inventory for order {data['order_id']}")
    return True

async def compensate_inventory(data: Dict[str, Any]) -> bool:
    await asyncio.sleep(0.1)
    print(f"Releasing inventory for order {data['order_id']}")
    return True

async def process_payment(data: Dict[str, Any]) -> bool:
    await asyncio.sleep(0.1)
    amount = data.get('amount', 0)
    print(f"Processing payment: ${amount}")
    
    if amount > 1000:  # Simulate failure condition
        return False
    return True

async def compensate_payment(data: Dict[str, Any]) -> bool:
    await asyncio.sleep(0.1)
    print(f"Refunding payment for order {data['order_id']}")
    return True

async def schedule_shipping(data: Dict[str, Any]) -> bool:
    await asyncio.sleep(0.1)
    print(f"Scheduling shipping to {data['shipping_address']}")
    return True

async def compensate_shipping(data: Dict[str, Any]) -> bool:
    await asyncio.sleep(0.1)
    print(f"Cancelling shipping for order {data['order_id']}")
    return True

# Usage
async def main():
    orchestrator = SagaOrchestrator()
    
    # Define saga steps and compensations
    steps = [reserve_inventory, process_payment, schedule_shipping]
    compensations = [compensate_inventory, compensate_payment, compensate_shipping]
    
    order_data = {
        'order_id': 'order_async_123',
        'amount': 500,
        'shipping_address': '456 Async Ave'
    }
    
    # Execute successful saga
    result = await orchestrator.execute_saga(steps, compensations, order_data)
    print(f"Async saga result: {result}")
    
    # Execute failing saga
    failing_order_data = {
        'order_id': 'order_async_124', 
        'amount': 1500,  # Will cause payment to fail
        'shipping_address': '456 Async Ave'
    }
    
    result2 = await orchestrator.execute_saga(steps, compensations, failing_order_data)
    print(f"Failing async saga result: {result2}")

# Run the async example
# asyncio.run(main())
```

### 3. Saga Pattern with Persistence
```python

import json
import time
from typing import Dict, Any, List
from abc import ABC, abstractmethod

class SagaRepository(ABC):
    @abstractmethod
    def save_saga(self, saga_id: str, data: Dict[str, Any]):
        pass
    
    @abstractmethod
    def load_saga(self, saga_id: str) -> Dict[str, Any]:
        pass
    
    @abstractmethod
    def update_saga_status(self, saga_id: str, status: str, step: int):
        pass

class FileSagaRepository(SagaRepository):
    def __init__(self, file_path: str):
        self.file_path = file_path
    
    def save_saga(self, saga_id: str, data: Dict[str, Any]):
        try:
            with open(self.file_path, 'r') as f:
                sagas = json.load(f)
        except (FileNotFoundError, json.JSONDecodeError):
            sagas = {}
        
        sagas[saga_id] = {
            'data': data,
            'status': 'pending',
            'current_step': 0,
            'created_at': time.time()
        }
        
        with open(self.file_path, 'w') as f:
            json.dump(sagas, f, indent=2)
    
    def load_saga(self, saga_id: str) -> Dict[str, Any]:
        try:
            with open(self.file_path, 'r') as f:
                sagas = json.load(f)
            return sagas.get(saga_id, {})
        except (FileNotFoundError, json.JSONDecodeError):
            return {}
    
    def update_saga_status(self, saga_id: str, status: str, step: int):
        try:
            with open(self.file_path, 'r') as f:
                sagas = json.load(f)
        except (FileNotFoundError, json.JSONDecodeError):
            return
        
        if saga_id in sagas:
            sagas[saga_id]['status'] = status
            sagas[saga_id]['current_step'] = step
            sagas[saga_id]['updated_at'] = time.time()
            
            with open(self.file_path, 'w') as f:
                json.dump(sagas, f, indent=2)

class PersistentSaga:
    def __init__(self, repository: SagaRepository):
        self.repository = repository
        self.saga_id = None
        self.data = {}
    
    def start_saga(self, saga_id: str, initial_data: Dict[str, Any]):
        self.saga_id = saga_id
        self.data = initial_data
        self.repository.save_saga(saga_id, initial_data)
    
    def execute_step(self, step_func, compensation_func, step_name: str) -> bool:
        if not self.saga_id:
            raise ValueError("Saga not started")
        
        saga_data = self.repository.load_saga(self.saga_id)
        current_step = saga_data.get('current_step', 0)
        
        try:
            print(f"Executing step: {step_name}")
            success = step_func(self.data)
            
            if success:
                self.repository.update_saga_status(self.saga_id, 'in_progress', current_step + 1)
                return True
            else:
                raise Exception(f"Step {step_name} failed")
                
        except Exception as e:
            print(f"Step {step_name} failed: {e}")
            # Execute compensation
            compensation_func(self.data)
            self.repository.update_saga_status(self.saga_id, 'failed', current_step)
            return False
    
    def complete_saga(self):
        if self.saga_id:
            self.repository.update_saga_status(self.saga_id, 'completed', -1)

# Usage example with persistence
def demo_persistent_saga():
    repository = FileSagaRepository('sagas.json')
    saga = PersistentSaga(repository)
    
    # Start a new saga
    order_data = {
        'order_id': 'order_persist_123',
        'items': ['item1', 'item2'],
        'amount': 200,
        'shipping_address': '789 Persist St'
    }
    
    saga.start_saga('saga_persist_123', order_data)
    
    # Define step functions
    def reserve_inventory(data):
        print("Reserving inventory...")
        return True
    
    def compensate_inventory(data):
        print("Compensating inventory...")
    
    def process_payment(data):
        print("Processing payment...")
        return True
    
    def compensate_payment(data):
        print("Compensating payment...")
    
    # Execute steps
    if saga.execute_step(reserve_inventory, compensate_inventory, "inventory_reservation"):
        if saga.execute_step(process_payment, compensate_payment, "payment_processing"):
            saga.complete_saga()
            print("Saga completed successfully")
        else:
            print("Saga failed during payment processing")
    else:
        print("Saga failed during inventory reservation")

# demo_persistent_saga()
```
When to Use Which Pattern
Pattern Selection Guide
Problem	Recommended Python Patterns
Object creation complexity	Factory Method, Builder
Adding functionality dynamically	Decorator (use Python decorators)
Managing state-dependent behavior	State Pattern
One-to-many dependencies	Observer Pattern
Distributed transactions	Saga Pattern
Algorithm selection	Strategy Pattern (use functions)
Request handling chain	Chain of Responsibility
Complex object structure	Composite Pattern
Python-Specific Considerations

    Use Functions as Strategies: In Python, you can often use functions instead of strategy classes

    Leverage Decorators: Python's decorator syntax makes the decorator pattern very natural

    Use Context Managers: For resource management patterns

    Duck Typing: Reduces need for complex interface hierarchies

    First-Class Functions: Enables simpler implementations of many behavioral patterns

Best Practices for Python Design Patterns

    Keep It Simple: Don't over-engineer; use patterns only when they add value

    Leverage Python Features: Use language features before reaching for complex patterns

    Use ABC for Clear Interfaces: abc.ABC and @abstractmethod for defining interfaces

    Favor Composition: Use dependency injection and composition over inheritance

    Write Pythonic Code: Follow PEP 8 and use Python idioms

This comprehensive Python implementation covers all major design patterns with practical, runnable examples that demonstrate Python-specific best practices and idioms.