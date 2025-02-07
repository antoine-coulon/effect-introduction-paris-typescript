---
# try also 'default' to start simple
theme: dracula
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
# some information about the slides, markdown enabled
info: |
  ## Antoine Coulon, Paris TypeScript #1
  
  Using Effect to build Production-Ready TypeScript Software
# persist drawings in exports and build
drawings:
  persist: false
# use UnoCSS (experimental)
css: unocss
---

## Using Effect to build Production-Ready TypeScript Software

<br>
<br>

**Antoine Coulon** @ **Paris TypeScript la Conf'** #1

---
layout: center
class: "text-center"
---

# Effect: the **why** rather than the **how**

---

<div class="grid grid-cols-10 gap-x-4 pt-5 pr-10 pl-10">

<div class="col-start-1 col-span-7 grid grid-cols-[3fr,2fr] mr-10">
  <div class="pb-4">
    <h1><b>Antoine Coulon</b></h1>
    <div class="leading-8 mt-8 flex flex-col">
      <p class="mt-3">Lead Software Engineer</p>
      <p class="mt-3">Author <b color="cyan">skott, effect-introduction</b></p>
      <p class="mt-3">Advocate <b color="cyan">Effect</b></p>
      <p class="mt-3">Contributor <b color="cyan">Rush.js, NodeSecure</b></p>
    </div>  
  </div>
  <div class="border-l border-gray-400 border-opacity-25 !all:leading-12 !all:list-none my-auto">
  </div>

</div>

<div class="pl-20 col-start-8 col-span-10">
  <img src="https://avatars.githubusercontent.com/u/43391199?s=400&u=b394996dd7ddc0bf7a317185ee9c378d5b609e12&v=4" class="rounded-full w-40 margin-0-auto" />

  <div class="mt-5">
    <div class="mb-4 flex justify-between"><ri-github-line color="blue"/> <b color="opacity-30 ml-2">antoine-coulon</b></div>
    <div class="mb-4 flex justify-between"><ri-twitter-line color="blue"/> <b color="opacity-30 ml-2">c9antoine</b></div>
    <div class="mb-4 flex justify-between"><ri-user-3-line color="blue"/> <b color="opacity-30 ml-2">dev.to/antoinecoulon</b></div>
  </div>
</div>

</div>

<style>
  h1 {
    color: #4c7fff;
  }
  img {
    margin: 0 auto;
  }
</style>

---

## Building Production-Ready Software: the challenges

<br>

> non-exhaustive

- Resilience: error management, retries, interruptions, timeouts.

- Dependencies: composition, testing, decoupling, dependency injection

- Concurrency: efficiency, race conditions

- Resource management: leaks

- Observability

---

## The JavaScript ecosystem: rich but fragmented

<br>

- control flow: ts-results, neverthrow, p-retry
- dependency injection: tsyringe, di, awilix, inversify
- schema validation: zod, yup, joi
- fp librairies: fp-ts, ramda
- data structures: immer, immutable-js


---

## Introducing Effect: a unified solution to most of these challenges

Effect is a generic data type representing a Program

<div class="text-center">

<br>

## `Effect<A, E, R>`

</div>

<br>

- A: Succeed, with a type of A: `Effect<Success>`

- E: Fail, for a given set of failures: `Effect<Success, Failures>`

- R: Require, a set of dependencies to be run: `Effect<Success, Failures, Dependencies>`

---

## Effect describes a Program: success, failures, dependencies 

<br>

```ts
import { Effect } from "effect";

const createUser: Effect.Effect<CreatedUser, UserAlreadyExists, UserRepository> = //
```

And all these type parameters can leverage discriminated unions to encode multiple possibilities.

```ts
const createUser: Effect.Effect<
                    CreatedUser, 
                    UserAlreadyExists | OrganizationNotFound, 
                    UserRepository | OrganizationRepository
                  > = //
```


---

## Resilience: **error management**

<br>

> Resilience is the art of designing and implementing systems which can react and recover from expected failures.

<div class="grid grid-cols-2 gap-x-4 pt-5">

<div>

**Synchronous operations**

```ts
// random.ts
export function generateRandomNumber(): number {
  // 
}
```

```ts
// main.ts
import { generateRandomNumber } from "./random";

function main() {
  return generateRandomNumber() * 10;
}
```
</div>

<div v-click>

**Asynchronous operations**

