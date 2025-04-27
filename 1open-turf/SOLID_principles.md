# SOLID Principles

The SOLID principles are a set of design guidelines in object-oriented programming that help make code more maintainable, scalable, and flexible. Here’s an explanation of each principle with examples:

- 1. Single Responsibility Principle (SRP)

     --A class should have only one reason to change, meaning it should only have one job or responsibility.

- 2. Open/Closed Principle (OCP)

     --Classes should be open for extension but closed for modification. You should be able to add new functionality without changing existing code.

- 3. Liskov Substitution Principle (LSP)

     --Objects of a superclass should be replaceable with objects of a subclass without affecting the functionality of the program.

- 4. Interface Segregation Principle (ISP)

     --A client should not be forced to implement an interface it doesn’t use. Create smaller, more specific interfaces instead of one large, general-purpose interface.

## 1. Single Responsibility Principle (SRP)

A class should have only one reason to change, meaning it should only have one job or responsibility.

**Example:**
Consider a `Book` class that has methods for both book details and saving book information to a file. This violates SRP, as `Book` is doing more than one job.

```javascript
// Bad Example: Violates SRP
class Book {
  constructor(title, author) {
    this.title = title;
    this.author = author;
  }

  getDetails() {
    return \`\${this.title} by \${this.author}\`;
  }

  saveToFile() {
    // Code to save book data to file
  }
}

// Good Example: Applying SRP
class Book {
  constructor(title, author) {
    this.title = title;
    this.author = author;
  }

  getDetails() {
    return \`\${this.title} by \${this.author}\`;
  }
}

class BookFileSaver {
  saveToFile(book) {
    // Code to save book data to file
  }
}
```

## 2. Open/Closed Principle (OCP)

Classes should be open for extension but closed for modification. You should be able to add new functionality without changing existing code.

**Example:**
Suppose you have a `Discount` class that applies a percentage discount. To add new discount types, you can use inheritance instead of modifying the `Discount` class.

```javascript
// Bad Example: Violates OCP
class Discount {
  applyDiscount(price, type) {
    if (type === 'percentage') {
      return price * 0.9;
    } else if (type === 'fixed') {
      return price - 20;
    }
  }
}

// Good Example: Applying OCP
class Discount {
  apply(price) {
    return price;
  }
}

class PercentageDiscount extends Discount {
  apply(price) {
    return price * 0.9;
  }
}

class FixedDiscount extends Discount {
  apply(price) {
    return price - 20;
  }
}
```

## 3. Liskov Substitution Principle (LSP)

Objects of a superclass should be replaceable with objects of a subclass without affecting the functionality of the program.

**Example:**
If a `Rectangle` class is extended by a `Square` class, the behavior should remain consistent.

```javascript
// Bad Example: Violates LSP
class Rectangle {
  constructor(width, height) {
    this.width = width;
    this.height = height;
  }

  setWidth(width) {
    this.width = width;
  }

  setHeight(height) {
    this.height = height;
  }

  getArea() {
    return this.width * this.height;
  }
}

class Square extends Rectangle {
  setWidth(width) {
    this.width = width;
    this.height = width;
  }

  setHeight(height) {
    this.width = height;
    this.height = height;
  }
}

// Good Example: Applying LSP
class Shape {
  getArea() {
    throw new Error('Method not implemented');
  }
}

class Rectangle extends Shape {
  constructor(width, height) {
    super();
    this.width = width;
    this.height = height;
  }

  getArea() {
    return this.width * this.height;
  }
}

class Square extends Shape {
  constructor(side) {
    super();
    this.side = side;
  }

  getArea() {
    return this.side * this.side;
  }
}
```

## 4. Interface Segregation Principle (ISP)

A client should not be forced to implement an interface it doesn’t use. Create smaller, more specific interfaces instead of one large, general-purpose interface.

**Example:**
Imagine an interface for a printer. We can separate functionalities for different printers.

```javascript
// Bad Example: Violates ISP
class Printer {
  print() {}
  scan() {}
  fax() {}
}

class BasicPrinter extends Printer {
  print() {
    // print functionality
  }
  scan() {
    throw new Error('Scan not supported');
  }
  fax() {
    throw new Error('Fax not supported');
  }
}

// Good Example: Applying ISP
class Printer {
  print() {}
}

class Scanner {
  scan() {}
}

class Fax {
  fax() {}
}

class BasicPrinter extends Printer {
  print() {
    // print functionality
  }
}
```

## 5. Dependency Inversion Principle (DIP)

High-level modules should not depend on low-level modules; both should depend on abstractions. Also, abstractions should not depend on details, details should depend on abstractions.

**Example:**
Suppose we have a `LightSwitch` class that depends on a `LightBulb`. We can make both depend on an abstraction.

```javascript
// Bad Example: Violates DIP
class LightBulb {
  turnOn() {
    console.log('LightBulb turned on');
  }

  turnOff() {
    console.log('LightBulb turned off');
  }
}

class LightSwitch {
  constructor(bulb) {
    this.bulb = bulb;
  }

  operate() {
    this.bulb.turnOn();
    this.bulb.turnOff();
  }
}

// Good Example: Applying DIP
class Switchable {
  turnOn() {}
  turnOff() {}
}

class LightBulb extends Switchable {
  turnOn() {
    console.log('LightBulb turned on');
  }

  turnOff() {
    console.log('LightBulb turned off');
  }
}

class LightSwitch {
  constructor(device) {
    this.device = device;
  }

  operate() {
    this.device.turnOn();
    this.device.turnOff();
  }
}
```

## Summary

- **SRP**: One responsibility per class.
- **OCP**: Open for extension, closed for modification.
- **LSP**: Subtypes should be substitutable for their base types.
- **ISP**: Clients should not depend on unused methods.
- **DIP**: High-level modules should not depend on low-level modules directly.
