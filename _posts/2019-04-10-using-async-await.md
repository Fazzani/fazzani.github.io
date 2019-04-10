---
title: Working with Async Await
categories: [c#, .net]
category: .net
last_modified_at: 2019-04-10 10:30:01 +0200
tags:
  - c#
  - async
  - await
---

# Working with Async Await

1. Avoid .Result and .Wait
   - Use await for asynchronous code
   - Use .GetAwaiter.GetResult() for synchronous code

2. Use .ConfigureAwait(false) when the calling thread isn't needed
   - Most business logic does not need to return to the calling thread

3. Avoid return await
   - When await is only used in the return statement, return the Task instead
   > Caution: Don't do this inside of try/catch & using blocks

---

<!-- ## Singleton Pattern

**volatile** is used to create a memory barrier* between reads and writes on the variable.
**lock**, when used, causes memory barriers to be created around the block inside the lock, in addition to limiting access to the block to one thread.
Memory barriers make it so each thread reads the most current value of the variable (not a local value cached in some register) and that the compiler doesn't reorder statements. -->
