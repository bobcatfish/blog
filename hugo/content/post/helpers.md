---
title: "Helpers are functions too!"
date: 2021-05-30
draft: false
---

_Let's not talk about the fact that I setup this blog MORE THAN THREE YEARS AGO and then did nothing. Since we're not talking about it, I'll move on._

I want to tell you about one of my big pet peeves (if you know me, or if I ever write more than 2 blog posts, you'll know that I have a LOT of pet peeves when it comes to software - sorry not sorry): when people talk about "helpers" in their code.

Some examples of where this comes up:

* Code review comment: "Why don't you move this test logic into a helper?"
* Pull request title: "Add some helpers"
* Conversation: "Yeah maybe we can just put that into a helper library"
* IN THE ACTUAL CODE ITSELF: `helpers/helper.go`

_How do YOU feel about this? When you read the above examples, what do you think? Maybe you think something like, "yeah, okay, those are totally normal and reasonable, where is this going." Just you wait!_

![internal suffering when someone talks about helpers](/helpers.png)

I did a quick google to see what other people think about helpers and I found [this question on stack overflow](https://softwareengineering.stackexchange.com/questions/247267/what-is-a-helper-is-it-a-design-pattern-is-it-an-algorithm). The title kinda cracks me up:

> What is a helper? Is it a design pattern? Is it an algorithm?

This is SUCH A TYPICAL THING IN SOFTWARE! People throwing around a term that no one actually knows how to define, assuming some kind of shared understanding that may or may not exist? Software do be like that. And if you are familiar with some of my other work, such as [spending a surprising amount of time trying to figure out what CI/CD means](https://github.com/cdfoundation/glossary/blob/main/definitions.md#cicd) you'll know that I love questions like this.

But back to stack overflow: kudos to Aaron Hall for actually asking the bold question "What is a helper?" I'M GLAD SOMEONE FINALLY ASKED! And the top answer is oh, oh so sweet \*chef's kiss\*:

> A Helper class is a lesser known code smell where a coder has identified some miscellaneous, commonly used operations and attempted to make them reusable by lumping them together in an unnatural grouping.

THANK YOU MR. COCHESE. You took the words right out of my mouth... but I'm still going to talk about it anyway.

I strongly disagree with some other answers here:

* "A helper is a harmless additional class or method" HARMLESS YOU SAY??? HARM-_FUL_ I SAY!
* "There is a subset of these, called "helper functions", that are actually useful and a good pattern." I  must disagree, sir, madam or zir. What is a helper function but a function that is being referred to as lesser?

_Side note: anyone else listen to [my brother, my brother and me](https://www.themcelroy.family/mbmbam)? How about this podcast idea: it's like MBMBAM, but stackoverflow... I think I might have something here..._

Okay so that's enough recycling Stack Overflow for content, here are my reasons for being
anti-helper:

1. [`helper` is not a descrpitive name](#helper-is-not-a-descriptive-name)
2. [`helper` perpetuates a derogatory attitude toward test code](#helper-perpetuates-a-derogatory-attitude-toward-test-code) _wow now I'm regretting the length that title cuz that was LONG to type_

And in conclusion: [call them functions, treat them right](#conclusion-call-them-functions-and-treat-them-right)

## `helper` is not a descriptive name

So off the bat, calling something a "helper" doesn't really _help_ (haha) me know what it is. (What I DO know
is that this is not code you think is very important, but that's the next point.)

I'm subscribed to the idea that **bad names are bad factoring**. That is, when something in code has a bad name, especially a vague name, this is a hint (a "smell" if you will) that something is wrong.

Golang completely won over my heart with [this blog post describing bad package names](https://blog.golang.org/package-names#TOC_5.). And I don't think I could say it better than they said it:

> Packages named `util`, `common`, or `misc` provide clients with no sense of what the package contains. This makes it harder for clients to use the package and makes it harder for maintainers to keep the package focused. Over time, they accumulate dependencies that can make compilation significantly and unnecessarily slower, especially in large programs.

When you use a name like `util` for a package, and then try to reason about what could go in that packge, well what code does something you'd say is a `util`? Well anything that's a utility. What ISN'T a utility?

And similarly, what ISN'T a helper? A helper helps. Any code that I use, any lib that I use, **helps** me.

_My OTHER least favorite name for something in code is "manager". Because, what does it mean to manage? How do you know what a manager should or shouldn't do? (I also happen to wonder this in the context of the role called "manager" but that's another blog post.)_

Anyway, especially after that blog post won my heart, everytime I see one of these vague names being used in code, I feel like a cat being pet backward - and that's also how I feel when people call code "helpers".

### Et tu, Golang?

Now, if you use Golang, you might be chuckling to yourself as you read the above - because I'm saying Golang has these great recommendations about naming things well, and I'm like THEREFORE DON'T CALL THINGS HELPER but you may know that actually Golang ITSELF has a function in its STANDARD TESTING PACKAGE NO LESS (see the next section) called [`Helper`](https://golang.org/pkg/testing/#B.Helper). Why you gotta hurt my feelings like that Golang?

If you're curious you can [take a look at all the back and forth that led to the existence of this function](https://github.com/golang/go/issues/4899#issuecomment-66075252). It stretches from 2014 to 2017 which kinda hints to me at a situation where folks were not immediately sure about how to move forward, and then they eventually just kinda did _something_ - and from what I can tell, they didn't debate the name of the function at all. There's an interesting point at which [Rob Pike refers to testing helpers as "the devil's handiwork"](https://codereview.appspot.com/12405043#msg3). Unfortunately [he didn't elaborate](https://github.com/golang/go/issues/4899#issuecomment-66075253), BUT I can't help but think that he's recognizing that there's something odd about the whole feature.

When you use `t.Helper()` in a function, the line number for assertion errors wihtin that function are reported as the caller instead of the line within the function. So I'm just gonna say, that's awful weird. I've definitely used it myself - and sometimes it makes for a more informative test failure, but like: ultimately I need to go find where the failure ACTUALLY is, which is being intentionally hidden by this function.

In the words of [Rob Pike](https://github.com/golang/go/issues/4899#issuecomment-66075246) (who seems to have eventually come around):

> Still not convinced this is a good idea.

The last thing I'll say about this is it would have been nice if the function could have been named for what it actually does. What does it do? It hides file and line information. Maybe it could have been called `t.HideLineNumber` or `t.PretendImNotImportantEvenThoImCodeJustLikeEverythingElse`.

## `helper` perpetuates a derogatory attitude toward test code

"But Christie," I hear you say. "I didn't call it that in the code! I just _referred_ to it as a helper."

Well I have a question back at you: "WHY did you call it a helper, though?" Is it not because you felt this code was somehow "less than" other code? You didn't want to call it a library, or a function, or a package, you felt this needed a different name.

IS IT NOT BECAUSE IT IS CODE BEING USED BY TESTS???? (*cough* GOLANG *cough*)

Okay definitely sometimes people say "helper" when they're talking about code outside of tests (or apparently sometimes, entire repositories on GitHub), but I swear in my experience at lesat that 9/10 times, when someone says "helper" it's because they're talking about code they want to call from their tests.

Why do libraries we use in our test code need a special name? Why not call them: libraries?

I suspect there are 2 reasons:

1. We inherently think of our test code as "lesser"
2. If we called it a "library" we might feel we have to start treating it like a library

If we called these functions what they are - functions - and put them into carefully factored libraries, we might start treating them like libraries, including documenting them and even - wait for it - EVEN WRITING TESTS FOR THEM.

But who wants to do that - write tests for test code - you're talking crazy Christie!

I don't think I am though! Test code is CODE. It's code we need to maintain and interact with for however many years into the future out project is going to be active for (hopefully decades even!). So I think it's totally reasonable, nay, DESIREABLE to maintain the LIBRARIES (repeat after me: libraries you write for tests are still libraries) that you write to support your tests.

That would be a lot more work than just calling these functions "helpers" though. Cuz if something is just a helper, then who cares what the function signature is? Who cares how reusable it is? Who cares if it has docs, or tests....

So: what I'm saying is: calling functions we write for tests "helpers" allows us to treat them as lesser - which is easier in the short term but a maintainablity nightmare for everyone who has to deal with them later. Remember: we don't optimize for how quickly we can write the code NOW, we want to optimize for the long haul.

## Conclusion: Call them functions, and treat them right

In the words of [what appears to be the question asker answering their own question](https://softwareengineering.stackexchange.com/a/414543) (wait was this some kind of setup...):

> Please don't call anything a "helper."

Call it what it is: a function, which is code, that you're gonna need to maintain just like all the rest of your code. Call it a function and treat it just as well as you'd tread the rest of your code. The person who has to maintain the tests you're writing 10 years from now will thank you. Or at least, not silently scream your name as they look at the blame for your "helper". I swear I've never done that... 🤞
