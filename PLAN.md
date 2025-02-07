30mn ~ 15 slides

1) Landing page


2) The why's before any how...

-> The goal of this talk is not to talk about a solution like that, I'm not fan of that approach
-> Instead what I propose you is to go through various kind of problems we face when writing programs
-> Effect as one solution, not the only one. Could be anything else, just turns out that Effect is the only one solving such amount of problems in the TypeScript landscape


3) Who am I

Freelance Software Engineer

Open-Source Author (skott)

Effect advocate (effect-introduction, talks, Effect Paris co-organizator which is by the way one of the community partners of the conference)

3+ years of building Effect in production 


4) Some of the universal challenges when building Production-Ready Software

Universal stuff, we're not talking about TypeScript stuff only
We have 

    - Resilience
        -> errors
        -> retries
        -> interruptions
        -> timeouts

    - Dependencies
        -> testing
        -> decoupling
        -> dependency injection

    - Concurrency
        -> fine-grained control over concurrency
        
    - Resource management
        -> strong guarantees of resources management

    - Observability

5) Production-Ready is hard: Problems in the ecosystem

- fragmented ecosystem
    control flow: ts-results, neverthrow, p-retry
    dependency injection: tsyringe, di, awilix, inversify
    schema validation: zod, yup, joi
    fp librairies: fp-ts, ramda, 
    data structures: immer 

6) What is Effect: a unified solution to face these challenges with TypeScript

Effect addresses all these problems in an efficient way.

Effect takes its root using a data type `Effect<A, E, R>`

7) Resilience

Definition

When it comes to dealing with errors, TypeScript=try/catch not type-safe
= defensive programming, not modeling errors

8) Dependencies

-> Describing Dependencies in a type-safe way
An Effect program can't run until all dependencies are satisfied

-> Testing

Definition

Effect comes-in builtin with the DIP, SOLID principles

9) Concurrency

Definition

10) Resource Management

Definition

11) Observability

Definition, builtin OTEL support

12) Ecosystem

All available modules

-> Batching/Caching
-> Input/Output validation -> Schema ?
-> http module
-> etc, etc.

13) Disclaimers

Effect does not necessarily need to be an all-in
    -> it's a toolkit, you only pick what you need
    -> tree-shakeable 
    -> micro module
    -> good promise/callback/sync interop
    -> Effect is solving full-stack problems


14) And now, you have the choice

Thing is, you will face these challenges, with or without Effect.

Go the hard way, and I encourage that to really feel the pain

Go the pragmatic way, using Effect (or equivalent = none yet)


15) More resources

Discord
effect-introduction

