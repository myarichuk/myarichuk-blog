---
title: "'Production Ready' Non-Negotiable: Comprehensive Testing"
date: 2023-11-18T19:31:25+02:00
tags:
  - Software Engineering
  - Theory
  - DevOps
  - Production-Ready
categories:
  - Programming
  - Best Practices
top_img: production-ready-testing.jpg
cover: production-ready-testing.jpg

---

Early in my career, I joined a team that had extensive test coverage—hundreds of unit and functional tests with reasonable mocks. Management was proud of it. The metrics looked great. But when I started digging into the actual tests, I found something shocking: most of them didn't have assertions. They were testing that code ran, not that code worked correctly.

The codebase was in free fall for other reasons (years of accumulating debt), but the tests created a false sense of security. Everything looked fine on paper. The real problem: nobody was actually verifying anything.

That experience taught me something critical: **a false sense of security is worse than no tests at all.**

This is what production-ready testing actually looks like.

## Why Testing Matters in Production Systems

When I worked on RavenDB, I learned that testing isn't about coverage percentages or test counts. It's about catching the bugs that break customers' data. A web app that crashes for 10 minutes? Bad. A database that silently corrupts data during replication? Catastrophic.

In production systems, tests are how you verify:
- Consistency under distributed failures
- Behavior under network partitions
- Data integrity through edge cases
- That your fix doesn't break something else

Tests are your safety net. But only if they actually check for the right things.

## The Three Types of Tests That Matter

Let's start with the big three: unit testing, integration testing, and end-to-end testing. Each tests a different scope, and each catches different types of bugs.

#### Unit Testing: The Small Stuff

Unit testing isolates and tests individual functions or components. It’s fast, targeted, and great for verifying that a specific piece of logic does what you expect.

