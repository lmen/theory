# Functional programming

Characteristics of a functional programing. 


There is no shared state between functions. Not having shared state there are no **side efects** while following the function evocations. In a simple manter, not having shared state it ensures that 

```
assertEquals(f(x), f(x));
```

**is always true no matter the value o x ** (since function f does not have shared state).

The _assignement_ statement in pure functional languages is very restrict.

# Object Oriented

Characteristics of an Object Oriented:

> Procedures (algoritms) + state (data)
> But procedures can be exposed (to the public) but the state is hidden (encapsulation). Which gives to **immutable objects.**