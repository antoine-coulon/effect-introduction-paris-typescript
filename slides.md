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
      <p class="mt-3">Freelance Lead Software Engineer</p>
      <p class="mt-3">Author <b color="orange">skott, effect-introduction</b></p>
      <p class="mt-3">Advocate <b color="orange">Effect</b></p>
      <p class="mt-3">Contributor <b color="orange">Rush.js, NodeSecure</b></p>
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

- **Resilience**: error management, retries, interruptions, timeouts

- **Dependencies**: composition, testing, decoupling, dependency injection

- **Concurrency**: efficiency, race conditions, starvation, atomics

- **Resource** management: resource-safety guarantees, leaks

- **Observability**: tracing, logging, metrics

---

## The JavaScript ecosystem: massive but fragmented

<br>

> non-exhaustive

- **Control flow**: ts-results, neverthrow, p-retry

- **Dependency injection**: tsyringe, di, awilix, inversify

- **Schema validation**: zod, yup, joi, arktype

- **FP librairies**: fp-ts, ramda

- **Data structures**: immer, immutable-js


---

## Introducing Effect: a unified solution to most of these challenges

<br>

> Effect is a generic data type representing a Program

<div class="text-center">

<br>

## `Effect<A, E, R>`

</div>

<br>

- [A] Success `Effect<Success>`

- [E] Failures `Effect<Success, Failures>`

- [R] Requirements `Effect<Success, Failures, Dependencies>`

---

## Effect is explicit: success, failures, dependencies 

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
<b>→ Same explicitness problem and different APIs</b>
</div>


---

## Resilience: **error management**

> Re-creating information that we had... but lost during the process

<div>

```ts
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

> Introducing Error as Values

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


<div class="grid grid-cols-2 gap-x-4 pt-2">

<div>

**Retrying**

```ts
import { Effect, pipe, Schedule, Duration } from "effect";

const schedulePolicy = pipe(
  Schedule.recurs(5),
  Schedule.addDelay(() => Duration.millis(500)),
  Schedule.compose(Schedule.elapsed),
  Schedule.whileOutput(
    Duration.lessThanOrEqualTo(Duration.seconds(3))
  ),
  Schedule.whileInput(
    (e) => e instanceof Error && e.message !== "_"
  )
);

const programWithRetryPolicy = pipe(
  Effect.failSync(() => new Error("Some_error")),
  Effect.retry(schedulePolicy),
  Effect.catchAll(() => Effect.sync(() => {
    console.log("Program ended")
  }))
);
```
</div>

<div v-click>

**Resource-safe interruptions and timeouts**

<div>

→ An Effect is interruptible by default
<br>
→ Interruptions are guaranteed to be propagated


```ts
import { Effect } from "effect";

const task = Effect.gen(function* () {
  console.log("Start processing...");
  yield* Effect.sleep("2 seconds");
  console.log("Processing complete.");
  return "Result";
});

const timedEffect = task.pipe(Effect.timeout("1 second"));
```
</div>

</div>


</div>


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


```ts {1,9-11|3,6-7|13-18|20-21} {lines:true}
import { Effect, pipe } from "effect";

const registerUser: Effect.Effect<
  CreatedUser,
  UserAlreadyExistsError,
  UserRepository
> = // whatever Effect there

// ts(2345): Type 'UserRepository' is not assignable to type 'never'
//             ^^^^^^^^^^^^
Effect.runSync(registerUser);

const registerUserWithSatisfiedDependencies: Effect<CreatedUser, UserAlreadyExistsError, never> = pipe(
  registerUser,
  Effect.provideService(UserRepository, {
    createUser: () => Effect.succeed(new CreatedUser()),
  })
);

// compiles and works
Effect.runSync(registerUserWithSatisfiedDependencies);
```

---

## Dependencies: seemless testing

> Dependency Inversion Principle makes testing easy

```ts {1-5|8,12|all} {lines:true}
class InMemoryUserRepository implements UserRepository {
  createUser() {
    //
  }
}

test("Should blabla", async () => {
  const fakeRepository = new InMemoryUserRepository();

  const user = await pipe(
    createUser(), 
    Effect.provideService(UserRepository, fakeRepository),
    Effect.runPromise
  );

  expect(user).toEqual("whatever");
});
```

---


## Concurrency: properly managing concurrency is **complex**

> "concurrency is about dealing with lots of things at once", Rob Pike

<br>

- Difficult to achieve a deterministic execution model
- Issues with shared resources and resource management
- Deadlocks, race conditions, starvation
- Memory and CPU control and efficiency

<br>

→ Node.js and the Event Loop model solve many concurrency issues... but some are still left


---

## Concurrency: bounded vs unbounded

<div class="grid grid-cols-2 gap-x-4 pt-1">

<div v-click>

**Promise**

```ts
const userIds = Array.from(
  { length: 1000 }, (_, idx) => idx
);

function fetchUser(id: number): Promise<User> {}

// UNBOUNDED 
function retrieveAllUsers() {
  return Promise.all(userIds.map(fetchUser));
}
```

- No interruption handling
- No guarantee on resource release
- No control over concurrent execution
- `Promise#allSettled` provides finer control over the produced result but suffers from the same issues

