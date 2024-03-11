---
layout: post
title: The sneaky stackoverflow exception
categories: [dotnet, stackoverflow, dependencymanager]
---
It was a productive day as my colleague and I were finalizing work on a feature. However, after a series of commits, we discovered that a particular commit had triggered the failure of a substantial portion of our REST tests. The situation left us baffled.

Not only were the tests for the modified services failing, but also some unrelated ones were affected. Determined to identify the root cause, we systematically went through the following steps:

1. Confirmed successful project compilation - check.
2. Verified absence of warnings or errors in the code - clean as could be.
3. Reverted back to the state before the problematic commits - everything worked flawlessly.
4. Confirmed passing integration and unit tests - all clear.
5. Local execution of REST and end-to-end tests - failure.

In an attempt to shed light on the issue, we initiated the tests in debug mode. As we ran the test project, we noticed an anomaly: certain tests were failing after specific requests.

```c#
this.api.DoSomething(dto);
this.api.DoSomething(secondDto); 
this.api.DoSomething(thirdDto); // <= failure occurs here
this.api.DoSomethingDifferent();
```

Curious, we experimented by altering the order of events:

```c#
// These calls did not depend on each other

this.api.DoSomething(thirdDto); // switched with the first line
this.api.DoSomething(dto);
this.api.DoSomething(secondDto); // moved to the third line
this.api.DoSomethingDifferent();
```

Upon recompiling and rerunning the project, we encountered the unexpected:

```c#
this.api.DoSomething(thirdDto);
this.api.DoSomething(dto); // now the server crashes here?
this.api.DoSomething(secondDto);
this.api.DoSomethingDifferent();
```

Surprisingly, changing the order again resulted in a crash on a different line. 

*Sigh* Okay...

Our next move was to enter debug mode on the server and run the tests in debug mode. The tests started, the server started, and as anticipated, the tests began executing, leading to a server crash. The scenario mirrored what we had observed earlier. "Great," we thought, "now we can at least see what the error message is" However, there was nothing in the big black console window.

After some investigation, We discovered a Visual Studio setting that filtered console logs. Adjusting it to display all logs provided a breakthrough, and we ran the tests and server in debug mode once more.

```Stackoverflow exception```

Something seems fishy. Why are we getting a stack overflow exception? Why does it occur after some requests but not others? It appears to be random.

We decided to debug the server implementation. Starting the server and tests in debug mode, we allowed the tests to commence. In a classic fashion, I forgot to place a breakpoint, so we had to start over. The second time around, I ensured the breakpoint was set, initiated everything in debug mode, and prepared for what seemed like the ride of my life.

`Stackoverflow exception`

Curiously, it seemed like the breakpoint wasn't being hit. To confirm, I added a breakpoint to the controller and repeated the process.

`Stackoverflow exception`

The breakpoint was not being hit. You can never be sure so I added more breakpoints.

`Stackoverflow exception` with no breakpoints beeing hit.

Confused and frustrated, I used my professional Googling skillz and stumbled upon an [interesting GitHub issue](https://github.com/dotnet/runtime/issues/62239). Reading through the thread, I found a [comment](https://github.com/dotnet/runtime/issues/62239#issuecomment-987086077) suggesting that using `Microsoft.Extensions.DependencyInjection` with dependencies in a loop could lead to a stack overflow exception.

Yes... it turned out that our code was creating too much dependencies. After some laughter and refactoring, our pull request was green, and the issue was resolved.

Cheers to microsoft for the clear error message ![crying cat thumbs up image](https://i.kym-cdn.com/entries/icons/original/000/034/772/Untitled-1.png)


Reflecting on the experience, it became one of those challenges that give me a valuable lessons. Thanks to this *experience* :star2: :star2: :star2:, I discovered the peculiar log settings in Visual Studio and even delved into the documentation of the dependency injection library's source code.