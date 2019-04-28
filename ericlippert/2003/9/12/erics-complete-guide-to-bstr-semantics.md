<div id="page">

# Eric's Complete Guide To BSTR Semantics

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/12/2003 3:28:00 PM

-----

<div id="content">

<span> </span>

<span>If you've ever done any C++ or C programming that used COM objects, you'll certainly have seen code like this: </span>

<span></span>

<span>STDMETHODIMP CFo:Bar(BSTR bstrABC) { // ... </span>

<span></span>

<span>What is this </span><span>BSTR</span><span> thing, and how does it differ from </span><span>WCHAR\*</span><span> ? </span>

<span></span>

<span>Low-level languages like C or C++ allow you great freedom in deciding which patterns of bits are used to represent certain concepts.  Unicode strings are an excellent example.  The standard way to represent an n-character Unicode string in C++ is as a pointer to a 2 x (n + 1) byte buffer where the first 2 x n bytes are unsigned short integers representing the characters and the final two bytes in the buffer are zeros, terminating the string. </span>

<span></span>

<span>For notational convenience we shall take a page from Hungarian notation and call such a beast a </span><span>PWSZ</span><span>, short for "Pointer to Wide-character String, Zero-terminated".  As far as the C++ type system is concerned, a </span><span>PWSZ</span><span> is an </span><span>unsigned short \*</span><span>. </span>

<span></span>

<span>COM uses a somewhat different approach to storing string data, an approach which is sufficiently similar to allow good interoperability between code expecting </span><span>PWSZ</span><span>s and code providing COM strings. Unfortunately they are sufficiently different that the subtle differences can cause nasty bugs if you are not careful and cognizant of those differences. </span>

<span></span>

<span>COM code uses the </span><span>BSTR</span><span> to store a Unicode string, short for "Basic String". (So called because this method of storing strings was developed for OLE Automation, which was at the time motivated by the development of the Visual Basic language engine.) </span>

<span></span>

<span>From the *<span>compiler's</span>* point of view a </span><span>BSTR</span><span> is also an </span><span>unsigned short \*</span><span>.  The compiler will not care if you use </span><span>BSTR</span><span>s where </span><span>PWSZ</span><span>s are expected and vice-versa.  But that does not mean that you can do so without impunity\!  There would not be two names for the same thing if they were not in some way different; these two things are different in a number of ways.  </span>

<span></span>

<span>In *<span>most</span>* *<span>cases</span>* a </span><span>BSTR</span><span> may be treated as a </span><span>PWSZ</span><span>.  In *<span>almost no cases</span>* may a </span><span>PWSZ</span><span> be treated as a </span><span>BSTR</span><span>. </span>

<span></span>

<span>Let me list the differences first and then discuss each point in excruciating detail. </span>

<span></span>

<span>1) A </span><span>BSTR</span><span> must have identical semantics for </span><span>NULL</span><span> and for </span><span>""</span><span>.  A </span><span>PWSZ</span><span> frequently has different semantics for those. </span>

<span>2) A </span><span>BSTR</span><span> must be allocated and freed with the </span><span>SysAlloc\*</span><span> family of functions.  A </span><span>PWSZ</span><span> can be an automatic-storage buffer from the stack or allocated with </span><span>malloc</span><span>, </span><span>new</span><span>, </span><span>LocalAlloc</span><span> or any other memory allocator. </span>

<span>3) A </span><span>BSTR</span><span> is of fixed length.  A </span><span>PWSZ</span><span> may be of any length, limited only by the amount of valid memory in its buffer. </span>

<span>4) A </span><span>BSTR</span><span> always points to the first valid character in the buffer.  A </span><span>PWSZ</span><span> may be a pointer to the middle or end of a string buffer. </span>

<span>5) When allocating an n-byte </span><span>BSTR</span><span> you have room for n/2 wide characters.  When you allocate n bytes for a </span><span>PWSZ</span><span> you can store n / 2 - 1 characters -- you have to leave room for the null. </span>

