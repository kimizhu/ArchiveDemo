<div id="page">

# Immutability in C\# Part Three: A Covariant Immutable Stack

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 12/6/2007 10:55:00 AM

-----

<div id="content">

<div class="mine">

Now suppose we had a hypothetical future version of C\# in which interface covariance worked, and we wanted a covariant immutable stack. That is, we want to be able to implicitly convert an <span class="code">IStack\<Giraffe\></span> to <span class="code">IStack\<Mammal\></span>. As we've already discussed, this doesn't make much sense in an array, even though doing so is legal in C\# today. If you cast a <span class="code">Giraffe\[\]</span> to <span class="code">Mammal\[\]</span> then you can try to put a <span class="code">Tiger</span> into the <span class="code">Mammal\[\]</span> and it will fail at run time. It seems like the same should be true of stacks -- if you cast an <span class="code">IStack\<Giraffe\></span> to <span class="code">IStack\<Mammal\></span> then you can push a <span class="code">Tiger</span> onto the stack of <span class="code">Giraffe</span>s, which should fail, right?

Maybe. Maybe not.

The interface we defined the other day is not suitable for covariance because it uses the variant parameter as both an input and and output. Let's get a bit crazy for a minute here -- suppose we got rid of the input on the <span class="code">Push</span>:

<span class="code"> </span>

    public interface IStack\<+T\> : IEnumerable\<T\>  
    {  
        IStack\<T\> Pop();  
        T Peek();  
        bool IsEmpty { get; }  
    }

This interface is now suitable for covariance. If you have a stack of <span class="code">Giraffe</span>s and you want to treat it as a stack of <span class="code">Mammal</span>s, you can do so with perfect type safety, since we know that we will never be pushing a <span class="code">Tiger</span> onto that thing. We'll only be reading off <span class="code">Giraffe</span>s, all of which are <span class="code">Mammal</span>s. This may seem less than useful, but we'll see what we can do:

<span class="code"> </span>

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

Notice that <span class="code">Push</span> has disappeared from the empty stack and is static on the nonempty stack, and hey, check it out:

<span class="code"> </span>

IStack\<Giraffe\> s1 = Stack\<Giraffe\>.Empty;  
IStack\<Giraffe\> s2 = Stack\<Giraffe\>.Push(new Giraffe("Gerry"), s1);  
IStack\<Giraffe\> s3 = Stack\<Giraffe\>.Push(new Giraffe("Geoffrey"), s2);  
IStack\<Mammal\> s4 = Stack\<Mammal\>.Push(new Tiger("Tony"), s3);

Oh my goodness, we just pushed a <span class="code">Tiger</span> onto a stack of <span class="code">Mammal</span>s that is actually a stack of <span class="code">Giraffe</span>s underneath, but everything is still typesafe\! There is nothing we can do here to cause an exception at runtime in the type system. It all just works. All the stacks of <span class="code">Giraffe</span>s continue to be stacks of <span class="code">Giraffe</span>s; they're immutable, so they could hardly be anything else.

This code is pretty ugly though. So...

<span class="code"> </span>

    public static class Extensions  
    {  
      public static IStack\<T\> Push\<T\>(this IStack\<T\> s, T t)  
      {  
        return Stack\<T\>.Push(t, s);  
      }  
    }

and now we can say

<span class="code"> </span>

IStack\<Giraffe\> sg1 = Stack\<Giraffe\>.Empty;  
IStack\<Giraffe\> sg2 = s1.Push(new Giraffe("Gerry"));  
IStack\<Giraffe\> sg3 = s2.Push(new Giraffe("Geoffrey"));  
IStack\<Mammal\> sm3 = sg3; // Legal because of covariance.  
IStack\<Mammal\> sm4 = sm3.Push(new Tiger("Tony"));

Is that slick or what?

Next time: what about an immutable queue?

</div>

</div>

</div>

