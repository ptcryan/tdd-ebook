How is TDD about analysis and what does "GIVEN-WHEN-THEN" mean?
===============================================================

Is there really a commonality between analysis and TDD?
-------------------------------------------------------

From Wikipedia:

> Analysis is the process of breaking a complex topic or substance into smaller parts to gain a better understanding of it.

Thus, for TDD to be about analysis, it has to fulfill two conditions: 

1.  Be a process of breaking a complex topic into smaller parts
2.  Allow gaining a better understanding of such broken topics

In the story about Johnny, Benjamin and Jane, I included a part where they analyze requirements using concrete examples. Johnny explained that this is a part of a technique called Acceptance Test-Driven Development. The process followed by the three characters fulfilled both mentioned conditions for a process to be analytical. But what about TDD itself?

Although I used parts of the ATDD process in the story to make the analysis part more obvious, similar things happen at pure technical levels. For example, when starting development with a failing application-wide Statement (we will talk about levels of granularity of Statements later. For now the only thing you need to know is that the so called "unit tests level" is not the only level of granularity we write Statements on), we may encounter a situation where we need to call a web method and assert its result. This makes us think: what should this method be called? What are the scenarios supported? What do I need to get out of it? How should its user be notified about errors? Many times, this leads us to either a conversation (if there is another stakeholder that needs to be involved in the decision) or rethinking our assumptions. This is how we gain a better understanding of the topic we are analyzing, which makes TDD fulfill the first of the two requirements for it to be an analysis method.

But what about the first requirement? What about breaking a complex logic into smaller parts?

If you go back to our example, you will note that both when talking to a customer and when writing code, Johnny and Benjamin used a TODO list. This list was first filled with whatever scenarios they came up with, but later, they would add a smaller unit of work. This is one way complex topics are decomposed into smaller items that all land on the TODO list (the other way is mocking, but we will not get into that yet). Thanks to this, we can focus on one thing at a time, crossing off item after item from the list after it is done. If we learn something new or encounter new issue that needs our attention, we can add it to the TODO list and get back to it later, for now continuing our work on the current point of focus.

An example TODO list from the middle of implementation may look like this (do not read through it, I put it here only to give you a glimpse):

1.  --- Create entry point to the module (top-level abstraction)
2.  --- Implement main workflow of the module
3.  --- Implement Message interface
4.  --- Implement MessageFactory interface
5.  Implement ValidationRules interface
6.  --- Implement behavior required from Wrap method in LocationMessageFactory class
7.  Implement behavior required from ValidateWith method in LocationMessage class for Speed field
8.  Implement behavior required from ValidateWith method in LocationMessage class for Age field
9.  Implement behavior required from ValidateWith method in LocationMessage class for Sender field

Note that some items are already crossed out as done, while other remain undone and waiting to be addressed.

Ok, that is it for the discussion. Now that we are sure that TDD is about analysis, let's focus on the tools we can use to aid and inform it. You already saw both of them in this book, now we are going to have a closer look.

Gherkin
-------

Hungry? Too bad, because the Gkerkin I am gonna talk about is not edible. It is a notation and a way of thinking about behaviors of the specified piece of code. It can be applied on different levels of granularity -- any behavior, whether of a whole system or a single class, may be described using it.

In fact we already used the Gherkin notation, we just didn't name it so. Gherkin is the GIVEN-WHEN-THEN structure that you can see everywhere, even as comments in the code samples. This time, we are stamping a name on it and analyzing it further.

In Gherkin, a behavior description consists of three parts:

1.  Given -- a context
2.  When -- a cause
3.  Then -- a effect

In other words, the emphasis is on causality in a given context.

As I said, there are different levels you can apply this. Here is an example for such a behavior description from the perspective of its end user (this is called acceptance-level Statement):

```gherkin
Given a bag of tea costs $20
When I buy two of them
Then I should be charged $30 due to discount
```

And here is one for unit-level (note the line starting with "And" that adds to the context):

```gherkin
Given a list with 2 items
When I add another item
And check items count
Then the count should be 3
```

While on acceptance level we put such behavior descriptions together with code as the same artifact (If this does not ring a bell, look at tools like SpecFlow or Cucumber or FIT to get some examples), on the unit level the description is usually not written down literally, but in form of code only. Still, this structure is useful when thinking about behaviors required from an object or objects, as we saw when we talked about starting from Statement rather than code. I like to put the structure explicitly in my Statements -- I find that it makes them more readable. So most of my unit-level Statements follow this template:

```csharp
[Fact]
public void Should__BEHAVIOR__()
{
  //GIVEN
  ...context...

  //WHEN
  ...trigger...

  //THEN
  ...assertions etc....
}
```

Sometimes the WHEN and THEN sections are not so easily separable -- then I join them, like in case of the following Statement specifying that an object throws an exception when asked to store null:

```csharp
[Fact]
public void ShouldThrowExceptionWhenAskedToStoreNull()
{
  //GIVEN
  var safeList = new SafeList();

  //WHEN - THEN
  Assert.Throws<Exception>(
    () => safeList.Store(null)
  );
}
```

By thinking in terms of these three parts of behavior, we may arrive at different circumstances (GIVEN) at which the behavior takes place, or additional ones that are needed. The same goes for triggers (WHEN) and effects (THEN). If anything like this comes to our mind, we add it to the TODO list.

TODO list... again!
-----------------

