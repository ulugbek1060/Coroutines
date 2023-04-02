# Coroutines

Kotlin Coroutines is a library for asynchronous programming in Kotlin that simplifies the handling of long-running tasks that might otherwise block the main thread and cause the app to become unresponsive. Coroutines allow developers to write asynchronous code in a sequential and easy-to-read manner, similar to synchronous code.

Here are some key concepts of Kotlin Coroutines:

- Coroutine: A lightweight thread that can be suspended and resumed at any point without blocking the main thread.
- Coroutine Context: A set of rules that define how coroutines should behave, such as which dispatcher to use for running code on a specific thread.
- Dispatcher: An object that determines which thread or threads should be used for running coroutines. The most common dispatchers are `Dispatchers.Main` (for running code on the main/UI thread) and `Dispatchers.IO` (for running I/O-bound tasks on a background thread).
- Suspending Function: A function that can be paused and resumed later without blocking the calling thread. Suspending functions are marked with the `suspend` keyword.

Here's an example of how to use Kotlin Coroutines to perform a long-running task on a background thread:

```kotlin
import kotlinx.coroutines.*

fun main() {
    println("Main Thread: ${Thread.currentThread().name}")
    GlobalScope.launch(Dispatchers.IO) {
        println("Coroutine Thread: ${Thread.currentThread().name}")
        val result = doLongRunningTask()
        withContext(Dispatchers.Main) {
            println("Main Thread: ${Thread.currentThread().name}")
            updateUI(result)
        }
    }
}

suspend fun doLongRunningTask(): String {
    delay(5000) // Simulate long-running task
    return "Result"
}

fun updateUI(result: String) {
    // Update UI with result
}
```

In this code, we create a coroutine using `GlobalScope.launch` and specify the `Dispatchers.IO` dispatcher to run the coroutine on a background thread. We then call `doLongRunningTask`, which is a suspending function that simulates a long-running task by delaying for 5 seconds.

After the task is complete, we use `withContext` to switch to the `Dispatchers.Main` dispatcher and update the UI with the result using the `updateUI` function.

In Kotlin Coroutines, a `CoroutineScope` is an object that provides a structured way to launch and manage coroutines. A `CoroutineScope` is typically created for a specific task or component of an application, and it can be used to launch child coroutines that are tied to its lifecycle.

Here are some key concepts related to coroutine scopes:

1. Coroutine Context: A set of rules that define how coroutines should behave, such as which dispatcher to use for running code on a specific thread. The context of a coroutine scope is used as the default context for all coroutines launched within it.

2. Parent-child relationships: When you launch a coroutine from within a `CoroutineScope`, the new coroutine becomes a child of the parent coroutine. This means that if the parent coroutine is cancelled or fails, all of its children will also be cancelled.

3. Structured concurrency: By using `CoroutineScope` objects to manage coroutines, you can ensure that all coroutines are properly cancelled when they are no longer needed. This helps prevent memory leaks and other issues that can arise from long-running or uncontrolled coroutines.

Here's an example of how to create and use a `CoroutineScope`:

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val scope = CoroutineScope(Dispatchers.Default)

    scope.launch {
        println("Coroutine 1 started")
        delay(1000)
        println("Coroutine 1 completed")
    }

    scope.launch {
        println("Coroutine 2 started")
        delay(2000)
        println("Coroutine 2 completed")
    }

    delay(3000)
}
```

In this code, we create a new `CoroutineScope` using the `CoroutineScope()` constructor and specify the `Dispatchers.Default` dispatcher as its context. We then launch two child coroutines using the `launch` function on the scope.

The first coroutine delays for 1 second before completing, while the second coroutine delays for 2 seconds before completing. We use `delay` to pause the main thread for 3 seconds to allow both coroutines to finish.

When we run this code using `runBlocking`, we see that both coroutines complete successfully:

```
Coroutine 1 started
Coroutine 2 started
Coroutine 1 completed
Coroutine 2 completed
```

# Job

In Kotlin Coroutines, a `Job` is a handle to a coroutine that can be used to control its lifecycle and cancel it if necessary. A `Job` is created whenever a coroutine is launched using the `launch`, `async`, or `runBlocking` functions.

Here are some common use cases for working with `Job`s in Kotlin Coroutines:

1. Cancelling a coroutine: You can cancel a running coroutine by calling the `cancel()` function on its associated `Job`. This will cause the coroutine to stop executing and any resources it was using to be released.

2. Waiting for multiple coroutines to complete: You can use the `CoroutineScope.launch` function to launch multiple coroutines and obtain a reference to their associated jobs. You can then use the `joinAll()` function to wait for all of the coroutines to complete before continuing execution.

3. Handling errors: You can use the `try-catch` block inside a coroutine to handle errors that occur during its execution. If an exception is thrown, the coroutine will be cancelled and any parent coroutines will also be cancelled.

Here's an example of how to launch a coroutine and obtain its associated job:

```kotlin
import kotlinx.coroutines.*