Practically, unit tests use [mocks](https://www.geeksforgeeks.org/software-engineering-mock-introduction/) to isolate the code under test from its dependencies. A well-written unit test tests *only* one thing.

The catch: unit tests can lie to you. Your function works perfectly in isolation but breaks when it touches the real world. At RavenDB, we had to learn that unit tests for query optimization don’t mean much if you don’t test the query planner talking to the storage engine.

#### Integration Testing: Where Things Actually Work Together

Integration testing is where you spin up multiple components and see if they actually work together. This is where you catch the bugs that unit tests miss—the off-by-one errors in pagination, the race conditions in concurrent writes, the subtle ordering bugs that only show up when components interact.

In an integration test, you’re running a mini-version of your real system. This could be multiple services, a real (or in-memory) database, message queues—whatever your system actually needs to do its job.

{% note info %}
**In-memory databases for testing:** You could set up integration tests with persistent databases, but in-memory databases are faster and simpler to set up/tear down. The tradeoff: they may not catch storage-layer bugs that your real database would catch. For production-critical systems, consider mixing both approaches.
{% endnote %}

#### End-to-End Testing: The Real World

End-to-end testing is testing your system the way users actually use it. Not through unit test mocks, not through a test harness—through the actual API, with real network calls, real data flows.

At RavenDB, E2E tests caught consistency issues that integration tests missed. They caught bugs where locally-optimized code broke under real network conditions. They caught edge cases that no unit test author would think to mock.

E2E tests are slow and flaky if you’re not careful, but they’re the closest thing to production without actually being in production.

#### Other Testing Types: Important But Situational

Beyond the big three of unit, functional, and integration testing, there are many other types. While these are crucial, they're akin to spices in a dish - used according to the need of the project. They’re situational, often tailored to specific project requirements.

* **Performance Testing:**  This ensures your system can withstand heavy loads without breaking. It’s also useful for identifying application breaking points and throughput.
* **Security Testing:** Consider this your app’s digital immune system check. In a world brimming with threats ranging from amateur hackers to advanced cybercriminals, security testing is not just important; it's imperative. However, its relevance may diminish for internal tools not intended for public release. This topic deserves its own blog post, or maybe three.
* **Automated Testing (automated UI testing):** A form of black-box testing where user activities are automated and UI interactions like button clicks are simulated. It's essential for repetitive tasks and a cornerstone of modern CI/CD pipelines.
* **Usability Testing:** Focuses on ensuring your app is intuitive and user-friendly. It's what differentiates a seamless user experience from a frustrating one.

## What Tests Actually Need to Do

Here’s the critical part: **a test without assertions isn’t a test. It’s just code running.**

That team with hundreds of tests but no assertions was running through the motions without actually checking anything. The builds passed. Management was happy. But when you dug into the actual test files, you found lines like:

```csharp
var result = MyFunction(input);
// TODO: verify result
```

Or worse: tests that called code but never checked the output. No assertions. No verification. Just "it didn’t crash, so it must work."

An [assertion](https://en.wikipedia.org/wiki/Test_assertion) is your reality check. It says: "I ran the code, and here’s what I expected to happen." Without it, you’re not testing—you’re just exercising code paths.

## Test Structure: Arrange, Act, Assert

When you write a test, follow this pattern:

1. **Arrange:** Set up your test environment (create objects, mock dependencies, set initial state)
2. **Act:** Call the code you’re testing
3. **Assert:** Verify the result is what you expected

This pattern (called **AAA**) keeps tests focused and readable. Each test should test one thing and verify one outcome.

## Testing Beyond Happy Paths

Most developers start by testing the happy path—the scenario where everything works correctly. It’s natural. But production doesn’t care about happy paths.

In production systems, you need to test:
- **Invalid inputs** — what happens if someone passes null, negative numbers, or strings that are too long?
- **Timeouts and failures** — what happens when a network call fails, or takes 30 seconds instead of 30ms?
- **Concurrent access** — what happens when two requests try to modify the same data simultaneously?
- **Edge cases and boundaries** — off-by-one errors, empty collections, maximum values

At RavenDB, we spent as much time testing failure scenarios as we did testing the happy path. Node crashes, network partitions, disk full, corrupted data—these are the tests that catch real bugs.

## Test Scope and Focus

A common mistake: tests that are too broad. A test that sets up an entire system, modifies 10 different things, and checks 5 outcomes is a test that fails for 5 different reasons. When it breaks, you don’t know which part is wrong.

Keep tests narrow. Each test should test one thing. If a test has multiple assertions, they should all be checking the same behavior from different angles.

## Test Coverage: The Right Metric

Aiming for around 80% code coverage is realistic and reasonable. But don’t obsess over the number. Coverage measures whether you ran the code—not whether your tests actually verify it works correctly.

The team that bragged about 100% coverage but had no assertions? Useless. A team with 60% coverage where every test has real assertions and real verification? Much safer.

Focus on:
- **Are your tests actually verifying outcomes?** (Assertions matter)
- **Are you testing edge cases and failure modes?** (Not just happy paths)
- **Do your tests catch regressions?** (When you change something, do tests fail?)

## Balancing Test Types

Too many unit tests and you’re maintaining brittle tests every time you refactor internals. Too many integration tests and your test suite runs forever.

The practical balance depends on your system:
- **Libraries and low-level systems** — more unit tests (they’re the public interface)
- **Web applications** — mix of unit and integration
- **Distributed systems** — heavy on integration and E2E (real-world interactions matter more than internal implementation)

At RavenDB, integration and E2E tests caught more real bugs than unit tests did. The distributed nature meant that internal correctness wasn’t enough—you had to test the system as a whole.

## Testing Methodologies: TDD vs. BDD

**Test-Driven Development (TDD)** means writing tests before you write the code. You write a test, watch it fail, then write code to make it pass. The idea is that you're forced to think about what you're building before you build it.

Does TDD work? In my experience, yes—for some problems. It's great when you have a clear specification and you're building something well-scoped. It forces you to think about edge cases early. But it's also easy to write tests that are too tightly coupled to your implementation, making refactoring painful.

**Behavior-Driven Development (BDD)** is similar but focuses on the *behavior* rather than the implementation. You write tests that describe what the system should do ("When I query with this parameter, I should get these results"), not how it should do it. This helps keep tests focused on outcomes rather than implementation details.

Both approaches have merit. The key is: **write tests that verify behavior, not implementation. That way, your tests stay valuable even as you refactor the code.**

## The Real Cost of Bad Testing

That team I mentioned at the start—hundreds of tests, no assertions, green builds across the board? They were building on quicksand. Every deployed change was a gamble. Bugs that should have been caught by tests made it to production.

The false sense of security was worse than having no tests at all. Because at least with no tests, you're nervous. You know you're taking risks. With bad tests, you're confident—and wrong.

Production-ready testing means:
- Tests that actually verify outcomes (with assertions)
- Tests that cover failure modes, not just happy paths
- Tests at the right scope (unit for components, integration for systems, E2E for user flows)
- Tests you trust enough to deploy based on

That's what separates systems that survive 2am incidents from systems that collapse.

## Series Roadmap

- 🔖 [**Introduction: Understanding Production-Ready Software**](https://www.graymatterdeveloper.com/2023/11/14/production-ready-intro/)
- 📈 [**Part 1: Structured Logging and Monitoring**](https://www.graymatterdeveloper.com/2023/11/14/production-ready-logging-monitoring/)
- ⏳ [**Part 2: Performance Metrics: Ensuring Reliability Under Pressure**](https://www.graymatterdeveloper.com/2023/11/15/production-ready-performance-metrics/)
- 🧰 [**Part 3: Memory Dumps: Deciphering the Black Box**](https://www.graymatterdeveloper.com/2023/11/15/production-ready-memory-dumps/)
- 🧪 [**Part 4: Comprehensive Testing: From Unit to Integration**](https://www.graymatterdeveloper.com/2023/11/18/production-ready-testing/) - *(You Are Here)*
- 🛡️ [**Part 5: Beyond the Basics: Security, Scalability, and Documentation**](#) *(Coming Soon)*

## What's Next?

Next, we'll explore memory dumps—your window into production crashes when everything else fails.