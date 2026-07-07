---
title: Works On My Machine(tm)
date: 2023-12-12T22:56:29+02:00
categories:
  - Programming
  - Debugging
tags:
 - Debugging
 - .Net
 - C#
 - Gotcha
top_img: top.jpg
cover: cover.jpg
---

Oh, but it works on my machine! Today, I found myself staring at the monitor, adding logs and reviewing existing ones, carefully stepping through code to rule out mistakes. The frustration was real. It's these kinds of head-scratchers that remind us why debugging is both an art and a science.

### The Unseen Culprit

Picture this: an orchestrator service working flawlessly in the local environment, but crashing consistently in production. After hours of investigation, I found the culprit: an unhandled exception in an ``IHostedService`` constructor. I was so focused on complex code issues and environmental differences that I missed the simple truth—a file I wanted to load into cache in the ``IHostedService`` constructor simply didn't exist in the deployed instance because CI/CD wasn't shipping it.

### The Smoking Gun

I remembered that a background service shouldn't bring down an entire application without a stack trace. But apparently, any unhandled exception in the hosted service thread would crash the process.

Before .Net 6, an unhandled exception in ``BackgroundService`` (which implements ``IHostedService``) wouldn't terminate the host—the exception would just be logged. But Microsoft [changed that behavior](https://learn.microsoft.com/en-us/dotnet/core/compatibility/core-libraries/6.0/hosting-exception-handling) in .Net 6. That was the surprise I wasn't expecting.

### Fixing It

For ``IHostedService`` implementations, you need explicit error handling. Wrap your startup logic in *try-catch* blocks to log exceptions and decide whether to crash or continue.

If you're using ``BackgroundService``, Microsoft provides a configuration option: ``BackgroundServiceExceptionBehavior``. Set it to ``Ignore`` to restore the pre-.Net 6 behavior (log the exception and keep running), or ``Rethrow`` to crash the process immediately.

```cs
services.Configure<HostOptions>(hostOptions =>
{
   hostOptions.BackgroundServiceExceptionBehavior = BackgroundServiceExceptionBehavior.Ignore;
});
```

The right choice depends on your system, which we'll explore next.

### Choosing Your Strategy

But picking between ``Ignore`` and ``Rethrow`` isn’t straightforward—it depends on your architecture.

#### Fail-Fast vs Resilience

"Fail fast" sounds appealing: stop problems before they spread. This works well for systems where invariants matter most, like distributed consensus protocols—you *want* to crash rather than corrupt state.

But in distributed RPC-based systems, one service crashing can trigger cascading failures across the mesh. I once saw a service grind to a halt due to ThreadPool starvation. The crash wasn’t the issue—it was that upstream services kept retrying, which only made things worse. A 15-minute outage became an hour-long incident.

Contrast this with event-driven or message-queue systems, where services are more decoupled. A single service crashing doesn’t automatically affect others—it just stops processing messages until it restarts.

#### The Circuit-Breaker Approach

Rather than a binary choice, you can actively monitor for broken invariants or specific error patterns and use a circuit breaker to prevent cascading failures. This adds a safety layer between "crash immediately" and "ignore and hope."

The catch: implementing this correctly is hard. You risk being overzealous and shutting down services on transient errors, causing more downtime than the original issue. Monitoring code itself can introduce overhead—I once instrumented code for Prometheus metrics and accidentally added 40% latency to our endpoints.

#### Practical Guidance

- **Match your strategy to your architecture**: RPC-heavy meshes favor resilience; message-queue systems can afford faster failure.
- **Monitor behavior under load**: Set up dashboards and run load tests. Issues appear under stress, not in happy-path testing.
- **Use circuit breakers judiciously**: Rate limiting and circuit breakers are powerful but need careful tuning. Over-engineering them obscures root causes.

### One More Thing

Since my bug was an exception in the constructor, I wondered: would the .Net runtime catch exceptions there? The answer is no—the ``BackgroundServiceExceptionBehavior`` policy only applies to exceptions thrown in ``ExecuteAsync()``, not the constructor. (You can verify this in the [.Net Runtime source](https://github.com/dotnet/runtime/blob/f9529fd9c3fe8a970fb7a68b7fddd15fb686b5dd/src/libraries/Microsoft.Extensions.Hosting/src/Internal/Host.cs#L176).)

So: never dismiss the possibility of a dumb exception. Be paranoid about exceptions in constructors and startup paths. Wrap them, log them, and test your deployment pipeline. *Just because you are paranoid doesn't mean nobody is after you.*