# What's the difference between conditional compilation and the conditional attribute?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/10/2009 10:12:00 AM

-----

**User**: Why does this program not compile correctly in the release build?

 

class Program  
{  
\#if DEBUG  
    static int testCounter = 0;  
\#endif   
    static void Main(string\[\] args)  
    {  
        SomeTestMethod(testCounter++);  
    }   
    \[Conditional("DEBUG")\]  
    static void SomeTestMethod(int t) { }  
}

**Eric**: This fails to compile in the retail build because testCounter cannot be found in the call to SomeTestMethod.

**User**: But that call site is going to be omitted anyway, so why does it matter? Clearly there's some difference here between removing code with the conditional compilation directive versus using the conditional attribute, but what's the difference?

**Eric**: You already know the answer to your question, you just don't know it yet. Let's get Socratic; let me turn this around and ask you how this works. How does the compiler know to remove the method call site? 

**User**: Because the method called has the conditional attribute on it.

**Eric**: *You* know that. But how does the *compiler* know that the method called has the conditional attribute on it?

**User**: Because *overload resolution* chose that method. If this were a method from an assembly, the *metadata* associated with that method has the attribute. If it is a method in source code, the compiler knows that the attribute is there because the compiler can analyze the source code and figure out the meaning of the attribute.

**Eric**: I see. So fundamentally, overload resolution does the heavy lifting. How does overload resolution know to choose that method? Suppose hypothetically there were another method of the same name with different parameters.

**User**: Overload resolution works by examining the *arguments* to the call and comparing them to the *parameter types* of each candidate method and then choosing the unique best match of all the candidates.

**Eric**: And there you go. Therefore *the arguments must be well-defined at the point of the call, even if the call is going to be removed*. In fact, the call *cannot* be removed unless the arguments are extant\! But in the release build, the type of the argument cannot be determined because its declaration has been removed.

So now you see that the real difference between these two techniques for removing unwanted code is *what the compiler is doing when the removal happens*. At a high level, the compiler processes a text file like this. First it "lexes" the file. That is, it breaks the string down into "tokens" -- sequences of letters, numbers and symbols that are meaningful to the compiler. Then those tokens are "parsed" to make sure that the program conforms to the grammar of C\#. Then the parsed state is analyzed to determine *semantic* information about it; what all the types are of all the expressions and so on. And finally, the compiler spits out code that implements those semantics.

The effect of a conditional compilation directive happens at *lex* time; anything that is inside a removed \#if block is treated by the lexer as a comment. It's like you simply deleted the whole contents of the block and replaced it with whitespace. But removal of call sites depending on conditional attributes happens at *semantic analysis* time; **everything necessary to perform that semantic analysis must be present. **

**User**: Fascinating. Which parts of the C\# specification define this behavior?

**Eric**: The specification begins with a handy “table of contents”, which is very useful for answering such questions. The table of contents states that section 2.5.1 describes "Conditional compilation symbols" and section 17.4.2 describes "The Conditional attribute".

**User**: Awesome.

