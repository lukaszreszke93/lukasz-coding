10 Tips To Improve Your Code Reviews
==========
<!-- more -->
## Talk about CODE, not the PERSON
It's common that we either take or write a comment personally.
Understandable, clean and performant code base should be our goal as a team.
That's why it's very important to talk about code.

for example: *I am afraid that this class is having multiple responsibilities. 
If we split it, we will be able to write tests to new parts of code and have more confidence changing it later, if necessary.*

## Split commits into smaller chunks. 
Try to separate pure business changes from variables renaming in 100 files or moving files between projects

## Think about business problem that the code is solving and look for issues in that context first
Its common for people to focus on finding something just to give a comment to other programmer.
Don't be that guy.
Focus on business problem first.
Variable name is less important than violation of your core business rules.

## Look for code smells like coupling code in bouned context that shouldn't have place
There are different types of coupling in different types of architectures.
Try to focus on those that are most important in your architecture
and find the best solution with your team mates.

## Remember! There's a person on the other side.
Be empathic! Someone might just as well have worse day.
Or maybe got stuck in some complex solution. 
And you might know how to simplify it. If you communicate it properly I bet that the other person gonna be happy to hear how to make things simplier.
We, programmers, love to learn, right?


## Be explicit with your comments.
Something that might be obvious to you, might not be obvious to PR author.
Try to be as explicit as you can with your comments (taking time into consideration, as this is important factor)

Don't get angry when you get asked about more details. Try to make culture where people are not afraid to ask questions.

## Talk about your expectations for code
In your team take about expectations that you have for code.
Decide on your coding convention and on code review's scope. It's important to have agreement on that.
It will generate less frustrations at the end of a day.

## Use static code analyser to pick up changes that are not compliant with coding conventions that your team uses.
When you have agreement on coding conventions, include some type of static code analyser.
You can do it in multiple different ways. Failing the build, automatic comments in pull request from bot, etc.
Again, it depends on what you agree on with your team.

## Be polite
You might be wrong.
The change you suggest might be just better according to your beliefs.
Always be polite.

## The best code review is the one where you don't leave a comment
Don't set yourself for finding ANY issue.

The best code review is the one where you don't write any comment.
It indicates that the team knows architecture well and follows coding standards.