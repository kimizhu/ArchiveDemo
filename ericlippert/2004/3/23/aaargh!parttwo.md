# Aaargh\! Part Two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/23/2004 10:58:00 AM

-----

I'm insanely busy prepping for VSLive\! today, so, once more, I complain about coding practice that are [drivin' me nuts](http://blogs.msdn.com/ericlippert/archive/2004/03/10/87384.aspx).

But first: why do pirates dislike Johnny Depp's Oscar-nominated performance in "Pirates of the Caribbean"?

Because it wasn't AAAAAARRRR-rated.

Gripe \#3: More Complaining About Bad Macros

One day I was debugging a bunch of macro-ridden code that I didn't write. I wanted to know what the method signature of

CForm::GetMouseIcon was. So, silly me, I grepped the source code for CForm::GetMouseIcon. Nothing. Must be defined in the class header itself\! So I grepped for GetMouseIcon. Nothing. Aaargh\! It was drivin' me nuts\!

Inspiration hit -- I grepped for

MouseIcon and found in some header somewhere

DEFINE\_DERIVED\_GETSET\_METHODS(CForm, CServer, MouseIcon, IID\_IPicture)

Aaargh\! I *still* didn't know what the signature was until I managed to find that macro definition and decode it\! Why do people write code like this? You can't easily debug into

CForm::GetMouseIcon, you can't find it easily using grep, etc. Why define this stuff in macros? Does it really make anyone's life easier?

Gripe \#4: Exception Handling, Return Values and Smart Pointers

You probably knew I couldn't actually *not* complain about smart pointers,

[the bane of my existence](http://blogs.msdn.com/ericlippert/archive/2003/09/16/53016.aspx). I'm so predictable.

Before I start complaining about exception handling, please, I don't want to have a replay of that *Joel On Software* debate about the merits of structured exception handling. I think exception handling is super. I love writing code in C\# that uses exception handling.

catch allows you to isolate your error handling code into strictly scoped blocks, and finally allows you to isolate your cleanup code, and that's all good.

The problem I have with structured exception handling is that programs that mix libraries that use SEH and libraries that return error codes, (like, say, *every single COM API*) often turn into a godawful mess. You end up with the worst of both worlds: exceptions potentially going off all over the place acting as non-local gotos, and still having to pay careful attention to error return codes, thereby de-localizing the error handling code. C\# and the .NET Framework were designed to use exception handling from day one, so it makes a lot of sense. COM was not\!

As we saw

[last time](http://blogs.msdn.com/ericlippert/archive/2004/03/10/87384.aspx), the tempting thing to do is to wrap every error-returning function with a macro that turns it into a throw, and wrap every must-be-released handle with a smart object that will clean up when it goes out of scope. But the resulting complexity means that often the cure is worse than the disease. We've already seen the havoc that macros wreak. What about smart pointers?

Case in point -- one of my coworkers and I were debugging a weird problem in some of our code just last week. We called into a particular API which was setting the

GetLastError value. Something in our code was blowing it away, so we wrote some code to preserve it. It didn't work; when our function returned, GetLastError was still reporting that there had been no error. How could that be? What's wrong with this picture?

  // Ensure that we've preserved the error state we captured earlier  
  SetLastError(blah);  
  return hr;  
}

The caller of this method calls  GetLastError , but the state we just set has been blown away. OK, smart people, where does that happen?

Turned out that a smart pointer in the current scope was holding on to a proxy for an out-of-process COM object, and when the smart pointer went away, the out-of-proc object was released, which resulted in a whole slew of calls behind the scenes, which resulted in the

GetLastError value being overwritten. Of course, since this was a smart pointer destructor doing all this, the source code for the call site looked like this:

} \<-- calls destructor

Aargh\! It's drivin' me nuts\! Call me crazy, but I want operations bearing complex semantics and causing side effects that break my error handling to look **a little more like function calls and a little less like punctuation**. Had the code been written using dumb pointers, no problem

  // Ensure that we've preserved the error state we captured earlier  
  SetLastError(blah);  
  pProxy-\>Release();  
  return hr;  
} \<-- no op

Now it's obvious where the mistake is.

COM programming demands discipline and a keen understanding of how to write correct error-path code **whether you also use exception handling or not**. Therefore, I say don't use exception handling if you're going to be doing a whole lot of COM-style error handling too\! I like to think that I'm a fairly bright guy, but **I have great difficulty analyzing the semantics of code that has macros, template classes, smart pointers, exception raisers, error return values and cross-process COM calls all *in the same routine*. I'm not that smart\!**

I've developed a coding style that works very well with COM -- it is verbose, it is rigid, and it is very, very clear to even novice programmers exactly what the lifetime of all pointers in the code are -- and my code rarely has reference leaks because of it, even in error cases. And when it does, they are a snap to track down because every time a pointer is addrefed there is a call to AddRef and every time it is released there is a call to Release right there in the code, not tucked away in some template library. It's not hard to develop a good coding standard.

Next time: even more on bad interactions between exception handling and memory management

