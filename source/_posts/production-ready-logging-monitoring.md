---
title: "'Production Ready' Non-Negotiable: Structured Logging and Monitoring"
date: 2023-11-14 23:00:00
tags:
  - Software Engineering
  - Theory
  - DevOps
  - Production-Ready
categories:
  - Programming
  - Best Practices
top_img: production-ready-2.jpg
cover: production-ready-2.jpg
---

At RavenDB, I spent nights debugging failures where logs and monitoring seemed to contradict each other. One said "disk queue is dangerously high." The other said "1,000 backups just started simultaneously." Here's what I learned: they were both right, and together they told the whole story. Separately, they would have buried me for hours.

Let me show you what happened, because this is why structured logging and monitoring aren't optional—they're how you survive production.

## The Story: 1,000 Simultaneous Backups

A customer called with a strange problem: their server-wide backup was failing. The machine had plenty of disk space, plenty of CPU, plenty of RAM. Nothing was obviously overloaded. But backups kept timing out.

I started with the monitoring dashboard. One metric jumped out immediately: **Disk Queue Length was sky-high**. The disk wasn’t saturated—it was thrashing. Something was pounding it relentlessly.

Then I checked the logs. The answer was there: approximately 1,000 databases on that server instance were scheduled to start their backups at nearly the same moment. All at once. All competing for the same NAS drive. The feature worked fine during development (a handful of databases), but nobody accounted for what happens when you scale it to production.

**What the monitoring told us:** There’s a symptom—disk queue is in crisis.  
**What the logs told us:** Here’s the cause—1,000 concurrent backup processes.

Without both together, I would have spent hours chasing the wrong solution. I might have tweaked disk I/O settings, added caching, or blamed the NAS. Instead, the solution was simple: serialize the backups instead of running them in parallel. One line of code. But finding it required seeing the *symptom* and the *cause* together.

This is why structured logging and monitoring matter.

## Structured Logging: Not Just Another Log File

The logs showed me the cause. But only because they were structured—searchable, parseable, with context. Imagine you’re debugging that same problem but your logs looked like this:

```
10:00:00 ERROR: Backup failed
10:00:01 ERROR: Backup failed
10:00:02 ERROR: Backup failed
[thousands of identical lines...]
```

You’d have no idea *why*. No database IDs. No process counts. No context. You’d just see errors repeating.

{% note info %}
What do I mean by *well-organized*?

Imagine a log message in a traditional format: "2023-11-15 10:00:00 ERROR: Login failed for user 12345". Now, see it transformed into a structured format: `{"timestamp": "2023-11-15T10:00:00", "level": "ERROR", "message": "Login failed", "userId": 12345}`. The difference? The latter is far easier to parse and analyze programmatically.

{% endnote %}

Structured logging is about creating log messages that are consistent, searchable, and meaningful. Instead of the traditional text-based logs, structured logs are formatted in a way (like JSON) that machines can easily read and humans can understand without pulling their hair out.

{% note info %}

