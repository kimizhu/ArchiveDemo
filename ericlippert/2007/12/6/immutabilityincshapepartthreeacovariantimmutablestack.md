# Immutability in C\# Part Three: A Covariant Immutable Stack

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 12/6/2007 10:55:00 AM

-----

Now suppose we had a hypothetical future version of C\# in which interface covariance worked, and we wanted a covariant immutable stack. That is, we want to be able to implicitly convert an IStack\<Giraffe\> to IStack\<Mammal\>. As we've already discussed, this doesn't make much sense in an array, even though doing so is legal in C\# today. If you cast a Giraffe\[\] to Mammal\[\] then you can try to put a Tiger into the Mammal\[\] and it will fail at run time. It seems like the same should be true of stacks -- if you cast an IStack\<Giraffe\> to IStack\<Mammal\> then you can push a Tiger onto the stack of Giraffes, which should fail, right?

Maybe. Maybe not.

The interface we defined the other day is not suitable for covariance because it uses the variant parameter as both an input and and output. Let's get a bit crazy for a minute here -- suppose we got rid of the input on the Push:

 

    public interface IStack\<+T\> : IEnumerable\<T\>  
    {  
        IStack\<T\> Pop();  
        T Peek();  
        bool IsEmpty { get; }  
    }

This interface is now suitable for covariance. If you have a stack of Giraffes and you want to treat it as a stack of Mammals, you can do so with perfect type safety, since we know that we will never be pushing a Tiger onto that thing. We'll only be reading off Giraffes, all of which are Mammals. This may seem less than useful, but we'll see what we can do:

 

    public sealed class Stack\<T\> : IStack\<T\>  
    {  
        private sealed class EmptyStack : IStack\<T\>  
        {  
            public bool IsEmpty { get { return true; } }  
            public T Peek() { throw new Exception("Empty stack"); }  
            public IStack\<T\> Pop() { throw new Exception("Empty stack"); }  
            public IEnumerator\<T\> GetEnumerator() { yield break; }  
            IEnumerator IEnumerable.GetEnumerator() { return this.GetEnumerator(); }  
        }  
        private static readonly EmptyStack empty = new EmptyStack();  
        public static IStack\<T\> Empty { get { return empty; } }  
        private readonly T head;  
        private readonly IStack\<T\> tail;  
        private Stack(T head, IStack\<T\> tail)  
        {  
            this.head = head;  
            this.tail = tail;  
        }  
        public bool IsEmpty { get { return false; } }  
        public T Peek() { return head; }  
        public IStack\<T\> Pop() { return tail; }  
        public static IStack\<T\> Push(T head, IStack\<T\> tail) { return new Stack\<T\>(head, tail); }  
        public IEnumerator\<T\> GetEnumerator()  
        {  
            for(IStack\<T\> stack = this; \!stack.IsEmpty ; stack = stack.Pop())  
                yield return stack.Peek();  
        }  
        IEnumerator IEnumerable.GetEnumerator() {return this.GetEnumerator();}  
    }

Notice that Push has disappeared from the empty stack and is static on the nonempty stack, and hey, check it out:

 

IStack\<Giraffe\> s1 = Stack\<Giraffe\>.Empty;  
IStack\<Giraffe\> s2 = Stack\<Giraffe\>.Push(new Giraffe("Gerry"), s1);  
IStack\<Giraffe\> s3 = Stack\<Giraffe\>.Push(new Giraffe("Geoffrey"), s2);  
IStack\<Mammal\> s4 = Stack\<Mammal\>.Push(new Tiger("Tony"), s3);

Oh my goodness, we just pushed a Tiger onto a stack of Mammals that is actually a stack of Giraffes underneath, but everything is still typesafe\! There is nothing we can do here to cause an exception at runtime in the type system. It all just works. All the stacks of Giraffes continue to be stacks of Giraffes; they're immutable, so they could hardly be anything else.

This code is pretty ugly though. So...

 

    public static class Extensions  
    {  
      public static IStack\<T\> Push\<T\>(this IStack\<T\> s, T t)  
      {  
        return Stack\<T\>.Push(t, s);  
      }  
    }

and now we can say

 

IStack\<Giraffe\> sg1 = Stack\<Giraffe\>.Empty;  
IStack\<Giraffe\> sg2 = s1.Push(new Giraffe("Gerry"));  
IStack\<Giraffe\> sg3 = s2.Push(new Giraffe("Geoffrey"));  
IStack\<Mammal\> sm3 = sg3; // Legal because of covariance.  
IStack\<Mammal\> sm4 = sm3.Push(new Tiger("Tony"));

Is that slick or what?

Next time: what about an immutable queue?

