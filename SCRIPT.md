1) (45s) ✅

Hello everyone, I hope you're having a great conference here in Paris!

I believe I'm one of the only two french speakers today with Nicolas, and I'm proud of representing France and especially Paris.
So as you might know, if you have any questions about Paris, really please don't ask them to me, as you know Parisian people don't like people asking questions.

Just joking...

Alright today I'll talk about challenges we are facing when writing Production-Ready Software, and I will present you a library, Effect, which is one solution addressing these problems using TypeScript.

Quick questions, is there some people who already heard about Effect? Who's using Effect in production?

2) (30s) ✅

A small disclaimer regarding the talk: you won't be an Effect master after that talk and that's for a simple reason.

The goal of this talk is not to sell just a new solution like that, and by the way Effect is fully open-source so indeed there is nothing to sell, it's just I'm not fan of that approach. Instead what I propose you is to go through various kind of universal problems we face when writing programs, whatever the language or the ecosystem is, and see how Effect can become a great solution for these problems when using TypeScript.


3) (45s) ⌛

Before diving into the talk, let me introduce myself first.

My name is Antoine, I'm a Software Engineering Consultant, and my job is to help companies succeed using full-stack expertise.

I have around 3 years of experience building Effect in production.

I'm also involved in Open-Source development:

- I created skott, which is a tool to build project graphs that can be used in many ways, one of them is visualization to detect circular dependencies or other information
- I also wrote the `effect-introduction` on GitHub, which is currently one of the main ressources helping when starting with Effect
- I co-organize the Effect Paris meetup, which is btw one of the community partners of the conference


4) (1mn30)

As I said in the introduction, we all face some universal challenges when building Production-Ready Software. And what I mean by "universal" is that it's not problems specific to TypeScript, I'm talking about problems you will likely have with other languages and other ecosystems. It just turns out that TypeScript is the best language out there right? Agree?

Let's have a quick overview of some of these challenges:

- Resilience

You need to have proper error management, including retries, interruptions, timeouts.

- Dependencies

To integrate dependencies with need composition, testing, decoupling, dependency injection

- Concurrency

Our application must maximize efficiency through a proper concurrency management that is not blowing up memory or CPUs.

- Resource management

We must not leak resources

- Observability

Having precise information about what happens in production

[TRANSITION] And many others, these are the main ones we usually go through.

5) (2mn)

Facing these challenges is hard as solutions are sometimes complex. But it is even more complex when you need to compose all these solutions for the whole set of challenges. For instance, we could have multiple independent solutions that tackle each
of these problems, but then you need to compose these solutions together, and you end up having a lot
of glue and a lot of complexity, because APIs are not the same and are not necessarily meant to be packed together.

Let's see some of these solutions within the JavaScript ecosystem:

- control flow: ts-results, neverthrow, p-retry
- dependency injection: tsyringe, di, awilix, inversify
- schema validation: zod, yup, joi
- fp librairies: fp-ts, ramda
- data structures: immer, immutable-js

And it's great! But fragmented, and costly to manage, no unified API.


6) (3mn)

Effect is a unified solution to face these challenges with TypeScript. If you don't face any of these problems, then you're fine. Also, if you already have other solutions, then fine. 

Nonetheless from my experience of building applications with Effect for the past 3 years, I didn't see any solution that came even close to what Effect provides me within the TypeScript ecosystem with such productivity.

So let's start with a bit of concrete showcase of what Effect is.

Effect takes its root using a data type that aims to represent some kind of Program, or an Operation.

That data type is generic and has 3 parameters: `Effect<A, E, R>`

And that program can either:
- A: Succeed, with a type of A: `Effect<Success>`
- E: Fail, for a given set of failures: `Effect<Success, Failures>`
- R: Require, dependencies for that program to be run: `Effect<Success, Failures, Dependencies>`

Let's take a concrete example of an operation of creating a user.

```ts
const createUser: Effect.Effect<CreatedUser, UserAlreadyExists, UserRepository> = //
```

