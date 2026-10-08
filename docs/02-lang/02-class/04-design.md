---
title: 2.2.4. Reflection on Design
---

In statically typed languages, types determine data storage structures upon which operations depend (specifically offsets within the structure), making types the fundamental design unit. In contrast, in completely dynamically typed languages like JavaScript, types belong to values, meaning that data storage structures and valid operations are all determined by values. Consequently, it is unnecessary to design types for functions to operate on proper data, as functions operate on any data with proper values, which makes functions the basic design unit. Additionally, only global functions or methods of global objects are reusable at [the program level](/lang/function/intro).

Note that the paradigm is determined by the type system rather than whether a language is compiled or interpreted. Static typing implies a type-based (e.g. class-based) paradigm, whereas value-based typing implies a data + process (e.g. global objects + functions) paradigm. In principle, any language can be compiled or interpreted. In practice, compiled languages are typically statically typed, or dynamically typed but not going so far to be value-based as interpreted languages typically do. Additionally, interpreted languages usually support the class-based paradigm in a runtime way as well, since classes remain valuable for multiple objects of one class, non-global objects to be reusable. Python, as another example of an interpreted language with value-based typing, even chooses the class-based paradigm as its primary style.

```javascript title="Classes for Reusable Multiple, Non-Global Objects"
// Global factory function needed to reuse multiple, non-global objects
function createConnection(uri) {
  // Ephemeral (non-global) object to be freed after use
  return {
    socket: null,

    connect() {
      this.socket = openSocket(uri);
    },

    send(s) {
      this.socket.send(s);
    },

    close() {
      this.socket.close();
    },
  };
}

// Class as a better factory
function Connection(uri) {
  this.uri = uri;
  this.socket = null;
}

Connection.prototype.connect = function () {
  this.socket = openSocket(this.uri);
};

Connection.prototype.send = function (s) {
  this.socket.send(s);
};

Connection.prototype.close = function () {
  this.socket.close();
};
```

In fact, modern languages are increasingly borrowing from and converging with each other. Compiled languages are becoming more dynamic, with metaprogramming, reflection, [decorators](/lang/typescript/decorator), and type inference, to reduce rigid type contracts. Interpreted languages, on the other hand, introduce compile-time type checking to reduce runtime errors (e.g. [TypeScript](/lang/typescript/intro) for JavaScript), and leverage Just-in-Time (JIT) or Ahead-of-Time (AOT) compilers for native execution performance. Overall, paradigms are becoming more of a design philosophy than language mechanics.
