# Eric's Complete Guide To BSTR Semantics

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/12/2003 3:28:00 PM

-----

 

If you've ever done any C++ or C programming that used COM objects, you'll certainly have seen code like this: 

STDMETHODIMP CFo:Bar(BSTR bstrABC) { // ... 

What is this BSTR thing, and how does it differ from WCHAR\* ? 

Low-level languages like C or C++ allow you great freedom in deciding which patterns of bits are used to represent certain concepts.  Unicode strings are an excellent example.  The standard way to represent an n-character Unicode string in C++ is as a pointer to a 2 x (n + 1) byte buffer where the first 2 x n bytes are unsigned short integers representing the characters and the final two bytes in the buffer are zeros, terminating the string. 

For notational convenience we shall take a page from Hungarian notation and call such a beast a PWSZ, short for "Pointer to Wide-character String, Zero-terminated".  As far as the C++ type system is concerned, a PWSZ is an unsigned short \*. 

COM uses a somewhat different approach to storing string data, an approach which is sufficiently similar to allow good interoperability between code expecting PWSZs and code providing COM strings. Unfortunately they are sufficiently different that the subtle differences can cause nasty bugs if you are not careful and cognizant of those differences. 

COM code uses the BSTR to store a Unicode string, short for "Basic String". (So called because this method of storing strings was developed for OLE Automation, which was at the time motivated by the development of the Visual Basic language engine.) 

From the *compiler's* point of view a BSTR is also an unsigned short \*.  The compiler will not care if you use BSTRs where PWSZs are expected and vice-versa.  But that does not mean that you can do so without impunity\!  There would not be two names for the same thing if they were not in some way different; these two things are different in a number of ways.  

In *most* *cases* a BSTR may be treated as a PWSZ.  In *almost no cases* may a PWSZ be treated as a BSTR. 

Let me list the differences first and then discuss each point in excruciating detail. 

1) A BSTR must have identical semantics for NULL and for "".  A PWSZ frequently has different semantics for those. 

2) A BSTR must be allocated and freed with the SysAlloc\* family of functions.  A PWSZ can be an automatic-storage buffer from the stack or allocated with malloc, new, LocalAlloc or any other memory allocator. 

3) A BSTR is of fixed length.  A PWSZ may be of any length, limited only by the amount of valid memory in its buffer. 

4) A BSTR always points to the first valid character in the buffer.  A PWSZ may be a pointer to the middle or end of a string buffer. 

5) When allocating an n-byte BSTR you have room for n/2 wide characters.  When you allocate n bytes for a PWSZ you can store n / 2 - 1 characters -- you have to leave room for the null. 

6) A BSTR may contain any Unicode data including the zero character.  A PWSZ never contains the zero character except as an end-of-string marker.  Both a BSTR and a PWSZ always have a zero character after their last valid character, but in a BSTR *a valid character may be a zero character*. 

7) A BSTR may actually contain an odd number of bytes -- it may be used for moving binary data around.  A PWSZ is almost always an even number of bytes and used only for storing Unicode strings. 

Over the years I've found and fixed many bugs where the author assumed that a PWSZ could be used as a BSTR or vice-versa and thereby violated one of these differences.  Let's dig in to those differences: 

1) If you write a function which takes an argument of type BSTR then you are required to accept NULL as a valid BSTR and treat it the same as a pointer to a zero-length BSTR.  COM uses this convention, as do both Visual Basic and VBScript, so if you want to play well with others you have to obey this convention.  If a string variable in VB happens to be an empty string then VB might pass it as NULL or as a zero-length buffer -- it is entirely dependent on the internal workings of the VB program. 

That's not usually the case with PWSZ-based code.  Usually NULL is intended to mean "this string value is missing", not as a synonym for an empty string.  

In COM if you have some datum which could be a valid or could be missing then you should store it in a VARIANT and represent the missing value with VT\_NULL rather than interpreting a NULL string as different from an empty string. 