```ts
interface Promise<T> {}

async function generateRandomNumber(): Promise<number> {
  //
}

generateRandomNumber().catch((_: any) => {});

try {
  await generateRandomNumber();
} catch (exception: unknown) {}
```

</div>


</div>

<div class="text-center mt-5" v-click>
<b>-> Same explicitness problem and different APIs</b>
</div>


---

## Resilience: **error management**

> Re-creating information that we had... but lost during the process

<div>

```ts
// main.ts

function isSomeErrorException(exception: unknown): exception is SomeError {
  return exception instanceof Error && exception.name === "SomeError";
}

function isSomeOtherErrorException(exception: unknown): exception is SomeOtherError {
  return exception instanceof Error && exception.name === "SomeOtherError";
}

try {
  generateRandomNumber();
} catch (exception: unknown) {
  // Worst case? We don't even know what to expect from "exception"

  // Best case:
  if (isSomeErrorException(exception)) {
    // do something
  } else if (isSomeOtherErrorException(exception)) {
    // do something else
  }
}
```

</div>

---

## Resilience: **error management** with Effect

```ts {3-9|1,11-12|1,11-20|1,22} {lines:true}
import { Effect, pipe } from "effect";

class NumberIsTooBigError {
  readonly _tag = "NumberIsTooBigError";
}

class NumberIsTooSmallError {
  readonly _tag = "NumberIsTooSmallError";
}

export const generateRandomNumber: Effect.Effect<number, NumberIsTooBigError | NumberIsTooSmallError, never> = //

const main = pipe(
  generateRandomNumber,
  // exhaustive pattern matching
  Effect.catchTags({
    NumberIsTooBigError: () => Effect.succeed(0),
    NumberIsTooSmallError: () => Effect.succeed(1),
  })
);

type Main = Effect.Effect<number, never, never>;
```

---

## Resilience: more than just managing errors

Now we have a proper way of managing errors:
- filtering
- recovering
- retrying
- mapping
- pattern matching

Effect also provides
- Interruptions
- Timeouts
- Resource Management
- Patterns: Circuit Breaker, etc.

---

## Dependencies: Effect has your back

> Program to an interface, not an implementation

```ts {0|2-3|4|all} 
interface Effect<
  A, // success channel
  E, // errors channel 
  R, // dependencies channel
> {}
```

<div v-click>

<b class="mt-3">Built-in decoupling through Dependency Inversion Principle</b>

```ts
import { Effect, Context } from "effect";

interface UserRepository {
  createUser: () => Effect.Effect<CreatedUser, UserAlreadyExistsError, never>;
}

const UserRepository = Context.GenericTag<UserRepository>("UserRepository");

const registerUser: Effect.Effect<CreatedUser, UserAlreadyExistsError, UserRepository> = Effect.gen(function* () {
  const repository = yield* UserRepository
  
  yield* userRepository.createUser()
});


```
</div>

---

## Dependencies: type-safe dependency injection


---

## Concurrency


---


## Resource Management


--- 

## Observability

---

## Effect offers a wide but unified and composable ecosystem

<br>

- Batching/Caching, built-in `effect/Request` module
- Streams, built-in `effect/Stream` module
- Schema Validation, built-in `effect/Schema` module
- Configuration Management, built-in `effect/Config` module
- Data structures, built-in `effect/Data` module
- State Management, built-in `effect/Ref` module
- Scheduling, cron-like built-in module, `effect/Cron` module
- many more, via `effect/*` modules

---

## Disclaimer

**Effect does not necessarily need to be an all-in**

<div>
  <ul>
      <li v-click>It's a toolkit, you only pick what you need</li>
      <li v-click>tree-shakeable </li>
      <li v-click>micro module: lightweight subset of effect</li>
      <li v-click>good promise/callback/sync interop</li>
      <li v-click>usable on both frontend and backend ecosystems</li>
  </ul>
</div>

---

## And now you have the choice...

> Select your difficulty level: Either<HardMode, EasyMode>

<div class="grid grid-cols-2 gap-x-4 pt-5">

<div v-click>
   😵‍💫 A fragmented ecosystem complexifying composability and maintenability
</div>

<div v-click>
  🫵 A streamlined way of developing software with unified standard library and ecosystem
  
</div>

</div>

<div v-after class="flex justify-center mt-5">
   <img width="600" src="/hard-and-easy-mode-dark.png" alt="easy-mode" />
</div>


<style>
  h2 {
    color: #4c7fff;
  }
</style>

---

## Thanks for listening!

If you're interested in Effect, feel 

- discord
- effect-introduction


---
