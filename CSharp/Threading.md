
# Understanding async/await and Task Continuations in C#

We use async /await  to write non-blocking  code with Task . This document explains how async/await works, particularly focusing on the execution flow of methods using async/await, and how continuations are scheduled by the .NET runtime.

---

```csharp
static void Main(string[] args)
        {
            Console.WriteLine($"Start Main - Thread {Thread.CurrentThread.ManagedThreadId}");
            Method1();
            Console.WriteLine($"End Main - Thread {Thread.CurrentThread.ManagedThreadId}");
            Console.ReadLine();
        }

        static void Method1()
        {
            Console.WriteLine($"Start Method1 - Thread {Thread.CurrentThread.ManagedThreadId}");
            Method2();
            Console.WriteLine($"End Method1 - Thread {Thread.CurrentThread.ManagedThreadId}");
        }

        static void Method2()
        {
            Console.WriteLine($"Start Method2 - Thread {Thread.CurrentThread.ManagedThreadId}");
            Console.WriteLine($"End Method2 - Thread {Thread.CurrentThread.ManagedThreadId}");
        }

 ```
The above code is a simple sync method calls.So all the methods are executed by the main thread.

### What happen when we add "Async" keyword to the method signature ?
Transforms the method into a state machine Like how a loop or iterator gets turned into a hidden class.An async method is turned into a class that tracks its current "state",So it can pause at await, and resume later.

For example 

```csharp
using System;
public class C {
    public async void M() {
        
    }
}
```

This code is converted to state machine

```csharp

public class C
{
    [CompilerGenerated]
    private sealed class <M>d__0 : IAsyncStateMachine
    {
        public int <>1__state;

        public AsyncVoidMethodBuilder <>t__builder;

        public C <>4__this;

        private void MoveNext()
        {
            int num = <>1__state;
            try
            {
            }
            catch (Exception exception)
            {
                <>1__state = -2;
                <>t__builder.SetException(exception);
                return;
            }
            <>1__state = -2;
            <>t__builder.SetResult();
        }

        void IAsyncStateMachine.MoveNext()
        {
            //ILSpy generated this explicit interface implementation from .override directive in MoveNext
            this.MoveNext();
        }

        [DebuggerHidden]
        private void SetStateMachine([Nullable(1)] IAsyncStateMachine stateMachine)
        {
        }

        void IAsyncStateMachine.SetStateMachine([Nullable(1)] IAsyncStateMachine stateMachine)
        {
            //ILSpy generated this explicit interface implementation from .override directive in SetStateMachine
            this.SetStateMachine(stateMachine);
        }
    }

    [AsyncStateMachine(typeof(<M>d__0))]
    [DebuggerStepThrough]
    public void M()
    {
        <M>d__0 stateMachine = new <M>d__0();
        stateMachine.<>t__builder = AsyncVoidMethodBuilder.Create();
        stateMachine.<>4__this = this;
        stateMachine.<>1__state = -1;
        stateMachine.<>t__builder.Start(ref stateMachine);
    }
}
```

Also if we put async keyword ,it only create state machine,it will not run the method async.

### Then what is the use of async and this state machine ?

If we need to use await inside a method ,then the methods should be Async.So that we can able to know the current state of the function like what is current running and what are the remaining block of codes (continous block) need to run.

This is because we need to know whether the method is completed ,running or failed.If we use void means we will not know those status .Its will become like fire and forget.



![Alt](Asserts/withoutAsync.JPG?raw=true)

### What happens when we add `await` inside an async method?
When you use `await` in an async method, it does the following:

* If the await expression is a `Task` which is not completed, then a new thread from the TPL is assigned to the method .
* After that the  control is returned to the caller await method with a task contain status like "NOt completed".
* If the task expression is completed ,then the remaining code block (continueous block ) was executed by some other thread which was assigned by task sheduler in the  runtime 
* After the completion of the task expression method ,the caller method which is paused with a task is now turned to completed and again its continous block was executed by some other thread which was assigned by task sheduler in the  runtime.
* This process repeat until the main method is completed.
* Else the await contains method with no task expression ,then the same thread execute the method.

