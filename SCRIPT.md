1) (45s) ✅

Hello everyone, I hope you're having a great conference here in Paris!

I believe I'm one of the only two french speakers today with Nicolas, and I'm proud of representing France and especially Paris.

Alright today I'll talk about challenges we are facing when writing Production-Ready Software, and I will present you a library, Effect, which is one solution addressing these problems using TypeScript.

Quick question, is there some people who already heard about Effect? Who's using Effect in production?

2) (45s) ✅ 

A small disclaimer regarding the talk: you won't become an Effect master after that talk and that's for a simple reason.

That's because my objective is not to sell just a new solution like that, and by the way Effect is fully open-source so indeed there is nothing to sell, instead what I propose you is to go through various kind of problems we face when writing programs and see how Effect can become a great solution when using TypeScript.


3) (45s) ✅ 

Before diving into the talk, let me introduce myself first.

My name is Antoine, I'm a full-stack Software Engineer currently working as a Freelance and my the way I see my job is to help companies succeed using technical expertise.

I'm also involved in Open-Source development:

- I created skott, which is a tool to build project graphs that can be used in many ways, one of them is visualization to detect circular dependencies or other information

And I'm also contributing to the Effect ecosystem as an Advocate:
- I wrote the `effect-introduction` on GitHub, which is currently one of the main ressources helping when starting with Effect
- I co-organize the Effect Paris meetup


4) (45s) ✅ 

As I said in the introduction, we all face some universal challenges when building Production-Ready Software. And what I mean by "universal" is that it's not problems specific to TypeScript, I'm talking about problems you will likely have with other languages and within other ecosystems. 

[INTERACTION] It just turns out that TypeScript is the best language out there right? So let's try to solve them using TypeScript

During that talk we will go through each of these challenges and see how can we deal with them without and with Effect.

[TRANSITION] And by the way these are the main ones we usually go through but there are many more

5) (1mn) ✅

Facing these challenges is hard. But it is even more complex when you need to compose multiple solutions. For instance, we could have multiple independent solutions that tackle each one of these problems, but then you need to compose these solutions together, and you end up having a lot of glue and a lot of complexity, because APIs are not the same and are not necessarily meant to be packed together.

And it's great! But fragmented, and costly to manage, no unified API. And I'm not even talking about the maintenance cost, the vulnerability surface, etc.

6) (1mn30) ✅

So now it's time to talk about Effect. Effect is a unified solution to face these challenges with TypeScript. Just a side note, if you don't face any of these problems I have been presenting you, then you're fine. Also, if you already have other solutions and they work for you, then all good. 

Nonetheless from my experience of building applications with TypeScript for the past decade and Effect for the past 3 years, I didn't see any solution that came even close to what Effect provides me within the TypeScript ecosystem with such amount of productivity.

An Effect is nothing but a data type that aims to represent some kind of Program.

That data type is generic and has 3 type parameters: `Effect<A, E, R>`

- The first one represents the success of the Program `Effect<Success>`. This is what we are used to.
- The second one represents the set of failures that can be produced when executing that program: `Effect<Success, Failures>`
- The last one stands for the requirements in order for that program to be run: `Effect<Success, Failures, Dependencies>`

[TRANSITION] And during that talk we will see how powerful it is to have these three type parameters 

7) (2mn) ✅

Let's take a concrete example of a simple use case: creating a user.

As you can see we import Effect from the effect library.

We can then describe our operation as an Effect that : 
- if succeeding produces a CreatedUser
- but that can fail for a reason, in that case if the User already exists
- and that use case needs one dependency, which is a user repository, that might interact with some
kind of database to persist these operations.

And what's great with Effect is that we can encode a lot of information leveraging discriminated unions.

For example there the create User might also fail for another reason which is related to organizations. And interacting with organizations also require another reference to a repository.

And you will see how powerful these discriminated unions become just after. So you can basically accumulate information, not only coming from a single operation but also when chaining and nesting operations, because this is usually what happens in a real-world program.


