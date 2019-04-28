# Old school tree display

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/9/2010 6:58:00 AM

-----

I'm back from my various travels, refreshed and ready for more fabulous adventures in coding. A while back I did a coding challenge for you all: to [turn a sequence of strings into a fancy comma-separated list](http://blogs.msdn.com/b/ericlippert/archive/2009/04/15/comma-quibbling.aspx). You might also recall that I did a bit on [how to generate all possible arbitrary trees](http://blogs.msdn.com/b/ericlippert/archive/tags/tree+generation/), which I notated with a simple bracing format. Today, let's combine those two problems.

What I want to do is create a function which takes an arbitrary tree where each node has some string data, and turns it into a fancy string that has some nice properties. It should be compact and easy to read; the structure of the tree should be apparent from the output string. I'd like to have one node per line, and each line ends with the data in the string (which we can think of as the name of the node.)

Here's my Node class:

 

class Node  
{  
    public Node(string text, params Node\[\] children)  
    {  
        this.Text = text;  
        this.Children = children ?? new Node\[\] {};  
    }  
    public string Text { get; private set; }  
    public IList\<Node\> Children { get; private set; }  
}

Pretty straightforward. Note that there are no parent pointers.

My challenge to you is to fill in the blanks here:

 

sealed class Dumper  
{  
    static public string Dump(Node root) { /\* ... \*/ }  
    /\* ... \*/  
}

such that the Dump method produces this string:

 

a  
├─b  
│ ├─c  
│ │ └─d  
│ └─e  
│   └─f  
└─g  
  ├─h  
  │ └─i  
  └─j

when given this tree:

 

var root = new Node("a",  
    new Node("b",  
        new Node("c",  
            new Node("d")),  
        new Node("e",  
            new Node("f"))),  
    new Node("g",      
        new Node("h",  
            new Node("i")),  
        new Node("j")));

Notice that I am using the Unicode "box drawing" characters │ ├ ─ └. I used to write code to build user interfaces like this back in my Commodore 64 programming days. Ah, the halcyon days of my youth.

I've posted my solution [here](http://blogs.msdn.com/b/ericlippert/archive/2010/09/09/eric-s-solution-for-old-school-tree-dumping.aspx), but NO CHEATING. Write your own solution first, and then see how yours compares to mine.

When I discussed how to build a [graph colourizer](http://blogs.msdn.com/b/ericlippert/archive/tags/graph+colouring/) / Sudoku solver in July I did some analysis of the design decisions I made along the way. What are your design criteria as you approach this problem? For example, are you going to automatically go for a recursive solution because tree problems are usually most easily solved with recursion? Or are you going for an iterative solution? Will you prioritize obvious correctness over cleverness or vice versa? Did you miss having parent pointers? And so on - I'd be interested to know what your design criteria and choices were.

And... go\!