```csharp
static async Task  Main(string[] args)
        {
            Console.WriteLine($"Start Main - Thread {Thread.CurrentThread.ManagedThreadId}");
            await Method1();
            Console.WriteLine($"End Main - Thread {Thread.CurrentThread.ManagedThreadId}");
            Console.ReadLine();
        }
        static async Task Method1()
        {
            Console.WriteLine($"Start Method1 - Thread {Thread.CurrentThread.ManagedThreadId}");
            await  Task.Run(()=> Method2());
            Console.WriteLine($"End Method1 - Thread {Thread.CurrentThread.ManagedThreadId}");
        }
        static async Task Method2()
        {
            Console.WriteLine($"Start Method2 - Thread {Thread.CurrentThread.ManagedThreadId}");
            Console.WriteLine($"End Method2 - Thread {Thread.CurrentThread.ManagedThreadId}");
        }
```

### Note
Whenever declaring Async method ,the return type should be `Task` or `Task<T>` (for methods returning a value). If the method does not return a value, use `Task`. If it returns a value, use `Task<T>`.Then only we can able to await the method and also able to use await keyword inside  method.

![Alt](Asserts/returnTask.JPG?raw=true)]





## 🔁 async/await Execution Flow

### 1. Method Starts → Runs Synchronously Until `await`
```csharp
await Task.Run(() => Method2());
```
- Runs synchronously up to the `await`.
- The compiler splits the method into "before await" and "after await".

---

### 2. At `await` → Method Pauses and Returns a Task
- The method **pauses** at the `await`.
- It returns a `Task` representing "I'll continue later".
- Task is **not complete yet**.
- Control is returned to the caller (e.g., `Main` or runtime).
- The thread is **freed**.

> ✅ The remaining part of the method (after `await`) is called the **continuation**.

---

### 3. Runtime Schedules the Continuation
- When the awaited task completes (e.g., `Task.Run()` finishes),
- .NET picks an available thread from the **thread pool**,
- And runs the continuation (the rest of the async method).

---

### 4. Method Finishes → Task is Marked as Completed
- The async method completes execution.
- The returned `Task` is now **completed**:
  - Status: `RanToCompletion`, or `Faulted` / `Canceled` if error.
- The caller waiting on `await` is resumed.

---

### 5. The Pattern Repeats Up the Call Stack
- If the caller (e.g., `Main`) is also `awaiting`, it pauses.
- Its remaining code becomes a continuation.
- The runtime will schedule it **later**, on **any available thread**.

---

## ✅ Summary

- The method returns an **incomplete task** at `await`.
- The **continuation** is executed when the awaited task completes.
- The **runtime chooses** any thread pool thread to resume execution.
- Once the method completes, the task status becomes `Completed`.
- This pattern continues up the chain of async calls.

---

## 🧠 Final Insight

> The remaining work after `await` is wrapped as a Task and returned to the caller. The Task is incomplete at this point. When the awaited task completes, the continuation is scheduled on any available thread pool thread. After the continuation executes, the Task is marked as completed. This process repeats up the call stack. All this is managed by the .NET runtime using tasks and the thread pool, and that’s how the application continues to work asynchronously without blocking threads.

---

## ✅ Practical Example with Thread IDs

```csharp
static async Task Main(string[] args)
{
    Console.WriteLine($"Start Main - Thread {Thread.CurrentThread.ManagedThreadId}");
    await Method1();
    Console.WriteLine($"End Main - Thread {Thread.CurrentThread.ManagedThreadId}");
}

static async Task Method1()
{
    Console.WriteLine($"Start Method1 - Thread {Thread.CurrentThread.ManagedThreadId}");
    await Task.Run(() => Method2());
    Console.WriteLine($"End Method1 - Thread {Thread.CurrentThread.ManagedThreadId}");
}

static void Method2()
{
    Console.WriteLine($"Start Method2 - Thread {Thread.CurrentThread.ManagedThreadId}");
    Thread.Sleep(1000);
    Console.WriteLine($"End Method2 - Thread {Thread.CurrentThread.ManagedThreadId}");
}
```

---

## 🧰 Optional Explorations
- View the **state machine code** generated by the compiler.
- Understand how **exceptions** flow in async methods.
- See the difference with `ConfigureAwait(false)`.

---

**Tip:** Revisit this file anytime to refresh your understanding of async behavior in C#.

---
