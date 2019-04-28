# How do I mitigate a SQL injection vuln?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/1/2006 4:32:00 PM

-----

Joel [points out today](http://www.joelonsoftware.com/items/2006/11/01.html) that SQL injection vulnerabilities are common and bad, bad, bad. He does a good job of describing the attack but doesn't really talk about how to mitigate it.

When I advise people on how to close security holes like this I always tell them that **closing the original hole is probably not enough**. You don't want to make the attackers do one "impossible" thing because you just might be mistaken about what is actually impossible. **Make them do three or four impossible things**, and odds are good that it really will be impossible to cause harm.

To start with, **run all strings that come from users through a string checker** looking for anything out of the ordinary. If you expect a string to contain only letters, numbers and spaces, then write a regular expression that verifies that and get in the habit of rejecting all input that doesn't conform. That should make it impossible for attackers to put special characters like quotation marks in there.

Assume that a clever attacker will subvert that. **Assume that you've made a mistake** and forgotten to put a check in somewhere. Look for every place in the code that uses that user-supplied string. Don't stop at SQL construction. Anything that gets passed to JScript's **eval** could be an injection attack. **Anything that gets echoed back to the user could be a cross-site scripting attack**. Anything that gets **written to disk** could be an attempt to write a script onto the server's disk to trick an admin into running it. Eliminate as many of these as you can.

But how do you eliminate them? A great way to mitigate the risk of a SQL injection attack is to **use stored procedures**. Stored procedures ensure that only the query that you want to run actually runs. But they have nice properties in addition to being more secure against injection. They can be updated in the database, so that when the database structure changes, you change the stored procedure rather than searching through your code for SQL statement construction. And stored procedures often run faster because the database can optimize itself for them.

How else can we make it impossible for an attacker to run Joel's proposed attack? The attack Joel describes deletes stuff from the database, which is pretty unsubtle. A more subtle attack would consist of an update query which ups the user's available credit, or changes the prices of products, or some such thing. The database code should make use of the **principle of least privilege**. If you only expect it to look up results from particular database tables, then create a database account that only has those permissions, and then use that account from the server code. **Don't connect to the database as admin if you don't need admin privileges**; that's just asking for someone to abuse those privileges.

Furthermore, **don't keep secrets in source code.** For instance, put the name of the database server, the name of the database account, and the password in a registry key or an ACL'd file. Assume that attackers will obtain your source code. Keep machine names, employee names, **keep anything sensitive that an attacker could use out of the source code.**

How did Joel find out that the server was vulnerable at all? When the server helpfully told him that there was malformed SQL, and furthermore, what part of it was. Not only is this potentially a cross-site scripting attack (because user-supplied data is echoed back) but it is like waving a big sign explaining exactly how to construct an attack\! Detailed error messages that describe the internal state of the server should only be given to users who have been authenticated by the server and are known to be server developers. **Ordinary users should get a generic "something bad happened" error that explains nothing.**

For Joel's proposed attack to succeed, *everything* has to go wrong. The server has to fail to validate input, then use it in an insecure way, then connect to the database as an administrator. Regrettably, many server-side web apps leave themselves wide open to these sorts of attacks. **Eliminate all of these problems**, not just the string concatenation. Remember, think like an evil person, assume the worst, and make evildoers jump through as many hoops as you possibly can. Defense in depth\!

