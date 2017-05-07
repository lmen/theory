# Notes from OOP [https://www.youtube.com/watch?v=TMuno5RZNeE] 

## Class design principles

* SRP : the single responsability principle
* OCP : the open /close principle
* LSP : the liskov substitution principle
* ISP : the interface segregation principle
* DIP : The dependency inversion principle

## SRP -  The Single responsability principle

> A class should have one and only one, reason to change.

Don't change functions that could change for diferent reasons on the same class. Because a change in one function might break other class functions not related with the change.

A class should only have one concern.

## OCP - The open / close principle

> Modules should be open for extension but closed for modification.

it should be possible to change the beahviour of a module (class) without modifying the module code.

In java this principle could be follow by implementing the algoritms using interfaces and polymorfirm. But one thing is true, it is not possible to predict the future user requires. The ideal is to talk with the user before starting the implementation in order to understand what the algoritm should do.

the algorithms of a class should be adaptable from the outside world input without its code being changed.

## LSP - Liskov substitution principle
 > Derived classes must be usable through the base class interface, without the need for thr user to know the difference.

 Class hierachy design related principle.

## DIP - Dependency invertion principle

>  + High level modules should not depend upon low level modules. Both should depend upon abstractions 

In a programing language there is an hiearchy :

The main module should only use interfaces to comunicate with the lower level modules. The main module should not create "instances" of interface, but should be given to him (as function parameters for example) the instances. 

Composition root : place where the detail implmentations are created. Ususaly at the begining of the program.

> + Abstractions should not depend upon details. Details should depend upon abstractions

THis is related to the way, as abstracts (interfaces should be design). because this interface should have multiple implementation,each one with specific implementation detail, the abstraction (interface) definition should not have any specific element related with a specific implementation. It should be abstractions.  
