# Instance-bound nested classes in JScript .NET

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/2/2005 2:30:00 PM

-----

The other day someone asked me about a slightly odd but very useful feature in JScript.NET, and I figure if one person asks me about it, maybe more people want to know as well.  In JScript.NET you can say class alpha {  
  var foo;  
  class bravo {  
    function blah() {  
      print (foo);  
    }  
  }  
} var x = new alpha();  
x.foo = 123;  
var z = new x.bravo();  
z.blah(); the instance of alpha is bound to the *instance* of bravo's scope chain. Therefore when z.blah runs, foo is bound to x.foo. Of course, this means that when you instantiate a new instance of the inner class, you've got to do so via an instance of the enclosing class. We implemented this in JScript .NET so that ASP.NET would work nicely. Behind the scenes, ASP.NET builds a class which extends a page class and sticks your code into it. If you had something like \<script runat="server"\>  
  class foo {  
    function bar() {  
      Response.Write("hello");  
    }  
  }  
\</script\>  
\<%  
  var x = new foo();  
  foo.bar();  
%\> Then that would generate a class roughly like this:  
  
class MyPage extends Page {  
  class foo {  
     function bar() {  
       Response.Write("hello");  
     }  
  }  
  function Render() {   
    var x = new foo();  
    foo.bar();  
  }  
} The nested class is bound to the page instance, and hence can access the Response object of the page base class. The JScript.NET compiler is smart enough to figure out that really you meant     var x = new this.foo(); C\# does not implement instance-bound nested classes. If you want C\#-like behaviour in JScript.NET, you can get it. In JScript.NET if you don't want a nested class to be bound to an instance of the outer class then you stick the static modifier onto the inner class declaration. In that case, the enclosing class essentially becomes not much more than another level of namespace. Would you like to see something like this in a hypothetical future version of C\#? Would it be useful?