Without proper tooling, structured logging would remain a fancy concept.  
[Kibana](https://www.elastic.co/kibana) and [Splunk](https://www.splunk.com/), are powerful and battl-proven tools in managing structured logs. They are not the only ones that can handle structured logs, of course, but they are pretty ubiqutous.

* Kibana, part of the Elastic Stack, offers robust log visualization and advanced search (query) capabilities.
* Splunk is a bit more holistic, it also provides powerful search features, it has analysis, and visualization features too.

Those two tools do similar jobs, but in the end, they are not exactly the same: Kibana excels in visualizations and data discovery. It is ideal when you need to analyze log patterns over time. Splunk, on the other hand, is a powerhouse for digging deep into data, making it more suitable for complex, data-intensive environments.

{% endnote %}

### Why Structured Logging is Awesome

1. **Searchability**: Ever tried finding a needle in a haystack? That’s what sifting through traditional log files feels like. With structured logging, you can easily filter and search for specific information. Tools like Kibana provide advanced query capabilities, allowing you to make advanced filters for log data.
2. **Consistency**: Consistency is key, especially when you’re dealing with large, distributed systems. Structured logs ensure uniform log formatting across different parts of your application.
3. **Context**: If properly configured and implemented, structured logs provide context. They tell you not just what happened, but also where and why it happened, making debugging more convenient.
4. **Correlation**: If [correlation Id](https://filipnikolovski.com/posts/correlating-logs/) is used, structured logs can be used to correlate events from different components or services.

### Best Practices

* **Choose the Right Level**: Debug, info, warning, error – each level serves a purpose. Use them wisely.
* **Include Essential Information**: Timestamps, error codes, user IDs – the more context, the better. Don't just announce 'there was an error' - write exactly what happened.
* **Keep It Clean**: Log what’s necessary. Too much information can be as bad as too little. What's worse, too much logs may even cause various [performance issues](https://betterprogramming.pub/logs-can-have-a-strong-impact-on-stability-performance-and-garbage-collection-fc47c600a1e0)

## Monitoring: Your Software’s Real-Time Dashboard

Now, the monitoring. The structured logs told me *what happened and why*, but the monitoring told me it was *happening right now* and that it was *critical*. 

Monitoring is your real-time window into your system’s health. It’s not about historical analysis—it’s about knowing *now* that something is wrong before it becomes catastrophic. In that backup case, the Disk Queue Length spike said: "Stop. Something is thrashing the disk. Investigate immediately." Without that alert, the customer might have waited hours for someone to notice and dig into logs.

Think of it as the dashboard in your car. Your gauges don’t tell you *why* the engine temperature is rising—they just tell you it’s rising. You look under the hood (logs) to understand why. But you need that dashboard (monitoring) screaming at you first.

### Key Aspects of Effective Monitoring

1. **System Metrics**: Keep an eye on the basics – CPU usage, memory usage, disk I/O, network stats. These are like the vital signs of your application.
2. **Application Metrics**: These are specific to your application. Number of active users, transaction throughput, response times, queue and cache sizes – metrics that tell you about the performance and health of your application.
3. **Alerts**: What happens when something goes wrong? You should have alerts set up to notify you of potential issues before they escalate.

{% note info %}
**Note that:**

* Battle-proven monitoring toolkits like [Prometheus](https://prometheus.io/docs/introduction/overview/) often have out-of-the-box [alerting features](https://prometheus.io/docs/alerting/latest/overview/)
* More often than not, toolkits like Prometheus have out-of-the-box [exporters](https://prometheus.io/docs/instrumenting/writing_exporters/) that provide data. For example, [this exporter](https://github.com/prometheus/node_exporter) is a really good provider of *system metrics* (like disk, memory or network usage)

{% endnote %}

### Monitoring Best Practices

* **Define Meaningful Thresholds**: Not every spike in CPU usage is a crisis. Set thresholds that make sense for your application.
* **Avoid Alert Saturation**: Excessive, unnecessary alerts can lead to indifference, much like the boy who cried 'wolf' in the famous fairy tale. Ensure that your alerts are meaningful and actionable.
* **Historical Data is Gold**: Make sure to keep historical data, be it a database or tools like Kibana. Without historical data, how will you debug a critical system failure at 2am?


## Why Together Matters

Monitoring without logs leaves you blind to *why*. You see the symptom but not the cause. You react but don't understand.

Logs without monitoring leave you slow. You might eventually piece together what happened from historical logs, but by then the damage is done. You're doing post-mortems instead of preventing fires.

Together, they work like this:
1. **Monitoring alerts you** — something is wrong, *right now*
2. **Logs explain it** — here's what's happening and why
3. **You fix it** — fast, with confidence

That's the difference between a 30-minute incident and a 3-hour one. Between understanding your system and guessing.

## One Important Note

What I've written here isn't a silver bullet. There are cases where the cost of implementing and maintaining structured logging and monitoring exceeds the value—especially for small, non-critical systems. But if your system is production-ready, if people depend on it, if downtime costs money or trust? These aren't optional. They're table stakes.

## Conclusion

In the world of production-ready software, structured logging and monitoring are non-negotiable. They are essential because they give you the insights and foresight needed to ensure your application is not just surviving but thriving in production. More importantly, they give you the speed and confidence to fix things before customers notice they're broken.

## Series Roadmap

- 🔖 [**Introduction: Understanding Production-Ready Software**](https://www.graymatterdeveloper.com/2023/11/14/production-ready-intro/)
- 📈 [**Part 1: Structured Logging and Monitoring**](https://www.graymatterdeveloper.com/2023/11/14/production-ready-logging-monitoring/) - *(You Are Here)*
- ⏳ [**Part 2: Performance Metrics: Ensuring Reliability Under Pressure**](https://www.graymatterdeveloper.com/2023/11/15/production-ready-performance-metrics/)
- 🧰 [**Part 3: Memory Dumps: Deciphering the Black Box**](https://www.graymatterdeveloper.com/2023/11/15/production-ready-memory-dumps/)
- 🧪 [**Part 4: Comprehensive Testing: From Unit to Integration**](https://www.graymatterdeveloper.com/2023/11/18/production-ready-testing/)
- 🛡️ [**Part 5: Beyond the Basics: Security, Scalability, and Documentation**](#) *(Coming Soon)*

## What's Next?

In the [next post](https://www.graymatterdeveloper.com/2023/11/15/production-ready-performance-metrics/), we will explore the second non-negotiable: performance metrics and discuss why they are important.