2) BSTRs are always allocated and freed with SysAllocString, SysAllocStringLen, SysFreeString and so on.  The underlying memory is cached by the operating system and it is a serious, heap-corrupting error to call free or delete on a BSTR.  Similarly it is also an error to allocate a buffer with malloc or new and cast it to a BSTR.  Internal operating system code makes assumptions about the layout in memory of a BSTR which you should not attempt to simulate.  

PWSZs on the other hand can be allocated with any allocator or allocated off the stack.  

3) The number of characters in a BSTR is fixed.  A ten-byte BSTR contains five Unicode characters, end of story.  Even if those characters are all zeros, it still contains five characters.  A PWSZ on the other hand can contain fewer characters than its buffer allows: 

WCHAR pwszBuf\[101\];  
pwszBuf\[0\] = 'X';  
pwszBuf\[1\] = ' 

pwszBuf is a one-character string which may be lengthened to up to a 100 character string or shrunk to a zero-character string. 

4)         A BSTR always points to the first valid character in the buffer.  This is not legal: 

BSTR bstrName = SysAllocString(L"John Doe");  
BSTR bstrLast = \&bstrName\[5\]; // ERROR 

bstrLast is not a legal BSTR.  That is perfectly legal with PWSZs though: 

WCHAR \* pwszName = L"John Doe";  
WCHAR \* pwszLast = \&pwszName\[5\]; 

5\) and 6) The reasons for the above restrictions make more sense when you understand how exactly a BSTR is really laid out in memory, and this also explains why allocating an n-character BSTR gives you room for n characters, not n-1 like a PWSZ allocator. 

When you call SysAllocString(L"ABCDE") the operating system actually allocates sixteen bytes.  The first four bytes are a 32 bit integer representing the number of valid bytes in the string -- initialized to ten in this case.  The next ten bytes belong to the caller and are filled in with the data passed in to the allocator.  The final two bytes are filled in with zeros. You are then given a pointer to the data, not to the header. 

This immediately explains a few things about BSTRs: 

  - The length can be determined immediately.  SysStringLen does not have to count bytes looking for a null like wcslen does.  It just looks at the integer preceding the pointer and gives you that value back.

  - That's why it is illegal to have a BSTR which points to the middle of another BSTR.  The length field would not be before the pointer. 

  - 
A BSTR can be treated as a PWSZ because there is always a trailing zero put there by the allocator.  You, the caller, do not have to worry about allocating enough space for the trailing zero.  If you need a five-character string, ask for five characters. 

That's why a BSTR must be allocated and freed by the Sys\* functions.  Those functions understand all the conventions used behind-the-scenes. 

7) Because a BSTR is of a known number of bytes there is no need for the convention that a zero terminates a string.  Therefore zero is a legal value inside a BSTR.  This means that BSTRs can contain arbitrary data, including binary images.  For this reason BSTRs are often used as a convenient way to marshal binary data around in addition to strings.  This means that BSTRs may be, in some odd situations, an odd number of bytes.  It is rare, but you should be aware of the possibility. 

Whew\!  To sum up, that should explain why a BSTR may usually be treated as a PWSZ but a PWSZ may not be treated as a BSTR unless it really is one.  The only situations in which a BSTR may not be used as a PWSZ are (a) when the BSTR is NULL and (b) when the BSTR contains embedded zero characters, because the PWSZ code will think the string is shorter than it really is and (c) the BSTR does not in fact contain a string but rather arbitrary binary data.  The only situation in which a PWSZ may be treated as a BSTR are when the PWSZ actually is a BSTR, allocated with the right allocator. 

In my own C++ code I avoid misunderstandings by making extremely careful use of Hungarian Notation to keep track of what is pointing to what.  Hungarian Notation works best when it captures semantic information about the variables which is obscured by the type signature.  I use the following conventions: 

bstr --\> a real BSTR  
pwsz --\> a pointer to a zero-terminated wide character string buffer  
psz  --\> a pointer to a zero-terminated narrow character string buffer  
ch   --\> a character  
pch  --\> a pointer to a wide character  
cch  --\> a count of characters  
b    --\> a byte  
pb   --\> a pointer to a byte  
cb   --\> a count of bytes