fun main() {
    val job = GlobalScope.launch {
        // Coroutine code here
    }
    // Do other work here
    job.cancel() // Cancel the coroutine
}
```

In this code, we create a new coroutine using `GlobalScope.launch` and store its associated job in the `job` variable. We then do some other work outside of the coroutine before cancelling it using the `cancel()` function.

Note that cancelling a coroutine does not guarantee that it will stop immediately, as it may need time to clean up any resources it was using. You can use the optional parameter of the `cancel()` function to specify whether you want to cancel the coroutine immediately or wait for it to finish its current work.

# SupervisorJob

In Kotlin Coroutines, a `SupervisorJob` is a type of `Job` that is used to manage a group of child coroutines. When a coroutine with a `SupervisorJob` fails, only the failed coroutine and its children are cancelled, while the other coroutines in the group continue to run.

Here's an example of how to use a `SupervisorJob`:

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val supervisor = SupervisorJob()
    val scope = CoroutineScope(coroutineContext + supervisor)

    scope.launch {
        println("Coroutine 1 started")
        delay(1000)
        throw RuntimeException("Coroutine 1 failed")
    }

    scope.launch {
        println("Coroutine 2 started")
        delay(2000)
        println("Coroutine 2 completed")
    }

    delay(3000)
}
```

In this code, we create a new `SupervisorJob` using the `SupervisorJob()` constructor and add it to the coroutine context of a new `CoroutineScope`. We then launch two child coroutines using the `launch` function on the scope.

The first coroutine throws an exception after delaying for 1 second, simulating a failure. The second coroutine delays for 2 seconds before completing successfully.

When we run this code using `runBlocking`, we see that only the first coroutine is cancelled due to its failure, while the second coroutine continues to run:

```
Coroutine 1 started
Coroutine 2 started
Exception in thread "main" java.lang.RuntimeException: Coroutine 1 failed
	at MainKt$main$1$1.invokeSuspend(main.kt:10)
	at kotlin.coroutines.jvm.internal.BaseContinuationImpl.resumeWith(ContinuationImpl.kt:33)
	at kotlinx.coroutines.DispatchedTask.run(DispatchedTask.kt:106)
	at kotlinx.coroutines.EventLoop.processUnconfinedEvent(EventLoop.kt:136)
	at kotlinx.coroutines.internal.DispatchedContinuationKt.resumeCancellableWith(DispatchedContinuation.kt:305)
	at kotlinx.coroutines.intrinsics.CancellableKt.startCoroutineCancellable(Cancellable.kt:30)
	at kotlinx.coroutines.CoroutineStart.invoke(CoroutineStart.kt:110)
	at kotlinx.coroutines.AbstractCoroutine.start(AbstractCoroutine.kt:158)
	at kotlinx.coroutines.BuildersKt__Builders_commonKt.launch(Builders.common.kt:56)
	at kotlinx.coroutines.BuildersKt.launch(Unknown Source)
	at kotlinx.coroutines.BuildersKt__Builders_commonKt.launch$