As we said previously, a TODO list is a repository for anything that comes to our mind when writing or thinking about a Statement, but is not a part o the current Statement we are writing. We do not want to forget it, neither do we want it to haunt us and distract us from our current task, so we write it down as soon as possible and continue with our current task.

Suppose you are writing a piece of small logic that allows user access when he is an employee of a zoo, but denies access if he is a guest of the zoo. Then, after starting writing a Statement it gets to you that any employee can be a guest as well -- for example, he might choose to visit the zoo with his family during his vacation. Still, the two previous rules hold, so not to allow this case to distract you, you quickly add an item to the TODO list (like "TODO: what if someone is an employee, but comes to the zoo as a guest?") and finish the current Statement. When you are finished, you can always come back to the list and pick item to do next.

There are two important questions related to TODO lists: "what exactly should we add as a TODO list item?" and "How to efficiently manage the TODO list?". We will take care of these two questions now.

### What to put on the TODO list?

Everything that you need addressed but is not part of the current Statement. Those items may be related to implementing unimplemented methods, to add whole functionalities (such items are usually followed my more fine-grained sub tasks as soon as you start implementing the item), there might be reminders to take a better look at something (e.g. "investigate what is this component’s policy for logging errors") or questions about the domain that need to get answered. If you get carried away too much in coding that you forget to eat, you can even add a reminder ("TODO: get something to eat!"). The list is yours!

### How to pick items from TODO list? 

Which item to choose from the TODO list when you have several? I have no clear rule, although I tend to take into account the following factors:

1.  Risk -- if what I learn by implementing or discussing a particular item from the list can have big impact on design or behavior of the system, I tend to pick such items first. An example of such item is that you start implementing validation of a request that arrives to your application and want to return different error depending on which part of the request is wrong. Then, during the development, you discover that more than one part of the request can be wrong at a time and you have to answer yourself a question: which error code should be returned in such case? Or maybe the return codes should be accumulated for all validations and then returned as a list? 
2.  Difficulty -- depending on my mental condition (how tired I am, how much noise is currently around my desk etc.), I tend to pick items with difficulty that best matches this condition. For example, after finishing an item that requires a lot of thinking and figuring out things, I tend to take on some small and easy items to feel wind blowing in my sails and to rest a little bit. 
3.  Completeness -- in simplest words, when I finish test-driving an "if" case, I usually pick up the "else" next. For example, after I finish implementing a Statement saying that something should return true for values less than 50, then the next item to pick up is the "greater or equal to 50" case. Usually, when I start test-driving a class, I take items related to this class until I run out of them, then go on to another one.

### Where to put a TODO list?

There are two ways of maintaining a TODO list. The first one is on a sheet of paper, which is nice, but requires you to take your hands off the keyboard, grab a pen or pencil and then get back to coding every time you think of something. Also, the only way a TODO item written on a sheet of paper can tell you which place in code it is related to, is (obviously) by its text.

Another alternative is to use a TODO list functionality built-in into an IDE. Most IDEs, such as Visual Studio (and Resharper plugin has its own enhanced version), Xamarin Studio or eclipse-based IDEs have such functionality. The rules are simple -- you put special comments in the code and a special view in your IDE aggregates them for you, allowing you to navigate to each. Such lists are great because:

1.  They do not force you to take your hands off keyboard to add an item to the list.
2.  You can put such item in a certain place in the code where is makes sense and then navigate back to it with a click of a mouse. This, apart from other advantages, lets you write shorter notes than if you had to do it on paper. For example, a TODO item saying "TODO: what if it throws an exception?" looks out of place on the paper, but when added as a comment to your code in the right place, it is mighty sufficient.
3.  Many TODO lists automatically add items for certain things. E.g. in C\#, when you are yet to implement a method that was automatically generated by a tool, its body usually consists of a line that throws a `NotImplementedException` exception. Guess what -- `NotImplementedException` occurences are added to the TODO list automatically, so we do not have to manually add items to the TODO list for implementing the methods where they occur.

### Expanded TDD process with a TODO list 

In one of the previous chapters, I introduced you to the basic TDD process that contained three steps: write unfulfilled Statement, fulfill it and refactor the code. TODO list adds new steps to this process leading to the following list of steps:

1.  Examine TODO list and pick an item that makes most sense to implement next
2.  Write unfulfilled Statement
3.  Make it unfulfilled for the right reason
4.  Fulfill the Statement and make sure all already fulfilled Statements are still fulfilled
5.  Cross out the item from TODO list
6.  Repeat until no item is left on the TODO list

Of course, we are free to add new items to the TODO list as we make progress with the existing ones and at the beginning of each cycle the list is re-evaluated to choose the most important item to implement next taking into account what was added during the previous cycle.

### Potential downsides

There are also some downsides. The biggest is that people often add TODO items for other means than to support TDD and they never go back to such items. Some people joke that a TODO left in the code means "Once, I wanted to...". Anyway, such items may pollute your TDD-related TODO list with so much cruft that your own items are barely findable. To work around it, I tend to use a different tag than TODO (many IDEs let you define your own tags, or support multiple tag types out of the box. E.g. with Resharper, I like to use "bug" tag, because this is something no one would leave in the code) and filter by it. Another options is, of course, getting rid of the leftover TODO items -- if no one addressed it for a year, then probably no one ever will.

Another downside is that when you work with multiple workspaces/solutions, your IDE will gather TODO items only from current solution/workspace, so you will have few TODO lists -- one per workspace or solution. Fortunately, this isn’t usually a big deal.
