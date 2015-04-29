---
layout: post
title: Learning JavaScript
---

# Eloquent JavaScript

* Chapter 6: The secret life of objects

- When you add a property to an object, whether it is present in the prototype or not, the property is added to the object itself, which will hanceforth have it as its own property. If there is a property by the same name in the prototype, this property will no longer affect the object. The prototype itself is not changed.

- All properties that we create by assigning to them are enumerable. The standard properties in Object.prototype are all nonenumerable, which is why they do not show up in such a for/in loop.

- It is possible to define our own nonenumerable properties by using the Object.defineProperty function, which allow us to control the type of property we are creating.
