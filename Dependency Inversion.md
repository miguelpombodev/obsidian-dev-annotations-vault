Just like we know that every layer must only reference its inner layers, for example, entities don't know that Use Cases exist, and also Use Cases don't know that Controllers exist.

When an inner layer needs something from a external one, this inner layer defines a interface and the external layer implements this interface.
This action is a [[SOLID]] principle called [[Dependency Inversion]].
![[Pasted image 20260528121707.png]]

Dependency Inversion Principle