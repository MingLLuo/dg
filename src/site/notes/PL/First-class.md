---
{"dg-publish":true,"permalink":"/pl/first-class/","created":"2024-06-25T14:32:50.154+08:00","updated":"2024-06-25T14:39:30.199+08:00"}
---

#PL #First_Class 
A first-class object (or first-class citizen) can be used as values or passed as arguments, or inputs, to yet more functions.

Supports all the operations generally available include:
1. **Being passed as an argument to a function.**
2. **Being returned from a function.**
3. **Being assigned to a variable.**
4. **Being created and destroyed dynamically.**
5. **Being stored in data structures.**

In many programming languages, functions are first-class objects. This means you can treat functions just like any other object. For instance, you can store them in variables, pass them as parameters to other functions, return them from functions, and store them in data structures like arrays or lists.

### Application

1. **Higher-order Functions:**
   - Functions that take other functions as arguments or return them as results. This is common in functional programming paradigms. For example, in JavaScript:
     ```javascript
     function greet(name) {
       return function(message) {
         console.log(`${message}, ${name}!`);
       }
     }

     const greetJohn = greet('John');
     greetJohn('Hello');  // Output: Hello, John!
     ```

2. **Callback Functions:**
   - Functions passed as arguments to other functions that are intended to be called at a later time. This is widely used in asynchronous programming. For example, in Node.js:
     ```javascript
     const fs = require('fs');

     fs.readFile('example.txt', 'utf8', function(err, data) {
       if (err) throw err;
       console.log(data);
     });
     ```

3. **Functional Programming:**
   - Languages like Haskell and Lisp heavily use first-class functions. This enables the use of functions like `map`, `reduce`, and `filter` which operate on collections of data.

4. **Event Handling:**
   - In GUI applications, event handlers are often first-class functions. For example, in a browser environment:
     ```javascript
     document.getElementById('myButton').addEventListener('click', function() {
       alert('Button was clicked!');
     });
     ```

5. **Decorators:**
   - In Python, decorators are functions that modify the behavior of other functions or methods. They are made possible by the fact that functions are first-class objects. For example:
     ```python
     def my_decorator(func):
         def wrapper():
             print("Something is happening before the function is called.")
             func()
             print("Something is happening after the function is called.")
         return wrapper

     @my_decorator
     def say_hello():
         print("Hello!")

     say_hello()
     ```

6. **Command Patterns:**
   - In object-oriented design, the command pattern can use functions or objects as first-class citizens to encapsulate a request as an object, thereby allowing for parameterization of clients with queues, requests, and operations.

7. **Lambda Expressions:**
   - Anonymous functions that can be used wherever function objects are required. For example, in Python:
     ```python
     numbers = [1, 2, 3, 4]
     squared_numbers = list(map(lambda x: x ** 2, numbers))
     print(squared_numbers)  # Output: [1, 4, 9, 16]
     ```