Refactoring
============

In our industry its often the case that two people are used loosely, incorectly.
So let's start with definition given by Martin Fowler.

# Definitions

Refactoring (noun): a change made to the internal structure of software to make it easier to understand and cheaper to modify without changing its observable behavior.

Refactoring (verb): to restructure software by applying a series of refactorings without changing its observable behavior.

It means that you can spend a couple of hours *refactoring*, applying *refactoring transformations*.
Accoring to Martin Fowler, refactoring means introducing big changes in code by applying serires of small steps to achieve that.

## What refactoring is not?
Sometimes there's a case where refactoring is started and we dig deeper and deeper. In result we end up with code that is not working.
There are different causes for that. One of them is lack of plan or scope. The other can be lack of tests.
So according to definition, this is not refactoring. The reason its not refactoring is that, refactoring assumes *no change in its observable behavior*.

## So what is refactoring?
It's a process of reorganizing and cleaning up the code. Perhaps preparing it for a new feature, in order to develop it faster.
I usually do that when I know I am about to introduce some feature in some area. Then I am trying to think how can I make it easier for myself to get desired effect?
How do I reorganize the code? What function I could extract? etc.

Refactoring should lead to different structure of code that has **the same** behavior from end user perspective.
But it doesn't mean the code works as it did previously. Whenever we extract a function - we change the callstack, right?
Performance can change as well. It can either get better or worse. But performance **is not** and **shouldn't be** goal of refactoring.
Increasing performance of application is goal of optimization.

Programming is all about talking to computer and commanding it to do the stuff that we want.
But there's other important factor, which is sometimes forgotten (especially when someone states that his code works and is doing what it's supposed to do).

**Human** factor. There's a very little chance that you, as an author of piece of code, are last person reading that.
I always take assumption that someone will have to read my code, sooner or later.

Even worse.

It's probably gonna be me, after some time. And my memory is not so good. So I better write readable letter to myself from future instead of having to think about the same problem, again.

## Is refactoring going to slow me down?
Short term? Maybe.

Long term it will increase your productivity. Code will be more readable and understandable.

So it leads to overall faster delivery of code that has better quality.

## When should I refactor?
Always. 

Not kidding.

Have you ever experienced situation when there was small piece of functionality that had to be added to existing code?

Did you then think that if you would change existing structure, just a little bit, it would be piece of cake to add the new feature here?

That was the right thinking. It's way better direction to take instead of copy pasting function and changing only the piece of code that you need.
This way the code would end up with duplicated logic, which needs to be changed in multiple places, whenever business logic changes (and you have to find that code in multiple places..).

## Is that really always?
Well. There's a but.
You should have tests.
Either way, how can you verify that one of the key aspects of refactoring is fulfiled?
The one that is saying there's *no change in [code's] observable behavior*.
If you don't have tests, try to write them before you start.
There are three major benefits of that:
1. Test will let you know whenever you broke functionality, so you'll have doing more confidence doing small changes
2. You'll get to understand code better
3. Tests often lead to simple & elegant solutions


## Summary
Try to continously improve your code. That process is called *Continuous Refactoring* or *Ongoing refactoring*. Don't wait until **you have to do that** because project gets harder and harder to maintain.

Write tests. They'll help you now or in future, when you'll decide that some piece of code needs to be modified in order to extend functionality.


