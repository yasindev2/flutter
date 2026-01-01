## Event Loop Execution Rules (Deep Explanation)

The Dart event loop follows a **strict and deterministic algorithm**.  
These rules explain **exactly why Flutter stays responsive** and how `async` / `await` behave.

---

### Rule 1: Run ALL microtasks until the queue is empty

Before the event loop processes **any new event**, it first executes **every microtask** currently queued.

Microtasks include:
- Code after `await`
- `Future.then` callbacks
- `scheduleMicrotask`

Why this rule exists:

- Microtasks represent **continuations of already-started logic**
- Dart guarantees logical consistency by finishing this work first
- This prevents partial execution states

Important implication:

- If microtasks keep scheduling new microtasks, **UI events will wait**
- Overusing microtasks can delay rendering and user input

---

### Rule 2: Take ONE event from the event queue

Only after the microtask queue is completely empty does the event loop take **a single event** from the event queue.

Event queue contains:
- UI events (tap, scroll, navigation)
- Timers
- I/O completion events

Why only one event:

- Ensures fairness
- Prevents long-running event chains
- Allows microtasks to run between events

---

### Rule 3: Execute the event fully

Once an event is taken:
- Its callback runs synchronously
- The event is not interrupted
- If the event hits an `await`, execution pauses and the event ends

Important clarification:

- The event loop does **not wait** for async operations
- The event is considered complete as soon as it pauses

---

### Rule 4: Go back to step 1

After the event finishes:
- The event loop immediately checks the microtask queue again
- Any resumed async code runs before the next event
- This guarantees predictable async behavior

---

### Why These Rules Matter in Flutter

Because of this design:
- The UI never blocks on network or I/O
- Async code resumes immediately when ready
- User interactions remain responsive
- Frame rendering is not delayed by slow operations

---

### Mental Model Summary

Think of the event loop as:

> “Finish what you already started (microtasks),  
> then handle one new thing (event),  
> then repeat.”

This model explains:
- `async` / `await`
- UI responsiveness
- Why isolates are needed for CPU-heavy work
- Why Flutter feels smooth even with async code
