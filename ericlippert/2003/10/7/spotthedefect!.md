# Spot the Defect\!

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/7/2003 1:12:00 PM

-----

 

At Microsoft we have an internal email list called "Spot the Defect" -- people mail around buggy code they've discovered and we compete to see who can find the most problems with it.  It's fun, and you learn a lot about what other people consider bugs -- everything from security holes to lying comments\!

 

 

I love playing Spot the Defect. Here is the code for the WScript.Sleep method with the comments removed **and some serious bugs added**.  You'll note that this code has all the required features I mentioned in my previous post.  We go to sleep in one-second (or less) intervals, and tell the operating system to wake us up if COM posts a message to the message queue, because there might be an event handler to dispatch.  We also check to see if the host recorded a script error (either due to an event handler or due to the script timeout firing) so that we can abort the sleep.  This way we never keep the script alive more than a second after it was shut down due to error.

 

 

What bug did I add?

 

 

HRESULT CWScript::Sleep(long Time)

{

    const DWORD TimerGranularity = 1000;

    if (Time \< 0)

        return E\_INVALIDARG;

    DWORD StartTickCount = ::GetTickCount();

    DWORD EndTickCount = StartTickCount + Time;

    DWORD CurTickCount = StartTickCount;

    while(CurTickCount \< EndTickCount)

    {

        MSG msg;

        DWORD CurWaitTime = (DWORD)(EndTickCount - CurTickCount);

        if (CurWaitTime \> TimerGranularity)

            CurWaitTime = TimerGranularity;

        ::MsgWaitForMultipleObjects(0, NULL, TRUE, CurWaitTime, QS\_ALLINPUT | QS\_ALLPOSTMESSAGE);

        if (0 \!= ::PeekMessage(\&msg, NULL, 0, 0, PM\_REMOVE))

            ::DispatchMessage(\&msg);

        if (m\_pHost-\>FErrorPending())

            return S\_OK;

        CurTickCount = ::GetTickCount();

    }

    return S\_OK;

}

