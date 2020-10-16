---
title: "Software Blog: OOP Basics"
tags: [oop]
style: fill
color: info
description: Basics of object-oriented programming.
---

Object-oriented programming (OOP) is programming paradigm based on based on wrapping data and behaviour into bundles (or objects) that enables our code to be more modular, flexible and reusable. Classes are the "blueprints" that define the structure(states and behaviours) of our objects and objects are concrete instances of our classes. 
There are 4 key aspects of OOP that differentiate it from other paradigms. I used the acronym IPEA (like IKEA with a P) to remember the four:

1. Abstraction
2. Encapsulation
3. Inheritance
4. Polymorphism

Abstraction is the act of modelling a real world object or phenomenon to a limited extent that is relevant to our context. Our context determines what to model. For instance, if you were modelling a person in a healthcare database, you would include fields such as height, weight and past medical history. On the other hand, if you were to model someone in an employee database you might opt include fields such as salary, position and peer reviews.

Encapsulation is the ability to expose a limited interface to the rest of a program. In essence, it is hiding parts of its state and behaviours from other objects and clients. As an example, a mobile robot API designer will encapsulate all the code that goes into making a robot move (e.g. starting up obscure hardware modules, sending PWM signals to motors) and only expose a limited interface (e.g. move left, right forward or backward) to its client because moving the robot is the only thing of interest to the client.

Inheritance is the ability for new classes to build on top of existing ones to help us maximize code reuse. You might have an Animal parent class (superclass, with Dog and Cat children classes (subclass). Your subclasses would then inherit the behaviours (e.g. eat() and sleep() ) and attributes (e.g. weight and blood_type )of the Animal class.

Polymorphism is the ability of an object to take on multiple forms. More concretely, it is the ability of a program to call the underlying implementation of a program without knowing its true form. We may have a pointer to an Animal object which could be a Dog or a Cat . Regardless of dog/cat, we can call eat() and the program will help us determine what the object should eat (e.g. dog food vs. cat food) without us having to worry about the underlying animal type.

## Summary:
Abstraction = model real world object/phenomenon to a limited extent
Encapsulation = exposing a limited interface to the rest of the program
Inheritance = ability for new classes to build on existing ones
Polymorphoism = ability for a base object to take on different forms
