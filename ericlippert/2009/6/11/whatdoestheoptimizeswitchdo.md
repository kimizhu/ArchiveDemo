# What does the optimize switch do?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/11/2009 6:57:00 AM

-----

I was asked recently exactly what optimizations the C\# compiler performs when you specify the optimize switch. Before I answer that, I want to make sure that something is perfectly clear. The compiler’s “usage” string is not lying when it says:

/debug\[+|-\]     Emit debugging information  
/optimize\[+|-\]  Enable optimizations

Emitting debug information and optimizing the generated IL are orthogonal; they have no effect on each other at all (\*). The *usual* thing to do is to have debug on and optimizations off, or vice versa, but the other two combinations are perfectly legal. (And while I’m at it, turning debug information generation on does not also do /d:DEBUG; that’s also orthogonal. Again, the sensible thing to do is to keep /debug and /d:DEBUG in sync, but you do not have to; you might want to debug the optimized version without assertions and we’re not going to stop you.)

From this point on, when I say “emitting” I mean “producing metadata”, and when I say “generating” I mean “producing IL”.

So, first off, there are some optimizations that the compiler always performs, regardless of the optimize flag.

The language semantics require us to do constant folding because we need to detect that this is an error:

switch(x) {  
case 2:  
case 1+1:  
…

The compiler will replace usages of constant fields with the constant, but does not perform any more sophisticated [constant propagation](http://en.wikipedia.org/wiki/Constant_propagation), even though the compiler has a reachability analyzer.

Speaking of which, there are two kinds of reachability analysis we perform for [dead code elimination](http://en.wikipedia.org/wiki/Dead_code_elimination). First, there is the “according to spec” reachability. The specification calls out that reachability analysis consumes compile-time constants. This is the reachability analysis we use to give “unreachable code” warnings, to determine whether local variables are definitely assigned on all code paths, and to determine whether the end point of a non-void method is reachable. If this first pass determines that code is unreachable then we never generate IL for it. For example, this is optimized away:

if (false) M();

But if the expression involves non-compile-time constants then this first form of reachability analysis would tell you that the call to N here is reachable, and therefore is a definite assignment error:

int x = M();  
int y;  
if (x \* 0 \!= 0) N(y);

If you fixed that up so that y was definitely assigned, then the first-pass reachability analyzer would NOT trim this code because it still thinks that the call is reachable.

But clearly you and I know that the call is not reachable, because we know more about arithmetic than the specified reachability analyzer. We perform a second pass of reachability analysis during code generation which is much smarter about these things. We can do that because the second pass only affects codegen, it does not affect language semantics, error reporting, and so on.

The existence of that second pass implies that we do a simple arithmetic optimizations on expressions which are only partially constant. For example, if you have a method M that returns an integer, then code like

if (M() \* 0 == 0) N();

can be generated as though you’d written just:

M();  
N();

We have lots of simple number and type algebra optimizers that look for things like adding zero to an integer, multiplying integers by one, using “null” as an argument to “is” or “as”, concatenation of literal strings, and so on. The expression optimizations always happen, whether you’ve set the optimize flag or not; whether basic blocks that are determined to be unreachable after those optimizations are trimmed depends on the flag.

We also perform some small optimizations on some call sites and null checks. (Though [not as many as we could](http://stackoverflow.com/questions/946999/curious-c-using-statement-expansion).) For example, suppose you have a non-virtual instance method M on a reference type C, and a method GetC that returns a C. If you say GetC().M() then we generate a callvirt instruction to call M. Why? Because it is *legal* to generate a callvirt for an instance method on a reference type, and callvirt automatically inserts a null check. The non-virtual call instruction does not do a null check; we'd have to generate extra code to check whether GetC returns null. So that's a small code size optimization. But we can do even better than that; if you have (new C()).M(), we generate a call instruction because we know that the result of the "new" operator is never null. That gives us a small time optimization because we can skip the nanosecond it takes to do the null check. Like I said, it's a small optimization.

The /optimize flag does not change a huge amount of our emitting and generation logic. We try to always generate straightforward, verifiable code and then rely upon the jitter to do the heavy lifting of optimizations when it generates the real machine code. But we will do some simple optimizations with that flag set. For example, with the flag set:

  - Expressions which are determined to be only useful for their side effects are turned into code that merely produces the side effects.  
  - We omit generating code for things like int foo = 0; because we know that the memory allocator will initialize fields to default values.  
  - We omit emitting and generating empty static class constructors. (Which typically happens if the static constructor set all the fields to their default value and the previous optimization eliminated all of them.)  
  - We omit emitting a field for any hoisted locals that are unused in an iterator block. (This includes that case where the local in question is used only inside an anonymous function in the iterator block, in which case it is going to become hoisted into a field of the closure class for the anonymous function. No need to hoist it twice if we don’t need to.)  
  - We attempt to minimize the number of local variable and temporary slots allocated. For example, if you have:

for (int i = …)  {…}  
for (int i = …) {…}

then the compiler could generate code to re-use the local variable storage reserved for i when the second i comes along. (We eschew this optimization if the locals have different names because then it gets hard to emit sensible debug info, which we still want to do even for the optimized build. However, the jitter is free to perform this optimization if it wishes to.)

  - Also, if you have a local which is never used at all, then there is no storage allocated for it if the flag is set.  
  - Similarly, the compiler is more aggressive about re-using the unnamed temporary slots sometimes used to store results of subexpression calculations.  
  - Also, with the flag set the compiler is more aggressive about generating code that throws away “temporary” values quickly for things like controlling variables of switch statements, the condition in an “if” statement, the value being returned, and so on. [In the non-optimized build these values are treated as unnamed local variables, loaded from and stored to specific locations.](http://stackoverflow.com/questions/944443/what-are-these-opcodes-for) In the optimized build they can often be just kept on the stack proper.  
  - We eliminate pretty much all of the “breakpoint convenience” no-ops.  
  - If a try block is empty then clearly the catch blocks are not reachable and can be trimmed. (Finally blocks of empty tries are preserved as protected regions because they have unusual behaviour when faced with certain exceptions; see the comments for details.)  
  - If we have an instruction which branches to LABEL1, and the instruction at LABEL1 branches to LABEL2, then we rewrite the first instruction as a branch straight to LABEL2. Same with branches that go to returns.  
  - We look for “branch over a branch” situations. For example, here we go to LABEL1 if condition is false, otherwise we go to LABEL2.   
      
    brfalse condition, LABEL1  
    br LABEL2  
    LABEL1: somecode  
      
    Since we are simply branching over another branch, we can rewrite this as simply "if condition is true, go to LABEL2":  
      
    brtrue condition, LABEL2  
    somecode  
  - We look for “branch to nop” situations. If a branch goes to a nop then you can make it branch to the instruction after the nop.  
  - We look for “branch to next” situations; if a branch goes to the next instruction then you can eliminate it.  
  - We look for two return instructions in a row; this happens sometimes and obviously we can turn it into a single return instruction.

That’s pretty much it. These are very straightforward optimizations; there’s no inlining of IL, no loop unrolling, no interprocedural analysis whatsoever. We let the jitter team worry about optimizing the heck out of the code when it is actually spit into machine code; that’s the place where you can get real wins.

-----

(\*) A small lie. There is some interaction between the two when we generate the attributes that describe whether the assembly is Edit’n’Continuable during debugging.