</div>

<div v-click>

**Effect**

```ts
const userIds = Array.from(
  { length: 1000 }, (_, idx) => idx
);

// 
const retrieveAllUsers = pipe( 
  userIds,
  Effect.forEach(
    (id) => Effect.promise(() => fetchUser(id)), 
    // BOUNDED
    { concurrency: 30 },
    // OR UNBOUNDED
    { concurrency: "unbounded" },
    // OR INHERITED
    { concurrency: "inherit" }
  )
);
```

</div>

</div>

---

## Resource Management: interruptions and errors support


<div class="grid grid-cols-2 gap-x-4 pt-5">

<div>

```ts
import { setTimeout } from "node:timers/promises";

const leakingRace = () => Promise.race([
  setTimeout(1000), 
  setTimeout(10000)
]);

function raceWithInterruptions() {
  const abortController1 = new AbortController();
  const abortController2 = new AbortController();

  async function cancellableTimeout1() {
    await setTimeout(1000, undefined, { signal: abortController1.signal });
    abortController2.abort();
  }

  async function cancellableTimeout2() {
    await setTimeout(10000, undefined, { signal: abortController2.signal });
    abortController1.abort();
  }

  return Promise.race([cancellableTimeout1(), cancellableTimeout2()]);
}
```
</div>

<div v-click>

```ts
import { Effect } from "effect";

const race = Effect.raceAll([
  Effect.sleep(1000),
  Effect.sleep(10_000).pipe(
    Effect.onInterrupt(() => Effect.log("interrupted"))
  ),
]);
```

<br>


```ts
// React useEffect-like release function
const backgroundJob = Effect.async(() => {
  const timer = setInterval(() => {
    console.log("processing job...");
  }, 500);

  return Effect.sync(() => {
    console.log("releasing resources...");
    clearInterval(timer);
  });
});
```


</div>


</div>

---

## Resource Management: modeling resource-safe flows

<br>

- Scopes offer handle with built-in acquisition and release logic with strong guarantees
- Finalizers are guaranteed to be run in case of Interruptions or Errors
- No props drilling, everything nicely composes

<div v-click>

```ts
import { Effect, Console } from "effect"

// Define how the resource is acquired
const acquire = Effect.tryPromise({
  try: () => getResource(),
  catch: () => new Error("getMyResourceError")
})

// Define how the resource is released
const release = (res: MyResource) => Effect.promise(() => res.close())

// Define how the resource is used
const use = (res: MyResource) => Console.log(`content is ${res.contents}`)

//      ┌─── Effect<void, Error, never>
//      ▼
const program = Effect.acquireUseRelease(acquire, use, release)
```

</div>


--- 

## Observability: built-in Logging, Tracing, Metrics

> Built-in support for Logging, Tracing, Metrics

<div class="grid grid-cols-2 gap-x-4 pt-5">

<div>


```ts {15-18|all} {lines:true}
const getTodo = (
  id: number
): Effect.Effect<
  unknown,
  HttpClientError | TimeoutException
> =>
  httpClient.get(`/todos/${id}`).pipe(
    Effect.andThen((response) => response.json),
    Effect.andThen(S.decode(Todo)),
    Effect.timeout("1 second"),
    Effect.retry({
      schedule: Schedule.exponential(1000),
      times: 3
    }),
    Effect.andThen(
      Effect.log(`getTodo ${id}`)
    ),
    Effect.withSpan("getTodo", { attributes: { id } })
  )
```

</div>

<div>

```ts {all|3|6-10|all}
const child = Effect.void.pipe(
  Effect.delay("100 millis"),
  Effect.withSpan("child")
)

const parent = Effect.gen(function* () {
  yield* Effect.sleep("20 millis")
  yield* child
  yield* Effect.sleep("10 millis")
}).pipe(Effect.withSpan("parent"))
```

```ts {0|all}
const program = task("client", 2, [
  task("/api", 3, [
    task("/authN", 4, [task("/authZ", 5)]),
    task("/payment Gateway", 6, [
      task("DB", 7),
      task("Ext. Merchant", 8)
    ]),
  ])
])
```


</div>

</div>
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

## Effect makes the hard things easy

> A streamlined way of developing software with unified standard library and ecosystem

<div class="mt-5">

  <img src="/complexity.png" />

</div>

---

## Thanks for listening!

<div class="grid grid-cols-12 gap-x-4 pt-5 pr-10 pl-10">

<div class="col-start-1 col-span-10 grid grid-cols-[3fr,2fr] mr-10">
  <div class="pb-4">
    <ul class="leading-8 mt-8 flex flex-col">
    <li><b>Effect website</b>: <b color="cyan">https://effect.website</b></li>
    <li><b>Effect introduction</b>: <b color="cyan">https://github.com/antoine-coulon/effect-introduction</b></li>
    </ul>
  </div>

</div>

<div class="pl-20 col-start-11 col-span-12">
  <img src="/discord-qr.png" class="w-40 margin-0-auto" />
  <img src="/effect.png" class="rounded-full w-40 margin-0-auto mt-4" />
</div>

</div>

