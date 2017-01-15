Or, if I had to give a more sincere title, this would be *How to delete 6 servers in one move*. 

One of the first programming books I ever read, a book that ? an illuminating experience for me and I highly recommend to every developer, was [*The Pragmatic Programmer*](https://pragprog.com/book/tpp/the-pragmatic-programmer). Inside its pages you can find lots of good and bad practices, @things that may appear familiar to the skilled dev and ? to the younger one@.

One of the book chapters, declaratively states that you should never program by coincidence. This means that you should always know how your code actually works and not make assumptions about it, or simply copy and paste code you find in legacy programs or QnA sites. Pretty clear, right? Besides, why would you rush to use a line of code you do not fully understant, when this can be a motive to explore a new technology?

Well, maybe because real life is not always like this. You will always have deadlines running, always a lot of things to do and always a line that seems to work just fine will be tempting.

Last week, I was writing a script that would copy TLS CSRs from 6 SaltStack *minion* nodes to their Salt master, and the master would then send them to a private CA, have them signed and send the certificates back to the corresponding nodes. So, I created a temporal folder, that I would delete after the file copying was over. Still, pretty cool. But the thing is that I am not yet that good in Bash scripting, so I googled how I could automate the procedure and concluded to something like this:

```
catastrohic code here
```

![Coding Horror](../img/coding-horror.png)

Analyze why this is a mistake.

Quote jeff atwood: 

> You're an amateur developer until you realize that everything you write sucks. YOU are the Coding Horror.

Lessons learned:

* avoid programming by coincidence even if you are in a hurry; if something catastrophic happens the time you will spend to recover will be much more.
* similarly do not forget all best practices (e.g. for this occasion: be careful who has root privileges, use interactive rm, test before changes etc)
* be extremely careful when you are about to remove something!

Of course I was completely disapointed by the silly mistake I made, but once the damage was revoked and some time passed, I felt really good that I work in an environment that does not discourage me to occasionally make some mistakes. :)


