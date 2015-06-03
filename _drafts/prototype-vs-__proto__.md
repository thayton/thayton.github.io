Use https://www.draw.io/ and create some diagrams showing the difference
between prototype and __proto__ in Javascript.

So few books use pictures to describe how __proto__ is used for inheritence
or the fact that creating a function results in *two* separate objects: one
for the function and one for a prototype object that is referenced (linked to)
by the function object.

Use the Employee/Manager/WorkerBee example provided here:

https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Details_of_the_Object_Model

First draw a linked list of objects showing how we can look up attributes given a 
chain of objects.

Then show how a method will bind `this` according to which object invokes a method.
IE, parent object defines a method but when derived object calls method `this` is
bound to derived object. When parent object calls emthod `this` is bound to parent
to object.

Show hasOwnProperty() in diagram meaning that we can find a property in an object
without traversing the object chain.