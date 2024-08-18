---
title: OCP - Open/Closed Principle
tags:
  - dev
  - best-practise
  - patterns
  - ocp
  - solid
---

The Open-Closed Principle, represented by the "O" in SOLID, posits that software entities (classes, modules, functions, etc.) should be open for extension but closed for modification. In simpler terms, you should be able to add new functionality to existing code without altering its source.

## Problem, sollution without OCP

We have an Order class that handles various types of orders, including international orders. Here's the initial implementation:
```ts
class Order {
  constructor(private items: CartItem[]) {}

  calculateTotal() {
    return this.items.reduce((total, item) => total + item.price, 0);
  }
}

class CartItem {
  constructor(public name: string, public price: number) {}
}
```

Now, imagine your application grows, and you need to introduce a discount system. Without adhering to the OCP, you might be tempted to modify the Order class:
```ts
class Order {
  constructor(private items: CartItem[], private discount: number = 0) {}

  calculateTotal() {
    const subtotal = this.items.reduce((total, item) => total + item.price, 0);
    return subtotal - subtotal * (this.discount / 100);
  }
}
```
By adding the discount property directly to the Order class, you've modified it and `violated the OCP.`

## Applying the Open-Closed Principle
To follow the OCP, you should extend your code rather than modify it. Let's create a new class, DiscountedOrder, which extends the Order class:
```ts
class DiscountedOrder extends Order {
  constructor(items: CartItem[], private discount: number) {
    super(items);
  }

  applyDiscount() {
    return this.calculateTotal() * (1 - this.discount / 100);
  }
}
```
Now, you have a DiscountedOrder class that inherits the behavior of the base Order class and includes the discount calculation. This adheres to the OCP.

## Benefits
- Stability: Existing, tested code remains untouched, reducing the risk of introducing bugs when adding new features.
- Scalability: You can easily expand your codebase with new functionality without affecting its core components.
- Maintainability: Code that adheres to the OCP is more straightforward to maintain and extend over time.
- Reusability: You can reuse existing classes and modules in different contexts, maximizing code reuse.
