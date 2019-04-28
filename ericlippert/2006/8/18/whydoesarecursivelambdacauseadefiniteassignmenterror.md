# Why does a recursive lambda cause a definite assignment error?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 8/18/2006 3:09:00 PM

-----

Hey fabulous readers, sorry for not much blogging lately. Between implementing LINQ and making plans to attend my first Burning Man, there's been no time for anything. We've had lots of new ideas generated here for the type inferencing algorithm which I will discuss in detail when I get back.

For now, here's a question I got recently about definite assignment analysis and recursive functions. Clearly this is legal:

delegate void Action\<A\>(A a);  
delegate void Traversal\<T\>(Tree\<T\> t, Action\<T\> a);  
  
static void DepthFirstTraversal\<T\>(Tree\<T\> t, Action\<T\> a){  
  if (t == null) return;  
  a(t.Value);  
  DepthFirstTraversal(t.Left, a);  
  DepthFirstTraversal(t.Right, a);  
}  
static void Traverse\<T\>(Tree\<T\> t, Traversal\<T\> f, Action\<T\> a){  
  f(t, a);  
}  
static void Print\<T\>(T t){  
  System.Console.WriteLine(t.ToString());  
}  
static void PrintTree\<T\>(Tree\<T\> t){  
  Traverse(t, DepthFirstTraversal, Print);  
}  

What if we wanted that last one to use lambdas?

static void PrintTree\<T\>(Tree\<T\> t){  
  Traversal\<T\> df = (t, a)=\>{  
    if(t==null) return;  
    a(t.Value);  
    df(t.Left, a);  
    df(t.Right, a);  
  };  
  Traverse(t, df, Print);  
}  

That doesn't work because it's a definite assignment error. The compiler says that local variable df is being used before it is assigned. You'd have to say

Traversal\<T\> df = null;  
df = (t, a)=\>{  

And suddenly it would start working.

What the heck? I mean, the original code is a syntactic sugar for:

privateclass Locals{  
  public Traversal\<T\> df;  
  public void M\<T\>(Tree\<T\> t, Action\<T\> a){  
    if(t==null) return;  
    a(t.Value);  
    this.df(t.Left, a);  
    this.df(t.Right, a);  
  }  
}  
static void PrintTree\<T\>(Tree\<T\> t){  
  Locals locals = new Locals();  
  locals.df = new Traversal\<T\>(locals.M\<T\>);  
  Traverse(t, locals.df, Print);  
}  

And clearly there is no use-before-initialization problem there.

The reason this is an error is because though there is actually no definite assignment problem here, it is not too hard to find a similar situation which does create a problem. Here's a silly example that nevertheless demonstrates a real problem:

 

delegate D D(D d);  
//...  
D d = ((D)(c=\>c(d))(e=\>e); 

If you trace out the logic of that thing you'll find that in fact the local variable delegate d is potentially used before it is initialized, and therefore this must be an error.

We could immensely complicate the definite assignment rules by adding rules to discover which lambda and anonymous method bodies are guaranteed to be never called during the initialization, and therefore suppress definite assignment errors on some locals in their bodies. But one of the goals of the standardization process is to make rules that are clear and unambiguous and comprehensible by mortals; enabling recursive anonymous methods isn't a big enough win for the amount of complexity, particularly when you consider that there is the simple workaround of assigning to null and then reassigning.

