# He's So Dreamy

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/2/2012 9:23:00 AM

-----

Happy New Year all\!

It has just been brought to my attention that this blog and the [Programmer Ryan Gosling photo blog](http://programmerryangosling.tumblr.com) share at least one reader:

[![RyanGosling1](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/metablogapi/8468.RyanGosling1_3.jpg "RyanGosling1")](http://programmerryangosling.tumblr.com/post/14709857038)

I admit it, I LOL'd.

In the interests of total accuracy I'd like to point out that [the first entry on the blog contains a subtle error](http://programmerryangosling.tumblr.com/post/14706660907):

[![RyanGosling3](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/metablogapi/7418.RyanGosling3_3.jpg "RyanGosling3")](http://programmerryangosling.tumblr.com/post/14706660907)

.NET actually supported generic covariance and contravariance on interface and delegate types from the beginning; generics were introduced in version 2, and **they always allowed variance**. You could write programs in MSIL and compile them with ILDASM and have generic variance, no problem. No "mainstream" language supported the feature until v4, and none of the interfaces or delegates in the class libraries were marked as variant, so effectively variance did not become a well-used feature until v4, but the capability was always there.

As a response to the unknown reader to submitted the first photo above, I give you the following:

[![kinopoisk.ru](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/metablogapi/6675.RyanGosling2_thumb_1.jpg "kinopoisk.ru")](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/metablogapi/5621.RyanGosling2_4.jpg)

**Next time:** some depressing news about breaking changes: they're everywhere\!

