What is the scope of a unit-level Statement in TDD?
===================================================

Ha, now I have to admit that I deferred for a long time to answer a pretty fundamental question: what should be the scope of a single Statement? If I put the whole system together, can I write a Statement for its behavior? Or maybe the other way round -- there should be a Statement for each method of each class, including the private ones? Well, first thing I want to explain is that there are multiple levels we can write our Statements on. This varies depending on the TDD authority, but in this book, we will cover two of such levels -- unit level and acceptance level. For now, let's stick to the unit level, which is what we have done so far anyway. The time will come for the rest. This is, however, a good moment to stop and consider the "scope" of a single unit-level Statement in TDD. Is it method scope? Class scope? Feature scope?

Let's try to answer the question by examining some TDD unit-level Statements:

Is it class scope? 
------------------

Let's see the first example and try to answer this question:

```csharp
[Fact] public void
ShouldThrowValidationExceptionWithFatalErrorLevelWhenValidatedStringIsEmpty()
{
  //GIVEN
  var emptyString = string.Empty;
  var validation = new Validation();

  //WHEN
  var exceptionThrown = Assert.Throws<CustomException>(
    () => validation.Perform(emptyString) 
  );
  
  //THEN
  Assert.True(exceptionThrown.IsFatalError);
}
```

This is an example of a well-written unit-level Statement. Ok, so let's see... how many real classes take part in this spec? Three: a string, an exception and the validation. So the class scope is not the most accurate description.

Or a method scope?
------------------

So, maybe the scope covers a single method, meaning a Statement always exercises one method of a specified object?

Let's consider the following example:

```csharp
[Fact] public void 
ShouldBeFulfilledWhenEventOccursThreeTimes()
{
  //GIVEN
  var rule = new FullQueueRule();
  rule.Queued();
  rule.Queued();

  //WHEN
  rule.Queued();

  //THEN
  Assert.True(rule.IsFulfilled());
}
```

Count with me: how many methods are called? Depending on how we count, it is two (`Queued()` and `IsFulfilled()`) or four (`Queued(), Queued(), Queued(), IsFulfilled()`). In any case, not one. So it is not method scope either.

It is the scope of class behavior!
----------------------------------

The proper answer is: behavior! Each TDD Statement specifies a single behavior. I like how [Amir Kolsky and Scott Bain](http://www.sustainabletdd.com/) phrase it, by saying that each unit-level Statement should "introduce a behavioral distinction not existing before".

It may look that "behavior" scope is broader than method or class-level scope, since such Statement can span multiple classes and multiple methods. This is only partially true. That is because e.g. Statements with method scope can span multiple behaviors (which, by the way, is a sign of poorly written Statement). Let's take a look at an example:

```csharp
[Fact] public void 
ShouldReportItCanHandleStringWithLengthOf3ButNotOf4AndNotNullString()
{
  //GIVEN
  var bufferSizeRule = new BufferSizeRule();
  
  //WHEN
  var resultForLength3 
    = bufferSizeRule.CanHandle(Any.StringOfLength(3));
  //THEN
  Assert.True(resultForLength3);

  //WHEN again?
  var resultForLength4 
    = bufferSizeRule.CanHandle(Any.StringOfLength(4))
  //THEN again?
  Assert.False(resultForLength4);

  //WHEN again??
  var resultForNull = bufferSizeRule.CanHandle(null);
  //THEN again??
  Assert.False(resultForNull);
}
```

Note that it specifies three (or two -- depending on how you count) behaviors: acceptance of string of allowed size, refusal of handling string above the allowed size and a special case of null string. As I said -- this is an antipattern and is sometimes called a "check-it-all test". The issue with this kind of Statement is that it can be evaluated to false for at least two reasons -- when the allowed string size changes and when null handling is done in another way. Also, xUnit tools by default stop execution on first error, so, assuming that the first assertion fails, we will not know the outcome of the next assertion unless we fix the previous one (does that mean we have to use a single `Assert` per Statement? We will take care of this question later. For now, let the answer be: "not necessarily").

On the other hand, Statements with behavior scope do not necessary have to be broader than those with class scope. Let's take the following example that proves it:

```csharp
[Fact] public void
ShouldReportItIsStartedAndItDoesNotYetTransmitVoiceWhenItStarts()
{
  //GIVEN
  var call = new DigitalCall();
  call.Start();
 
  //WHEN
  var callStarted = call.IsStarted;
  
  //THEN
  Assert.True(callStarted);

  //WHEN-THEN
  Assert.Throws<Exception>(
    () => call.Transmit(Any.InstanceOf<Frame>())
  );
}
```

Again, there are two behaviors here: reporting the call status after start and not being able to transmit frames after start. That is why this test should be split into two.

How to catch that you are writing a Statement about two or more behaviors rather than one? First, take a look at the test name -- if it looks strange and contains some "And" or "Or" words, it may (but does not have to) be about more than one behavior. Another way is to write the description of a behavior in a Given-When-Then way. If you have more than one item in the "When" section or the structure is not Given-When-Then, but rather a "Given-When-Then-When-Then" -- that is also a signal.
