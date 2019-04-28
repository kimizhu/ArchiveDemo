# Refreshing the Async CTP

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/13/2011 9:00:00 AM

-----

Good morning everyone\! I am pleased to tell you that the C\# and VB teams are announcing a "refresh" of the async Community Technology Preview at [MIX11](http://live.visitmix.com/mix11) today, and that it is as of right now [available on the Async CTP site](http://msdn.microsoft.com/en-us/vstudio/async.aspx).

Recall that the CTP release is an early look at our thinking for the [proposed async language features](http://blogs.msdn.com/b/ericlippert/archive/tags/async/) so that we can get your feedback. Rather than posting feedback here, [please let us know what you think on the Async Forum](http://social.msdn.microsoft.com/Forums/en-US/async/threads).

We've gotten a lot of good feedback so far; this refresh is intended to address the top ten-or-so concerns voiced so far about the CTP, namely:

  - The async feature can now be used for **Windows Phone 7 development**. Woo hoo\!
  - The new CTP works with [Visual Studio 2010 Service Pack 1](http://www.microsoft.com/downloads/en/details.aspx?FamilyID=75568aa6-8107-475d-948a-ef22627e57a5) and with **non-English-localized** versions. (Note that the CTP itself has not been localized into any language other than English, but it should now work with non-English versions of VS2010.)
  - The samples show some of our thinking on how **unit testing** might work with the async feature.
  - The license is now an "as is" EULA; that is to say that **you are permitted to use this technology in real live deployed apps if you choose to do so at your own risk.** Note that (1) I am not a lawyer, so don't ask me for legal advice about the EULA and (2) I am not *recommending* that you go live with this\! This is still a very early build of this technology. But if you and your customers like living on that leading edge, you can do so.
  - There are a number of new features in **System.Threading.Tasks.Dataflow**.dll.
  - **AsyncCtpLibrary\_Phone**.dll and **AsyncCtpLibrary\_Silverlight**.dll now have **full support for tasks**.
  - **Unhandled exceptions in async void methods** are now posted to the current synchronization context.
  - We've made some **changes to the "awaiter" pattern that enable better performance**.
  - The state machine transformation now aggressively nulls out fields no longer in use, enabling **more efficient garbage collection**.
  - A number of bugs involving **race conditions with finally blocks,** **other** **code generation errors, library code and the IDE** have been fixed.

Watch [Lucian's blog](http://blogs.msdn.com/b/lucian/) and the Async Forum for more details on precisely what the changes outlined above are. Also, Mads and Alex have just posted interviews on Channel 9 giving an overview of the various changes in video form. Their videos are [here](https://channel9.msdn.com/posts/Mads-Torgersen-Visual-Studio-Async-CTP-SP1-Refresh-Overview) and [here](https://channel9.msdn.com/posts/Alex-Turner-Visual-Studio-Async-CTP-SP1-Refresh-WP7-Demo).

Some tips I've heard on getting it installed from those in the know:

  - Uninstall previous versions of the CTP.
  - If you're doing phone development, install the phone developer tools **before** you install SP1.
  - Install SP1 before installing the CTP refresh.
  - If you install a "hotfix" after SP1, it might not be compatible with the CTP.

I encourage you to check it out and post any thoughts or questions you might have to the forum. Thanks\!

And as always, thanks to my colleagues Mads, Alex and Lucian for their leadership in this feature area, and for providing this list of changes.