And all these type parameters can leverage discriminated unions to encode multiple possibilities.

```ts
const createUser: Effect.Effect<CreatedUser, UserAlreadyExists | OrganizationNotFound, UserRepository | OrganizationRepository> = //
```

[TRANSITION]: So now that you know what an Effect is and what it represents, let's see how it helps face each of these challenges I've been presenting just before 

7) (3mn)

Resilience

`Resilience is the art of designing and implementing systems which can react and recover from expected failures.`

- Error management

With TypeScript we need to deal with two kind of errors, synchronous and asynchronous errors.

Let's take for example a simple synchronous function that declares `generateRandomNumber(): number`, the problem we have is that we only encode the Success information. We have no clue if that function can fail or not. This is lacking from explicitness as you can end up having errors being thrown.

So what we end up doing is trying to re-create some kind of type-level and runtime information that we lose during the process of throwing exceptions.

Now if we take a look at asynchronous operations, we have same problem. Let's take for instance Promises, by default they are only meant to represent the success information.

And even worse, the problem is that these errors lack of representation but also their behavior change and the API to manage errors change.


Let's see how Effect solves this

-> What if we wanted to encode errors in a type-safe and composable way?

```
class UserAlreadyExists {
    readonly _tag = 'UserAlreadyExists'
}

```

And then on top of that you can have retries, interruptions, timeouts...

Putting all together we can see what this becomes without Effect, and what it is with Effect

8) Dependencies (3mn)

When it comes to Production-Ready Software, most of the time you need to integrate dependencies, such as a repository that communicates with some kind of database. The goal is to reduce coupling to these dependencies: `Program to an interface, not an implementation` -> DIP

Effect makes these dependencies super explicit by having the dedicated channel R for that information

Decoupling using Dependency Inversion Principle

-> Describing Dependencies in a type-safe way
An Effect program can't run until all dependencies are satisfied

1. Testing (dependency injection), DIP

2. Graphs of dependencies, type-safe and highly composable (showing graph of dependencies)

9) Concurrency (3mn)

"Concurrency is about dealing with a lot of things at once" - Rob Pike, Go creator

- Controlling the concurrency (bounded vs unbounded) 

Promise.all() on its own does not over any way of controlling the concurrency

Effect.all offers way of concurrency + inheritance

Resource-safe concurrency

-> Prendre la slide https://antoine-coulon.github.io/effect-introduction-for-paris-typescript-meetup/19?clicks=2


10) Resource Management (3mn)

Even out of the scope of concurrency you still need to have proper Resource Management

Promise.race comparison 
Clean interruption
Acquire/release semantics with strong guarantees

-> Structured Concurrency model, which links each operation to a parent and all

11) Observability (2mn)

Observability is a key element to have Production-Ready Software, as it helps you knowing what's going on in Production.

You often need: Logging, Tracing, Metrics...

Effect comes in with all that and has a built-in support for OpenTelemetry

-> logger
-> withSpan, composes a set of spans for tracing purposes, in case of failure or sucess everything is monitored


12) A wide ecosystem to cover even more challenges (2mn)

- Batching/Caching builtin
- Streams builtin `effect/Streams`
- Schema Validation (built-in, through `effect/Schema` module)
- Configuration Management (built-in, through `effect/Config` module)
- Data structures 
- Statement Management
- Scheduling (Cron module)

13) (1mn)

I just wanted to finish that talk by providing few disclaimers 

- Effect does not necessarily need to be an all: it's a toolkit, you only pick what you need
    -> tree-shakeable 
    -> micro module
    -> good promise/callback/sync interop
    -> Effect is solving full-stack problems, usable on both frontend and backend ecosystems

14) (1mn)

And now you have the choice

Keep in mind, you will face these challenges, with or without Effect.

Go the hard way, and I encourage that to really feel the pain

Go the pragmatic way, using Effect (or equivalent = none yet)

15) (15s)

Thanks for listening!

Discord
effect-introduction
