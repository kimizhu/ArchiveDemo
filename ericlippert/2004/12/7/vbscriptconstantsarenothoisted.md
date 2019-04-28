# VBScript Constants Are Not Hoisted

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 12/7/2004 11:02:00 AM

-----

A reader asked me the other day to riff a bit on why this works: Dim i  
For i = 1 To 2  
    print c  
Next  
Const c = 10 And this works For i = 1 To 2  
    print c  
    Dim i  
Next  
Const c = 10 but this fails with a "name redefined" error: For i = 1 To 2  
    print c  
    Const c = 10  
    Dim i  
Next To understand what's going on here, there are two preliminaries we should get out of the way. First, refresh your memory on [how constants work in VBScript](http://blogs.msdn.com/ericlippert/archive/2003/10/21/53264.aspx). Second, refresh your memory on [the hoisting algorithm](http://blogs.msdn.com/ericlippert/archive/2004/06/18/159378.aspx). OK, so we know the following:

  - **variable** declarations are logically hoisted to the top of the scope
  - **constants** are evaluated at code compilation time; the constants' values are spit into the code

Now, the weird part. **These facts do not imply that constant declarations are hoisted.** Constant declarations appear at first glance as though they are logically hoisted because the code generator spits the values into the code. But there are ways of exposing the fact that they are in fact not hoisted. In order to illumine how this works, I'll show you guys a bit of the VBScript code generator output. (Unless you've somehow managed to sneak into my office and obtain a debug build of VBScript, you don't have the utility that produces code generator dumps, so don't even ask\!) Let's start simple. i = 10  
print i  
Dim i ' will be hoisted generates this code: VarBind 'i' 1          Bind name 'i' to local variable slot 1  
Bos 0                  beginning of statement 0  
IntConst 10            put 10 on stack  
LocalSt 1              store top of stack in slot 1  
Bos 1  
LocalAdr 1             put reference to slot 1 on stack  
Call 'print' 1         call print with one argument (As you can see, the script engine is a "stack machine". Arguments for each opcode are put onto the stack, and each opcode consumes them off the stack. There are no "registers".) This, on the other hand print c  
Const c = 10 generates this code: Bos 0  
IntConst 10  
Call 'print' 1  
Bos 1  
IntConst 10  
ConstSt 'c' Two things pop out. First, the call to print is not referencing any kind of slot; it just gets the number 10. Second, no declaration for 'c' is hoisted to the top. The slot 'c' is not bound until the statement runs; it appears to be bound earlier, but that's an illusion created by the code generator spitting the constant. Now it should be clear what happens here: For i = 1 To 2  
    print c  
    Const c = 10  
Dim i  
Next This generates the code VarBind 'i' 1  
Bos 0  
IntConst 1           from  
IntConst 2           to  
IntConst 1           step  
For 1                use slot 1 for the loop variable  
Bos 1  
IntConst 10  
Call 'print' 1  
Bos 2  
IntConst 10  
ConstSt 'c'  
Bos1 3  
Next The first time that the constant is stored, it creates a new named slot called 'c', assigns the value, and marks the slot as "not writable". The second time, it tries to create the new slot, discovers that it exists, and raises an error. OK, this is a little weird, but it seems to be behaving pretty sensibly. Though the declaration is not hoisted, the code spit lets you use the value before the slot is created. Though the slot is created later, it can only be stored once, as you'd expect with a constant. What gets **really weird** is when you make use of the slot **before it is bound**: c = 11  
print c  
Const c = 10 This prints out "10" and then dies on the third line. The first line creates a named slot. The second line prints out the compile-time spit constant. The third line attempts to create a read-only slot but discovers that a slot with that name already exists, and errors out. Bos 0  
IntConst 11  
NamedSt 'c'  
Bos 1  
IntConst 10  
Call 'print' 1  
Bos 2  
IntConst 10  
ConstSt 'c'  
  
For similar reasons, this doesn't do what you might think: print TypeName(c)  
print Eval("TypeName(c)")  
Const c = 10 The first statement prints "Integer", because 10 is spit into the code gen, not a reference to 'c'. But a brand-new code generator is created when Eval is called, and it has no idea that earlier this value was bound to 10 at compile time. The evaluation engine therefore creates a new slot called 'c', initializes it to Empty, and runs the code, which prints "Empty". Then, when the third statement runs, it fails because the slot already exists. Ouch\! This is one of the weird situations where evaluating an expression with Eval yields a different result than just evaluating the expression normally. Long story short: as Groucho Marx once said, **if it hurts when you do that then** **don't do that**. Put your constant declarations at the top of your program like a sensible person and no one gets hurt\!