<span>6) A </span><span>BSTR</span><span> may contain any Unicode data including the zero character.  A </span><span>PWSZ</span><span> never contains the zero character except as an end-of-string marker.  Both a </span><span>BSTR</span><span> and a </span><span>PWSZ</span><span> always have a zero character after their last valid character, but in a </span><span>BSTR</span><span> *<span>a valid character may be a zero character</span>*. </span>

<span>7) A </span><span>BSTR</span><span> may actually contain an odd number of bytes -- it may be used for moving binary data around.  A </span><span>PWSZ</span><span> is almost always an even number of bytes and used only for storing Unicode strings. </span>

<span></span>

<span>Over the years I've found and fixed many bugs where the author assumed that a </span><span>PWSZ</span><span> could be used as a </span><span>BSTR</span><span> or vice-versa and thereby violated one of these differences.  Let's dig in to those differences: </span>

<span></span>

<span>1) If you write a function which takes an argument of type </span><span>BSTR</span><span> then you are required to accept </span><span>NULL</span><span> as a valid </span><span>BSTR</span><span> and treat it the same as a pointer to a zero-length </span><span>BSTR</span><span>.  COM uses this convention, as do both Visual Basic and VBScript, so if you want to play well with others you have to obey this convention.  If a string variable in VB happens to be an empty string then VB might pass it as </span><span>NULL</span><span> or as a zero-length buffer -- it is entirely dependent on the internal workings of the VB program. </span>

<span></span>

<span>That's not usually the case with </span><span>PWSZ</span><span>-based code.  Usually </span><span>NULL</span><span> is intended to mean "this string value is missing", not as a synonym for an empty string.  </span>

<span></span>

<span>In COM if you have some datum which could be a valid or could be missing then you should store it in a </span><span>VARIANT</span><span> and represent the missing value with </span><span>VT\_NULL</span><span> rather than interpreting a </span><span>NULL</span><span> string as different from an empty string. </span>

<span></span>

<span>2) </span><span>BSTR</span><span>s are always allocated and freed with </span><span>SysAllocString</span><span>, </span><span>SysAllocStringLen</span><span>, </span><span>SysFreeString</span><span> and so on.  The underlying memory is cached by the operating system and it is a serious, heap-corrupting error to call </span><span>free</span><span> or </span><span>delete</span><span> on a </span><span>BSTR</span><span>.  Similarly it is also an error to allocate a buffer with </span><span>malloc</span><span> or </span><span>new</span><span> and cast it to a </span><span>BSTR</span><span>.  Internal operating system code makes assumptions about the layout in memory of a </span><span>BSTR</span><span> which you should not attempt to simulate.  </span>

<span></span>

<span>PWSZ</span><span>s on the other hand can be allocated with any allocator or allocated off the stack.  </span>

<span></span>

<span>3) The number of characters in a </span><span>BSTR</span><span> is fixed.  A ten-byte </span><span>BSTR</span><span> contains five Unicode characters, end of story.  Even if those characters are all zeros, it still contains five characters.  A </span><span>PWSZ</span><span> on the other hand can contain fewer characters than its buffer allows: </span>

<span></span>

<span>WCHAR pwszBuf\[101\];  
</span><span>pwszBuf\[0\] = 'X';  
</span><span>pwszBuf\[1\] = ' </span>

<span></span>

<span>pwszBuf</span><span> is a one-character string which may be lengthened to up to a 100 character string or shrunk to a zero-character string. </span>

<span></span>

<span>4)         A </span><span>BSTR</span><span> always points to the first valid character in the buffer.  This is not legal: </span>

<span></span>

<span>BSTR bstrName = SysAllocString(L"John Doe");  
</span><span>BSTR bstrLast = \&bstrName\[5\]; // ERROR </span>

<span></span>

<span>bstrLast </span><span>is not a legal </span><span>BSTR</span><span>.  That is perfectly legal with </span><span>PWSZ</span><span>s though: </span>

<span></span>

<span>WCHAR \* pwszName = L"John Doe";  
</span><span>WCHAR \* pwszLast = \&pwszName\[5\]; </span>

<span></span>

