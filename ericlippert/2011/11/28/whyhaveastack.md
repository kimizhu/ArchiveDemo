# Why have a stack?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/28/2011 10:39:03 AM

-----

[Last time](http://blogs.msdn.com/b/ericlippert/archive/2011/11/18/why-il.aspx) I discussed why it is that we have all the .NET compilers target an "intermediate language", or "IL", and then have jitters that translate IL to machine code: because doing so ultimately reduces the costs of building a multi-language, multi-hardware platform. Today I want to talk a bit about why IL is the way it is; specifically, why is it a "stack machine"?

To begin with, what is a "stack machine"? Let's consider how you might design a machine language that could describe the operation of adding together two integers to make a third. You could do it like this:

add \[address of first addend\], \[address of second addend\], \[address of sum\]  
  
  

When the machine encounters this instruction it looks up the values of the addends stored in the two addresses, somehow adds them together -- how it does so is its business -- and stores the result in the third address.

You might instead say that there is a special region of memory called the "accumulator" which knows how to add a given value to itself:

clear\_accumulator  
increase\_accumulator \[address of first addend\]  
increase\_accumulator \[address of second addend\]  
save\_accumulator \[address of sum\]  
  
  

Or, you could say that there is a special region of memory called the "stack" which can grow and shrink; you get access to the items on the top:

push\_value\_at \[address of first addend\]  
push\_value\_at \[address of second addend\]  
add  
pop\_value\_into \[address of sum\]  
  
  

The "add" instruction takes the two values off the top of the stack, somehow adds them, and then puts the result back on the stack; the net result is that the stack shrinks by one.

A virtual machine where most of the instructions are of this third form is called a "stack machine", for obvious reasons.

IL specifies a stack machine, just like many other virtual machines. But most hardware instruction sets actually more closely resemble the second form: registers are just fancy accumulators. Why then are so many virtual machines specifying stack machines?

There are several reasons, but again, it primarily comes down to lowering costs. Stack machines are very easy to understand, they are very easy to write a compiler front-end for, they are easy to build interpreters for, they are easy to build jitters for, and they provide a concise abstraction for the common case. The common case is that the result of any particular operation is only going to be of interest for a brief period.

Imagine, for example, if we chose the first strategy for IL, and then had to compile an expression like x = A() + B() + C(); What would we have to do in the first case? Something like this:

create\_temporary\_storage // for result of A()  
call A(), \[address of temporary storage\]  
create\_temporary\_storage // for result of B()  
call B(), \[address of temporary storage\]  
create\_temporary\_storage // for result of first addition  
add \[address of first temporary storage\], \[address of second temporary storage\], \[address of third temporary storage\]  
...  
  
  

You see how this goes? The IL is getting huge, and all so that we can keep track of precisely which memory locations are used to store values that we are about to never care about again. A stack abstraction lets the stack implementation deal with the temporary storages; in a stack machine, the same code looks something like:

push \[address of x\]  
call A() // implicitly creates a temporary storage by pushing the result on the stack  
call B()  
add  
call C()  
add  
store // store result on top of stack in address just below it.  
  
  

The code is much smaller and much easier to understand. A stack machine is a very simple way to describe a complex computation; by being able to write code for such a simple machine, it lowers the cost of making a compiler. And not only is it easier to write compilers and jitters that target simple stack machines, it is easier to write other code analysis tools as well. The IL verifier, for example, can quickly determine when there is a code path through a method that, say, misaligns the stack, or passes the wrong types of arguments to a method.

