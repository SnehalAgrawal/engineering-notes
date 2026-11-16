# Dependency Injection (DI)

### 1. Overview

With DI, an object gets the things it needs (its dependencies) from outside, instead of creating them itself. It is the most common way to follow the Dependency Inversion principle. Code becomes easier to test and to rewire.

Analogy: without DI, a phone has its charger built in. With DI, you plug in any compatible charger.

Without DI, `Car` decides which engine to build:

```javascript
class Car {
  constructor() {
    this.engine = new Engine(); // tightly coupled
  }
}
```

With DI, the engine is passed in, so any engine works:

```javascript
class Car {
  constructor(engine) {
    this.engine = engine;
  }
  drive() { this.engine.start(); }
}

new Car(new ElectricEngine()).drive();
```

Frameworks can do the wiring for you with a container. Plain JavaScript has no built-in container, so wiring is done by hand at startup.

```python
# FastAPI: the framework calls get_engine() and passes the result in
from fastapi import Depends, FastAPI

app = FastAPI()

def get_engine():
    return Engine()

@app.get("/drive")
def drive(engine: Engine = Depends(get_engine)):
    engine.start()
    return {"status": "driving"}
```

In NestJS (TypeScript), a class marked `@Injectable()` and registered in a module is passed into constructors automatically.

### 2. Key Concepts

* **Dependency**: an object another object needs, like a database client.
* **Constructor injection**: pass dependencies to the constructor. Best for required ones, since the object can't exist without them.
* **Setter injection**: set dependencies after creation. Good for optional ones.
* **Field injection**: the framework writes straight into fields (like `@Autowired` on a field in Spring). Discouraged: dependencies are hidden and the class is hard to build in tests.
* **IoC container**: the framework part that creates dependencies and manages how long each one lives.
* **Benefits**: loose coupling, easy mocking in tests, and swapping implementations per environment without editing the class.

### 3. Real-World Usage

* **Unit tests**: inject a `MockPaymentGateway` into `OrderService`, so tests make no real HTTP calls.
* **Per-environment setup**: `S3Storage` in production, `LocalStorage` in development.
* **Frameworks**: Spring and Guice (Java), the built-in .NET container, Angular and NestJS all build on DI.

### 4. Tradeoffs

* **Setup**: you need one place where everything is wired together (the composition root).
* **Indirection**: you may have to read the container config to see which class is actually used.
* **Lifetimes**: singleton, scoped, and transient cause subtle bugs if misunderstood.

### 5. When NOT to Use

* **Small libraries**: don't make users install a DI container. Plain constructor parameters are enough.
* **Value objects**: `Money`, `DateRange`, or a DTO just hold data. Create them directly.

### 6. Interview Focus

* **IoC vs. DI**:
  * IoC (Inversion of Control) is the broad idea: a framework controls creation or flow and calls your code.
  * DI is one way to do IoC: dependencies come from outside. Callbacks, event handlers, and Template Method are other forms.
* **Lifetimes in a web app**:
  * Singleton: one instance for the whole app. Fits a connection pool or a logger.
  * Scoped: one instance per request. Fits request context or a DB session.
  * Transient: a new instance every time.
  * Classic bug: a singleton that holds a scoped dependency keeps the first request's instance forever (a "captive dependency").
* **Circular dependency (A needs B, B needs A)**: constructor injection can't build either one, so the container fails at startup. Usually a design smell. Fixes, best first:
  * Move the shared logic into a new class C that both use (A → C ← B).
  * Depend on an interface or an event instead of the other class.
  * Lazy injection: pass a getter, like `constructor(getB)`, and call it only when B is needed.
  * Setter injection: build both, then connect them. It works but hides the cycle.

### 7. Common Mistakes

* **Service Locator**: passing the container into a class so it fetches its own dependencies. It hides what the class really needs.
* **Too many constructor arguments**: a sign the class does too much (breaks SRP).
* **Hand-wiring a large app**: without a container, the startup file grows huge and fragile.