<span>5) and 6) The reasons for the above restrictions make more sense when you understand how exactly a </span><span>BSTR</span><span> is really laid out in memory, and this also explains why allocating an n-character </span><span>BSTR</span><span> gives you room for n characters, not n-1 like a </span><span>PWSZ</span><span> allocator. </span>

<span></span>

<span>When you call </span><span>SysAllocString(L"ABCDE")</span><span> the operating system actually allocates sixteen bytes.  The first four bytes are a 32 bit integer representing the number of valid bytes in the string -- initialized to ten in this case.  The next ten bytes belong to the caller and are filled in with the data passed in to the allocator.  The final two bytes are filled in with zeros. You are then given a pointer to the data, not to the header. </span>

<span></span>

<span>This immediately explains a few things about </span><span>BSTR</span><span>s: </span>

<span></span>

  - <span>The length can be determined immediately.  </span><span>SysStringLen</span><span> does not have to count bytes looking for a null like </span><span>wcslen</span><span> does.  It just looks at the integer preceding the pointer and gives you that value back.</span>

  - <span></span><span>That's why it is illegal to have a </span><span>BSTR</span><span> which points to the middle of another </span><span>BSTR</span><span>.  The length field would not be before the pointer. </span>

  - 
<span>A </span><span>BSTR</span><span> can be treated as a </span><span>PWSZ</span><span> because there is always a trailing zero put there by the allocator.  You, the caller, do not have to worry about allocating enough space for the trailing zero.  If you need a five-character string, ask for five characters. </span>

<span>That's why a </span><span>BSTR</span><span> must be allocated and freed by the </span><span>Sys\*</span><span> functions.  Those functions understand all the conventions used behind-the-scenes. </span>

<span>7) Because a </span><span>BSTR </span><span>is of a known number of bytes there is no need for the convention that a zero terminates a string.  Therefore zero is a legal value inside a </span><span>BSTR</span><span>.  This means that </span><span>BSTR</span><span>s can contain arbitrary data, including binary images.  For this reason </span><span>BSTR</span><span>s are often used as a convenient way to marshal binary data around in addition to strings.  This means that </span><span>BSTR</span><span>s may be, in some odd situations, an odd number of bytes.  It is rare, but you should be aware of the possibility. </span>

<span></span>

<span></span>

<span>Whew\!  To sum up, that should explain why a </span><span>BSTR</span><span> may usually be treated as a </span><span>PWSZ</span><span> but a </span><span>PWSZ</span><span> may not be treated as a </span><span>BSTR</span><span> unless it really is one.  The only situations in which a </span><span>BSTR</span><span> may not be used as a </span><span>PWSZ</span><span> are (a) when the </span><span>BSTR</span><span> is </span><span>NULL</span><span> and (b) when the </span><span>BSTR</span><span> contains embedded zero characters, because the </span><span>PWSZ</span><span> code will think the string is shorter than it really is and (c) the </span><span>BSTR</span><span> does not in fact contain a string but rather arbitrary binary data.  The only situation in which a </span><span>PWSZ</span><span> may be treated as a </span><span>BSTR</span><span> are when the </span><span>PWSZ</span><span> actually is a </span><span>BSTR</span><span>, allocated with the right allocator. </span>

<span></span>

<span>In my own C++ code I avoid misunderstandings by making extremely careful use of Hungarian Notation to keep track of what is pointing to what.  Hungarian Notation works best when it captures semantic information about the variables which is obscured by the type signature.  I use the following conventions: </span>

<span></span>

<span>bstr</span><span> --\> a real </span><span>BSTR</span><span>  
</span><span>pwsz</span><span> --\> a pointer to a zero-terminated wide character string buffer  
</span><span>psz </span><span> --\> a pointer to a zero-terminated narrow character string buffer  
</span><span>ch  </span><span> --\> a character  
</span><span>pch </span><span> --\> a pointer to a wide character  
</span><span>cch </span><span> --\> a count of characters  
</span><span>b   </span><span> --\> a byte  
</span><span>pb  </span><span> --\> a pointer to a byte  
</span><span>cb  </span><span> --\> a count of bytes </span>

</div>

</div>

