
# Understanding async/await and Task Continuations in C#

We use async /await  to write non-blocking  code with Task . This document explains how async/await works, particularly focusing on the execution flow of methods using async/await, and how continuations are scheduled by the .NET runtime.

### When to use Multi threading and Async/await?
Use multi-threading when you need to perform CPU-bound operations that can run in parallel, such as complex calculations or data processing. Use async/await for I/O-bound operations, such as network calls or file I/O, where you want to avoid blocking the main thread while waiting for the operation to complete.

### Getting Started with Async and Await


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




![Alt](Asserts/withoutAsync.JPG?raw=true)


### What is thread Pool ?

The thread pool is a collection of threads which is already ready that can be used to perform work asynchronously. When you use `Task.Run`, it schedules the work to be done on a thread from the thread pool, allowing for efficient use of system resources.

### Why using Thread Pool instead of Threads directly?
Using the thread pool is more efficient than creating and destroying threads manually. The thread pool manages a set of threads that can be reused for multiple tasks, reducing the overhead of thread creation and destruction.


### What happens when we add `await` inside an async method?
When you use `await` in an async method, it does the following:

* If the await expression is a `Task` which is not completed, then a new thread from the TPL is assigned to the method .
* After that the  control is returned to the caller await method with a task contain status like "NOt completed".
* If the task expression is completed ,then the remaining code block (continueous block ) was executed by some other thread which was assigned by task sheduler in the  runtime 
* After the completion of the task expression method ,the caller method which is paused with a task is now turned to completed and again its continous block was executed by some other thread which was assigned by task sheduler in the  runtime.
* This process repeat until the main method is completed.
* If the await expression is  `Not a Task` ,then the same thread execute the method.

### Note
Whenever declaring Async method ,the return type should be `Task` or `Task<T>` (for methods returning a value). If the method does not return a value, use `Task`. If it returns a value, use `Task<T>`.Then only we can able to await the method and also able know whether the method is completed,processing or failed using this task object.

![Alt](Asserts/returnTask.JPG?raw=true)]

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

See the above code .Here the Main thread (1) enters the `Main` method .It sees the await which is followed by a Non task expression `Method1()`. So the Main thread itself enters the Method1.There it sees another method `Method2()` which is awaited with a `Task Expression`. So the Main thread (1) is now paused and a new thread from the thread pool (2) is assigned to execute the `Method2()`.This thread assigning is done by the Runtime's Task Scheduler.Now the main thread which is paused is returned to the caller method `Method1` and gives a task object with status like Not Completed and The main thread is freed.After the completion of the `Method2` ,the remaining block (continuation block) is executed by some other thread in the thread pool.After the `Method1` is completed ,the status of the task in the caller of `Method1` in the main is set to completed and the remaining block of the main is executed by some other free thread in the thread pool and complete the main method.This is how thread works.

**Output**

```
Start Main - Thread 1
Start Method1 - Thread 1
Start Method2 - Thread 5
End Method2 - Thread 5
End Method1 - Thread 5
End Main - Thread 5
```

Now we have lot of confusion .I said that the Task expression and all the continuation blocks are execute by some other thread but here same thread which executes the Task Expression (Method2) is executing all other things.

### Why this is happening ?

Now consider Thread 2 is executing the `Method2()` and Completed.Now the thread is free now.So the thread pool is assigning a free thread (can be 2 or any other free thread) to the continuation block.Most of the time is assigns the same thread then a new one from the thread pool.Note that  `Most of time` , Not `Always`. 

I think now you got the picture why it is happening.

### Threads on Completed Tasks
Even though,we use Task Expression which is completed ,new thread is not assigned.The thread which executes the Context will execute the completed task expression.

```csharp
static async Task Main(string[] args)
{
    Console.WriteLine($"Start Main - Thread {Thread.CurrentThread.ManagedThreadId}");
    await Method1();
    Console.WriteLine($"End Main - Thread {Thread.CurrentThread.ManagedThreadId}");
    Console.ReadLine();
}
static async Task Method1()
{
    Console.WriteLine($"Start Method1 - Thread {Thread.CurrentThread.ManagedThreadId}");
    await Task.FromResult(10); // This is a completed task
    Console.WriteLine($"End Method1 - Thread {Thread.CurrentThread.ManagedThreadId}");
}
```
**Output**
```
Start Main - Thread 1
Start Method1 - Thread 1
End Method1 - Thread 1
End Main - Thread 1
```

### Task expressions without `await`
If you have a task expression without `await`, the method will not wait for the task to complete. The method will continue executing immediately after starting the task, and the task will run in the background.
```csharp
static async Task Main(string[] args)
{
    Console.WriteLine($"Start Main - Thread {Thread.CurrentThread.ManagedThreadId}");
    Method1(); // No await here
    Console.WriteLine($"End Main - Thread {Thread.CurrentThread.ManagedThreadId}");
    Console.ReadLine();
}
static async Task Method1()
{
    Console.WriteLine($"Start Method1 - Thread {Thread.CurrentThread.ManagedThreadId}");
    await Task.Run(() => Method2());
    Console.WriteLine($"End Method1 - Thread {Thread.CurrentThread.ManagedThreadId}");
}
static async Task Method2()
{
    Console.WriteLine($"Start Method2 - Thread {Thread.CurrentThread.ManagedThreadId}");
    Console.WriteLine($"End Method2 - Thread {Thread.CurrentThread.ManagedThreadId}");
}
```
**Output**
```
Start Main - Thread 1
Start Method1 - Thread 1
End Main - Thread 1
Start Method2 - Thread 5
End Method2 - Thread 5
End Method1 - Thread 5
```

### Foreground vs Background Threads
In .NET, threads can be either foreground or background. Foreground threads keep the application running until they complete, while background threads do not prevent the application from exiting.
Foreground threads are typically used for tasks that must complete before the application exits, while background threads are used for tasks that can run independently of the application's lifecycle.
```
static async Task Main(string[] args)
        {
            Console.WriteLine($"Start Main - Thread {Thread.CurrentThread.ManagedThreadId}");
            Task.Run(() => Method1());
            Task.Run(() => Method2());
            Console.WriteLine($"End Main - Thread {Thread.CurrentThread.ManagedThreadId}");
        }
        static async Task Method1()
        {
            Thread.Sleep(500); // Simulate some work
            Console.WriteLine($"End Method1 - Thread {Thread.CurrentThread.ManagedThreadId}");
        }
        static async Task Method2()
        {
            Thread.Sleep(500); // Simulate some work
            Console.WriteLine($"End Method2 - Thread {Thread.CurrentThread.ManagedThreadId}");
        }
```
**Output**
```
Start Main - Thread 1
End Main - Thread 1
```

### Why the main method ends immediately?
The main method is the foreground thread ends immediately because the `Task.Run` method starts the tasks in the background, it exits right after starting the tasks. The tasks run independently and do not block the main thread.

## Concurrency
Understand that That  **Async with Task = Concurrency** .Task or Async alone is not concurrency.Because we can use multiple task expression without await which will not concurrently.


### What happens when we use `ConfigureAwait(false)`?
When you use `ConfigureAwait(false)`, this means that the continuation after the await can run on any thread, not necessarily the original context (like the UI thread in a GUI application).





