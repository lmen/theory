# The Test Driven Development Laws

TDD is a discipline (a way to work) that has this sugested rules:

+ You are not allowed to write any production code unless it is to make a failing unit test;
+ You are not allowed tm write any more of a unit test than is sufficient to fail; and compilation failures are failures;
+ You are not allowed to write any mode production code than uis suficient to pass the one failing unit test;

TDD descrives a development workflow, where you have two planes.
- the test code that invoques the production code.
- the production code that is tested by the test code.

Both planes establish a feedback loop.

The result of TDD become the technical documentation of code.

The tests shouldn't be done after the code is done.

Supporting libraries:

* Junit
+ hamcrest to specify the expressions of intent (writing matcher rules)

Object oriented sofware Enginnerin  book from 1990

Unit test organzation sugestion:

* _given_: will assemply the code that is going to be tested
* _when_ : sets the input values and invokes the testeing code
* _then_ : do the main expected assertions    
* _where_ : do other assertions