[TRANSITION]: So now that you know what an Effect is and what it represents, let's see how it helps face each of these challenges I've been presenting just before 

8) (2mn) Resilience: error management ✅

The first problem I want to talk about is Resilience that can be described as the `art of designing and implementing systems which can react and recover from expected failures.`

Let's focus on Error management first.

With TypeScript we need to deal with two kind of errors, synchronous and asynchronous errors.

Let's take for example a simple synchronous function that declares `generateRandomNumber(): number`, the problem we have is that we only encode the Success information. We have no clue about whether that function can fail or not. This is lacking from explicitness as you can end up having errors being thrown.

Now if we take a look at asynchronous operations, we have same problem. Let's take for instance Promises, by default they are only meant to represent the success information.

The problem there is that we end up exceptions being typed as unknown or any, we often end up using defensive programming and having try/catch statements just for safety, sometimes we don't do anything in these catch blocks.

And even worse, the problem is that these errors lack of explicitness but also the behavior between synchronous and asynchronous APIs change and the way to manage errors is not the same.

9) (1mn) Re-creating the lost information ✅

And because exceptions are being typed as unknown or any, what we end up doing is trying to re-create some kind of type-level and runtime information that we lose during the process of throwing exceptions.

So at some point we had it, but then we threw an exception and then we just go back into the dark world of any and unknown.

[TRANSITION] We love our dear TypeScript type-system so why not just having another way of dealing with errors
 
10) (2mn) Error management with Effect ✅

What if I tell you that with Effect we can represent Errors as Values and deal with them as if they were part of the happy path

Let's just consider two simple TypeScript classes, with a tag, that will act as the discriminating property there, both at type-level and runtime

These classes can then be used together with Effect as part of the Failure channel.

And having that being explicitly represented is quite powerful as we can now do PATTERN MATCHING!

In that case we are using an exhaustive pattern matching, but with Effect you can also deal use non-exhaustive pattern matching.

The best thing is that because the pattern matching was exhaustive, the compiler now knows that we managed all errors, so that our Effect can encode back that information: we have no error left that need to be handled right after!

11) (1mn) Resilience: more than managing errors ✅

When it comes to resilience we also have other challenges, we need to have retries, with custom policies.

Effect provides us a very powerful module which is called Schedule, that can be used to represent some kind of Schedule patterns that can be both used for Retries, but also for recurring tasks, such as Cron jobs.

As you can see with one line of code we are able to retry an Effect with a very complex policy, that might have taken hundred lines of imperative code otherwise.

You also have to deal with resource-safety, interruptions and timeouts, and Effect provides you 
everything to deal with that.

12) (2mn) Dependencies 

For now we saw two of the three type parameters of the Effect data-type.

And as you know it comes to Production-Ready Software, most of the time you need to integrate dependencies, 

such as a repository that communicates with some kind of database. 

Effect makes these dependencies explicit by using the last type parameter and what's great about this is that
dependencies listed there are only interfaces that act as a contract for such dependency.

In other words, Effect helps us achieving decoupling through the Dependency Inversion Principle.

The goal is to reduce coupling to these dependencies: `Program to an interface, not an implementation`

We can refer to these dependencies through tags, that are linked to a dedicated interface, and we can rely on these services by yielding these tags that offer an access over the service contract

But as you can see, there is no concrete implementation there ; we just model an effect that needs a user repository matching the contract.

13) (2mn) Dependencies : type-safe DI

What's powerful with Effect is that we have type-safety around these dependencies, meaning that the compiler will complain if they are not injected.

For instance there we are trying to run the `registerUser` use case but as I mentioned we didn't provide any implementation of the repository yet. This is clearly made explicit as the UserRepository is still there in the Effect data type.

What we then need to do is to provide, or INJECT, a concrete implementation matching the contract to that Effect, satisfying all the requirements.
 
Once we do that, the type argument becomes "never", and it the same way as for error, it means that we managed all dependencies.

-> An Effect program can't run until all dependencies are satisfied and we have strong type-safety around it.

14) (1mn) Dependencies : testing

