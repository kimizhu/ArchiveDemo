# References and Pointers, Part Two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/10/2011 9:50:00 AM

-----

Here's a handy type I whipped up when I was translating some complex pointer-manipulation code from C to C\#. It lets you make a safe "managed pointer" to the interior of an array. You get all the operations you can do on an unmanaged pointer: you can dereference it as an offset into an array, do addition and subtraction, compare two pointers for equality or inequality, and represent a null pointer. But unlike the corresponding unsafe code, this code doesn't mess up the garbage collector and will assert if you do something foolish, like try to compare two pointers that are interior to different arrays. (\*) Enjoy\!

 

internal struct ArrayPtr\<T\>  
{  
  public static ArrayPtr\<T\> Null { get { return default(ArrayPtr\<T\>); } }  
  private readonly T\[\] source;  
  private readonly int index;

  private ArrayPtr(ArrayPtr\<T\> old, int delta)  
  {  
    this.source = old.source;  
    this.index = old.index + delta;  
    Debug.Assert(index \>= 0);  
    Debug.Assert(index == 0 || this.source \!= null && index \< this.source.Length);  
  }

  public ArrayPtr(T\[\] source)  
  {  
    this.source = source;  
    index = 0;  
  }

  public bool IsNull()  
  {  
    return this.source == null;  
  }

  public static bool operator \<(ArrayPtr\<T\> a, ArrayPtr\<T\> b)  
  {  
    Debug.Assert(Object.ReferenceEquals(a.source, b.source));  
    return a.index \< b.index;  
  }  
         
  public static bool operator \>(ArrayPtr\<T\> a, ArrayPtr\<T\> b)  
  {  
    Debug.Assert(Object.ReferenceEquals(a.source, b.source));  
    return a.index \> b.index;  
  }  
         
  public static bool operator \<=(ArrayPtr\<T\> a, ArrayPtr\<T\> b)  
  {  
    Debug.Assert(Object.ReferenceEquals(a.source, b.source));  
    return a.index \<= b.index;  
  }  
         
  public static bool operator \>=(ArrayPtr\<T\> a, ArrayPtr\<T\> b)  
  {  
    Debug.Assert(Object.ReferenceEquals(a.source, b.source));  
    return a.index \>= b.index;  
  }  
        
  public static int operator -(ArrayPtr\<T\> a, ArrayPtr\<T\> b)  
  {  
    Debug.Assert(Object.ReferenceEquals(a.source, b.source));  
    return a.index - b.index;  
  }  
         
  public static ArrayPtr\<T\> operator +(ArrayPtr\<T\> a, int count)  
  {  
    return new ArrayPtr\<T\>(a, +count);  
  }  
         
  public static ArrayPtr\<T\> operator -(ArrayPtr\<T\> a, int count)  
  {  
    return new ArrayPtr\<T\>(a, -count);  
  }  
         
  public static ArrayPtr\<T\> operator ++(ArrayPtr\<T\> a)  
  {  
    return a + 1;  
  }  
       
  public static ArrayPtr\<T\> operator --(ArrayPtr\<T\> a)  
  {  
    return a - 1;  
  }

  public static implicit operator ArrayPtr\<T\>(T\[\] x)  
  {  
    return new ArrayPtr\<T\>(x);  
  }

  public static bool operator ==(ArrayPtr\<T\> x, ArrayPtr\<T\> y)  
  {  
    return x.source == y.source && x.index == y.index;  
  }

  public static bool operator \!=(ArrayPtr\<T\> x, ArrayPtr\<T\> y)  
  {  
    return \!(x == y);  
  }

  public override bool Equals(object x)  
  {  
    if (x == null) return this.source == null;  
    var ptr = x as ArrayPtr\<T\>?;  
    if (\!ptr.HasValue) return false;  
    return this == ptr.Value;  
  }

  public override int GetHashCode()  
  {  
    unchecked  
    {  
      int hash = this.source == null ? 0 : this.source.GetHashCode();  
      return hash + this.index;  
    }  
  }

  public T this\[int index\]  
  {  
    get { return source\[index + this.index\]; }  
    set { source\[index + this.index\] = value; }  
  }  
}

Now we can do stuff like:

 

double\[\] arr = new double\[10\];  
var p0 = (ArrayPtr\<double\>)arr;  
var p5 = p0 + 5;  
p5\[0\] = 123.4; // sets arr\[5\] to 123.4  
var p7 = p0 + 7;  
int diff = p7 - p5; // 2

Pretty neat, eh?

UPDATE:

A number of people in the comments have asked why the code disallows a pointer "past the end" of the array. In fact the original C code that I was porting did use an invalid "marker" value as the "end of array" marker, and the code did thereby manipulate pointers "past the end" of the array. The original version of the ArrayPtr class that I actually used in the port to C\# supported having a pointer one past the end of the array, and threw an exception if you ever tried to dereference it. I thought this detail was distracting from the point of the article so I eliminated the feature from the C\# code before I posted it. Perhaps that was a premature optimization.

I have a similar C\# wrapper type for strings, where again, I permit a pointer "past the end" of the string where the null-terminating character would be in a C program. That class also supports common C idioms like "strlen" and whatnot. Such types are very handy when porting C code to C\#; ultimately of course it is better to use C\# idioms in the long run, but in the short run it is very useful to be able to get things working quickly.

\------------

(\*) Were this to be a public type then I'd make the assertions into exceptions because there is no telling what crazy thing the public is going to do; since this is an internal type I can guarantee that I'm using it correctly, so I'll use an assertion instead.

