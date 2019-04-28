# How Do Script Engines Implement Object Identity?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/26/2005 2:49:00 PM

-----

I've talked a few times in this blog about the semantics of the equality operators in various languages. (Such as [here](http://blogs.msdn.com/ericlippert/archive/2004/07/26/197302.aspx), and [here](http://blogs.msdn.com/ericlippert/archive/2004/07/30/202432.aspx).) Recently a reader asked me how JScript *implements* object identity. That is, given two objects, how do we know if they are "the same object" or not? For the remainder of this discussion I'm going to assume that we've got two non-null objects that we're comparing for identity. The details of how objects are compared to strings, numbers, etc, will have to wait for another blog entry. Note also that in VBScript, the comparison operator for objects is Is, not =. Also, == and === in JScript behave identically if both arguments are objects; essentially the implementation of Is, == and === are all the same as far as object comparison goes. Behind the scenes, everything in a VBScript/JScript program is represented as a variant. Most of the time, variants containing objects will have their variant type field set to VT\_DISPATCH, and their pdispVal field set with a pointer to the object. (Occasionally we'll have a VT\_UNKNOWN with a punkVal field instead, but lets ignore that corner case for now.) Suppose we want to compare two such objects for equality. We could pass the pdispVal of each to a helper function and compare the pointers. bool AreObjectsEqual(IDispatch \* pdisp1, IDispatch \* pdisp2)  
{  
  if (pdisp1 == pdisp2)  
    return true;  
  return false;  
} And we're done. Wow, that was easy. Except that it's wrong. In VBScript, those objects could be two different dispatch pointers to the same object because [VBScript supports non-default dispatches](http://blogs.msdn.com/ericlippert/archive/2003/10/10/53188.aspx). JScript does not support non-default dispatches, but in JScript there are also situations in which you can have two different dispatch pointers to the same object; we'll come to that later. If its not clear why the two pointers to the same object could be numerically different, read [Raymond's article on how COM objects are laid out in memory](http://blogs.msdn.com/oldnewthing/archive/2004/02/05/68017.aspx), and you'll see why. There can be many dispatch vtables. How can we ever determine when two COM objects are the same? We have to rely upon one of the most important rules of COM: if an object multiply inherits IUnknown then calling QueryInterface for IUnknown on *any* of those implementations must *always* give back the same pointer. As you can see in Raymond's example of multiple inheritance, there are two IUnknown vtables. It is illegal for each of them to return a pointer to itself when QI'd for IUnknown; the implementation must consistently pick one of them every time. So now it's pretty clear what we must do: bool AreObjectsEqual(IDispatch \* pdisp1, IDispatch \* pdisp2)  
{  
  IUnknown \* punk1 = NULL;  
  IUnknown \* punk2 = NULL;  
  if (pdisp1 == pdisp2)  
    return true;  
  if (pdisp1 == NULL || pdisp2 == NULL)  
    return false;  
  pdisp1-\>QueryInterface(IID\_IUnknown, (void\*\*)\&punk1);  
  pdisp2-\>QueryInterface(IID\_IUnknown, (void\*\*)\&punk2);  
  // This should never fail, but better safe than sorry.  
  if (punk1 \!= NULL)  
    punk1-\>Release();  
  if (punk2 \!= NULL)  
    punk2-\>Release();  
  // We're not dereferencing the pointers, so it is OK to use the  
  // locals even after their release.  
  if (punk1 == punk2)  
    return true;  
  return false;  
} Unfortunately, that's not the whole story either. For security reasons, Internet Explorer sometimes creates proxy objects -- little objects that detect if they are being used safely, and if they are, then they forward the call to the real implementation, on a separate object. You can run into situations in IE where you have two proxy objects to the same "real" object -- as far as COM is concerned, all three are different objects, but as far as IE is concerned, that's an implementation detail of the security system which we would like to hide from the script users. To solve this problem, IE implements a special interface that knows when two IUnknown objects are both proxies for the same object. The script engines actually do something like this: bool AreObjectsEqual(IDispatch \* pdisp1, IDispatch \* pdisp2) {  
  IUnknown \* punk1 = NULL;  
  IUnknown \* punk2 = NULL;  
  IObjectIdentity \* pObjectIdentity;  
  bool fRet = false;  
  HRESULT hr;  
  if (pdisp1 == pdisp2)  
    fRet = true;  
  else if (pdisp1 \!= NULL && pdisp2 \!= NULL) {  
    // This must always succeed.  
    pdisp1-\>QueryInterface(IID\_IUnknown, (void\*\*)\&punk1);  
    pdisp2-\>QueryInterface(IID\_IUnknown, (void\*\*)\&punk2);  
    if (punk1 == punk2)  
      fRet = true;  
    else if (punk1 \!= NULL && punk2 \!= NULL) { // This should never be false.  
      hr = punk1-\>QueryInterface(IID\_IObjectIdentity, (void \*\*)\&pObjectIdentity);  
      if (SUCCEEDED(hr)) && pObjectIdentity \!= NULL) {  
        hr = pObjectIdentity-\>IsEqualObject(punk2);  
        if (hr == S\_OK)  
          fRet = true;  
      }  
    }  
  }  
  if (pObjectIdentity \!= NULL)  
    pObjectIdentity-\>Release();  
  if (punk1 \!= NULL)  
    punk1-\>Release();  
  if (punk2 \!= NULL)  
    punk2-\>Release();  
  return fRet;  
} Which is a heck of a lot of code just to determine if two pointers are equal, but that's the tax you pay for object-oriented programming. Of course, you don't need to do any of this rigamarole with IObjectIdentity unless you're writing code that messes around with IE objects.