And because we are leveraging the Dependency Inversion Principle, it now becomes super easy to inject whatever instance of the service that fakes the behavior for testing purposes.


15) (1mn) Concurrency 

Now let's talk about another challenge, Concurrency.

Concurrency is quite complex because there are so many things to get right.

Even though Node.js and the Event Loop model solve many concurrency issues and abstract them for us,
we still have concurrency challenges.

We can still have race conditions, starvation, issues with shared resources, memory and cpu management...

16) (2mn) Concurrency

Let's take a simple example to showcase how poor control we have over concurrency when using Promises.

There we use Promise.all that takes an iterable of Promises, that will be fired concurrently, so depending on the size of userIds array we might have thousands of Promises being fired up at the same time.

This is what we call "Unbounded" concurrency meaning that there is no limit for that concurrent execution, we might end up using 100% CPU.

Also, we don't have any interruption handling, if one of these Promises fail, the other ones keep running.

Now with Effect, we can have exactly the same use case but with a better control over concurrent execution:

- we can limit the number of concurrent operations
- we can inherit from the concurrency config from the parent
- we can opt-in for unbounded as well 

And also, we have strong safety around interruptions and resource-safety if one of these operations fail unlike with Promises

17) (2mn) Resource Management 

Concurrency is directly linked to Resource Management, because most of the time we have Resource issues when dealing with concurrency, for instance what happens if what of the concurrent operation fails or needs to be interrupted.

Promise.race is a great example of a leaking API. You might not know it, but the loser of the racer does not get interrupted. So there the `setTimeout` keeps running for 10s in background, even though the race result is already settled.

Let's have a look of a way of interrupting async operations without Effect using the AbortController API, we need to create controller, forwarding the signal property, and correctly call the abort on each of these controllers to clear the timeout. 

That's pretty cumbersome and verbose and above all, it's not safe as the signal does not offer guarantees of the operation to be cancelled ; you only notify an abort but it's up to the receiver of that signal to implement the interruption / teardown logic.

Now let's see with effect, we also describe a race, but unlike the other version the loser gets directly interrupted by default.

And we can attach some handler to react on that interruption, but by default all Effect and interruptible and we have strong guarantees offered by the Effect runtime that they will be interrupted.

We have many APIs with Effect to model release logic, for instance there it looks alike the useEffect
pattern where you return a teardown Effect, that will be called if background job gets interrupted.


18) Resource Management (1mn)

Beyond the simple scope of these examples, Effect provides a fully fledged model around resource-safety, that composes nicely and is built-in.

19) Observability (2mn)

The last challenge I want to go through is Observability. And some of you might have went deep into the rabbit hole of observability and suffered from it.

Thing is, observability is a key element to have Production-Ready Software, so it's really sad that we can't fully leverage observability because it's too complex.

What if I tell you that Effect comes in with all that and has a built-in support for OpenTelemetry

We have a Logger, that has a high-level of flexibility

We also have Tracing, for instance there the withSpan function, in case of failure or sucess everything is monitored and is correctly nested in a structured way


20) (1mn) A wide ecosystem to cover even more challenges 

The Effect ecosystem provides a lot of modules and features that go beyond the scope of that talk:

caching, batching, streams, schema validation, data structures and many more...

most of these modules are built-in into effect and composes nicely 

21) (45s) Disclaimer

And right after the slide you might think that it's too huge and I don't have all these problems.

So it's time to provide a quick disclaimer: Effect does not necessarily need to be an all-in move: it's only a toolkit, you only pick what you need from it, and you just use it where you need to.

    -> tree-shakeable 
    -> micro module
    -> good promise/callback/sync interop
    -> usable both on frontend and backend environments, fully based on Web APIs

22) (1mn)

If you need to remember one thing from that talk is that Effect is meant to solve these hard problems
without involving exponential complexity level. 

You can solve all these problems without Effect, but you will likely end up with high level of complexity in your applications.

23) (15s)

And that's it for me, thanks for listening!

Discord
effect-introduction
