# Past performance is no guarantee of future results

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/31/2012 7:13:00 AM

-----

Before I get started today, a couple housekeeping notes. First off, sorry for no blog the last three weeks; I have been crazy busy adding features to the Roslyn C\# semantic analyzer. More on that in an upcoming episode. Second, check out the snazzy new [Developer Tools blog aggregation page](http://blogs.msdn.com/b/developer-tools/); it's one stop shopping for lots of great information, and it's mostly purple. [Vindication](http://www.codinghorror.com/blog/2006/12/eric-lipperts-purple-crayon.html)\!

OK, now that we've got the metablogging out of the way, on to a question I get occasionally:

**Is compiling the same C\# program twice guaranteed to produce the same binary output?**

No.

Well, that was an easy blog to write.

Maybe some more background might be useful.

**What's up with that?**

Well, a **language** is fundamentally nothing more than a set, possibly infinite, of the strings (\*); a string is a valid program if it is in the set, and invalid if it is not. A **compiler** is a program that takes a string, determines if it is a valid member of a particular language, and if it is, then produces as its output an equivalent (\*\*) valid string in a different language. For example, the C\# compiler takes as its input a string stored in a ".cs" file and produces as its output a program written in the MSIL language. There are of course a whole host of pragmatic details; the C\# compiler produces MSIL that has been encoded into a binary "portable executable" format rather than human-readable format. And the C\# compiler also takes in as inputs programs written in this binary format in the form of library references. But at a fundamental level, the job that the C\# compiler does for you is it takes in C\# code, analyzes it for correctness, and produces an equivalent program written in the PE format.

Any given C\# program has infinitely many equivalent MSIL programs; as a trivial example, we can insert as many extra nop "no operation" instructions as we like, anywhere we like. As a silly example, we can insert any verifiable code whatsoever in any unreachable code path\! And as a more interesting example, there are frequently situations in which the compiler must "make up" unique names for things like anonymous methods, anonymous types, fields of closure classes, and so on. I thought I'd talk a bit about those unique names.

When the compiler needs to generate a unique name for something its basic strategy is to pattern for a name that cannot possibly be a legal name, because it contains characters that MSIL considers legal in identifiers but C\# does not. It then inserts into the pattern strings such as the name of the current method, and finally, puts a unique number on the end. I described the exact pattern we use in [this StackOverflow answer](http://stackoverflow.com/questions/2508828/where-to-learn-about-vs-debugger-magic-names); note that this is an implementation detail of the compiler that we are considering changing in the future. (People occasionally complain that the names are too long and are eating up space in the metadata tables; we could make them much shorter.) The key point here is that uniqueness is guaranteed by incrementing a counter, which is a pretty easy way to guarantee uniqueness. As a practical matter we are not going to run out of unique identifiers; they only need to be unique within a given assembly, and no assembly has more than two billion compiler-generated identifiers.

This means that the actual content of a compiler-generated identifier is determined by the order in which the compiler gets around to analyzing a particular hunk of code that needs to generate a magical identifier. We make no guarantees that this order is deterministic\! Historically it has been mostly deterministic; the C\# compiler's analysis engine in the past has been single-threaded, and, if given the same list of files twice, will analyze the files in the same order and then analyze all the types in the program in the same order (\*\*\*). But note the assumptions here:

First off, we are assuming that we always get the same list of files every time, in the same order. But that's in some cases up to the operating system. When you say "csc \*.cs", the order in which the operating system proffers up the list of matching files is an implementation detail of the operating system; the compiler does not sort that list into a canonical order.

Second, we are assuming that the analysis engine of the compiler is single-threaded; there is no requirement that it be single threaded, and in fact we have toyed with making it multi-threaded in the past and likely will again in the future. Analyzing large programs is a great example of an "embarrassingly parallel" problem. Once the large-scale structure of the program -- all the types, methods, fields, and so on -- is known, then every method body can be analyzed in parallel; the contents of one method body never affect the analysis of a different method body. But once you make the compiler multi-threaded, there is no guarantee of any kind that the order is deterministic, and hence no guarantee that two compilations produce the same magic identifiers.

Now, all this is very interesting, which is why I told you all about it. I could have just cut right to the chase, which is to say: **the C\# compiler by design never produces the same binary twice**. The C\# compiler embeds a freshly generated GUID in every assembly, every time you run it, thereby ensuring that no two assemblies are ever bit-for-bit identical. To quote from the CLI specification:

> The Mvid column shall index a unique GUID \[...\] that identifies this instance of the module. \[...\] The Mvid should be newly generated for every module \[...\] While the \[runtime\] itself makes no use of the Mvid, other tools (such as debuggers \[...\]) rely on the fact that the Mvid almost always differs from one module to another.

So there you go; the runtime specification requires that every module (an **assembly** being a collection of one or more **modules**) have a unique identifier.

Moreover: let's not forget here that it's not the IL that runs; yet another compiler will translate the IL to machine code. The jitter certainly does not guarantee that jit compiling the same code twice produces the same machine code; the jitter can be using all kinds of runtime information to tweak the generated code to optimize it for size, for speed, for debuggability, and so on, at its whim.

**Assuming that the compiler produces correct code, why would you care if compiling the same program twice produces exactly the same output or not?**

The first customer to ask me whether the C\# compiler was deterministic had an interesting scenario. The customer was in an industry where their code was subject to government regulation. The government wanted to obtain the source code and the binary code that was going to be actually burned into a chip from the customer. The regulators were planning on performing a security review of the human-readable source code before allowing devices containing the chip to be approved for use in the state. Of course, the obvious attack by an unscrupulous provider is to make the source code and binary code not match; the source is benign and the binaries are hostile.

The regulator's strategy was going to be to compile the source code and then do a bit-for-bit comparison to the binary code, which of course you know now, will always fail in C\#.

What I suggested to them was: since the regulator was going to be doing the compilation anyways for the purposes of comparison, then the regulatory body should only take in the source code in the first place; they can give the binaries back to the company. That way the regulators have a guarantee that the binaries match the source code, because the regulators produced the binaries in the first place.

I believe they ignored my advice and ended up doing a diff and then verifying that the differing bits were only found in the Mvid column mentioned above. Of course I strongly cautioned them that they were taking advantage of an undocumented and completely unsupported compiler feature. **We make no guarantee that the compiler's behaviour is repeatable in this way.** More generally: **the C\# compiler was not designed to be a component of a security system, so don't use it as one.**

-----

(\*) And a string is of course a finite, ordered sequence of characters drawn from some alphabet.

(\*\*) What precisely makes a program in one language "equivalent" to a program in another language is a deep question that we're going to completely ignore.

(\*\*\*) The details are quite interesting; basically we start by walking each file, one after the other, and build up a tree of "top level" items: namespaces, types, methods, and so on. Once we have that tree, we iterate over it in a tree traversal that effectively does a partial order sort. The goal is to find an ordering, if one exists, that guarantees that the metadata emitting engine never has to emit metadata "out of order"; we prefer base type metadata to be emitted before derived type metadata and require that outer type metadata be emitted before nested type metadata. There are unusual code topologies in which our preferences cannot be met; in those topologies, the emitter can have poor performance. We therefore have a vested interest in ensuring that we find a good ordering to enumerate over all the types of a program.

