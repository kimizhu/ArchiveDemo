# Lambda Expressions vs. Anonymous Methods, Part Three

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/12/2007 12:45:00 PM

-----

[Last time](http://blogs.msdn.com/ericlippert/archive/2007/01/11/lambda-expressions-vs-anonymous-methods-part-two.aspx) I said that I would describe a sneaky trick whereby you can get variable type inference out of a lambda by using method type inference.

First off, I want to again emphasize that the reason we are adding so many type inferencing features to C\# 3.0 is not just because it is convenient to reduce the redundancy currently required. That's a nice-to-have feature, but certainly not a must-have feature. No, the reason why we are adding so many type inferencing features is because we are adding *anonymous types*. (We are adding anonymous types because they make writing queries so much more pleasant.) Since an anonymous type is, by definition, nameless, you need to be able to infer the type of anything which would otherwise have to be declared with a type name.

But if the type of a lambda expression comes from its target type, and the target type has to be named in the declaration, how can you create a lambda expression which, say, returns an anonymous type?

var f = (Customer c)=\>new {c.Name, c.Age, c.Address};

We have no way of saying in any type declaration what the type of the return is.  The best we can do is the incredibly weak

Func\<Customer, object\> f = (Customer c)=\>new {c.Name, c.Age, c.Address};

Yuck, all the type information about the tuple has been lost. We can do better than this.

The trick here is that we have extended the type inferencing algorithm on methods so that **generic method type variables can be inferred from the return types of lambdas**. Suppose we have this little helper identity function:

Func\<A, R\> MakeFunction\<A, R\>(Func\<A, R\> f) { return f; }

Now look what happens when we say

var f = MakeFunction((Customer c)=\>new {c.Name, c.Age, c.Address});

The method type inferencing engine says ok, we have an actual argument which is a lambda from Customer to some anonymous tuple type. We have a formal parameter which is a delegate from A to R. Therefore A is Customer, R is the anonymous tuple type. Now we know the return type of this generic function, and therefore abracadabra, the right hand side of the declaration has a type, so it is legal\! (And it's not even particularly unperformant, since the jitter will likely optimize away the identity function. Even if it doesn't, it's tiny compared to the cost of creating the delegate object.)

What if the lambda has an anonymous type for its parameter? Suppose we want a function from the anonymous tuple type above which returns a string. This is a little kludgier but we can still do it:

Func\<A, R\> MakeFunction\<A, R\>(Func\<A, R\> f, A a) { return f; }

var f = MakeFunction(c=\>c.Name, new {Name="", Age=0, Address=""} );

Now the type inference engine can't infer anything from the lambda parameter, since it is untyped. But it can infer the parameter type from the second actual argument. Once it knows the type of A, it can infer R from the return type of the lambda.

Pretty neat, eh?

