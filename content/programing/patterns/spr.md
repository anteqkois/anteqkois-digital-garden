---
title: SPR - Single Responsibility Principle
tags:
  - dev
  - best-practise
  - patterns
  - spr
  - solid
---

# SPR - Single Responsibility Principle

The Single Responsibility Principle (SRP), the first of the five principles of SOLID, is a fundamental concept in object-oriented software development.

- Is focused on the concept of a class or module having a single responsibility. It encourages designing classes or modules that do one thing and do it well, promoting a separation of concerns.
- Focuses on the responsibilities of classes or modules. It suggests that a class should have only one reason to change, meaning it should encapsulate a single responsibility or concern.
- Is closely related to the use of abstraction and interfaces, as it encourages designing classes or modules with well-defined responsibilities and interactions.

- Identify responsibilities: Analyze your classes to identify different responsibilities. Each responsibility is a potential axis of change.
- Divide and conquer: If a class has more than one responsibility, consider dividing it into smaller classes, each with its own responsibility.
- Code reuse: Classes with a single responsibility are easier to reuse because you can use them in different contexts without bringing unnecessary functionalities.
- Testability: Classes focused on a single responsibility are easier to test because you do not have to deal with multiple functionalities during the testing of a specific functionality.