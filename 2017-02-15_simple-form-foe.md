# Simple ~~Form~~ Foe

Let me start by saying that I love the idea of `simple_form`. An easy way to generate forms with a standard - and automated - way of displaying errors when they occur and just generally give your whole site a consistent look and feel with very little overhead.

I have come to realise that it may not be the friend it is pretending to be. On my latest project, I have been looking to migrate to using `simple_form` as I want to give my user better feedback on my forms around validation failures and it felt like the best option to achieve this.

For the vanilla forms/field this certainly proved to be the case, once I moved onto the more bespoke fields it soon become clear that what I believed was a straightforward task, was in fact far more complicated, and would require me implementing custom field renders, and trying to understand how data is passed between the definition and final rendering stage.

I found the internals of `simple_form` hard to get your head around. I'm sure the architecture fits together easily once you understand it, but for me, it all feels a bit like black magic, with most of the example being provided for trivial use cases.

As such, I won't be using `simple_form` on my current project. Not because I think it is bad software or unfit for purpose, I have chosen not to use it, as once I got it all working I would be fearful of needing future changes, and I feel this outweighs any benefit provided by the convenience of gem offers.

This may be due to the complex nature of my application, or it may just be `simple_form` should remain in the realms of vanilla forms.
